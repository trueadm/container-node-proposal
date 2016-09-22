**This document is a work-in-progress**

## What?

I propose the addition of a new DOM node called the ContainerNode. ContainerNodes have similarities with DocumentFragments, in that they are buckets for other DOM nodes and do not get inserted to the document. This is a basic overview of DocumentFragments:

- Like DocumentFragments, ContainerNodes *do not* get inserted into the document, rather their contents get inserted upon the ContainerNode being inserted.

- ContainerNodes will *not* lose their children upon being inserted into the document.

- Child nodes within the ContainerNode will always remain as child nodes until explicitly removed on the ContainerNode itself. 

- If a child node of a ContainerNode gets inserted at another position in the document, upon the ContainerNode being inserted (even if it was already inserted previously) all children nodes will return back into the position under the ContainerNode, in the order as they appear in the ContainerNode’s children NodeList.

- Even though ContainerNode is not actually in the document, it should be possible to call `removeChild` on a Node A that has had a DocumentNode's child nodes previously inserted, whereby any nodes that exist as child nodes in both Node A and the DocumentNode should get removed from Node A.

**Example**

An example of how the syntax might look:

```js
var container = document.createContainerNode();
var elem = document.createElement('div');
var text = document.createTextNode('text');

container.appendChild(elem);
container.appendChild(text);

document.body.appendChild(container);
console.log(container); // #container-node
console.log(container.childNodes); // [ <div></div>, "text" ]

// the ContainerNode never gets appended to the document

console.log(document.childNodes); // [ <div></div>, "text" ]

var someOtherElem = document.createElement('div');

someOtherElem.appendChild(elem);
console.log(someOtherElem.childNodes); // [ <div></div> ]

// even though we inserted elem into someOtherElem, 
// it should still remain a childNode of container

console.log(container.childNodes); // [ <div></div>, "text" ]

// to the document though, it has moved into someOtherElem

console.log(document.body.innerHTML); // text

// if I add another node into the container,
// it won't show on the document until
// the container is re-attached again

var someOtherText = document.createTextNode('more text');

container.appendChild(someOtherText);
console.log(container.childNodes); // [ "text", "more text" ]
console.log(document.body.innerHTML); // text

document.body.appendChild(container);
console.log(document.body.innerHTML); // text more text

// if I remove a node from the container,
// it won't affect the document

container.removeChild(someOtherText);
console.log(document.body.childNodes); // [ "text", "more text" ]

// if I remove the container from document.body, 
// its contents that exist as child nodes on document.body
// and as child nodes of container, should be removed

document.body.removeChild(container);
console.log(container.childNodes); // [ "text" ]
console.log(document.body.childNodes); // []

```

## Why?

The primary reason for adding the ContainerNode come from several problems in the virtual DOM world, specifically from libraries like Inferno and React. Some of these problems are:

### Virtual DOM Components

Virtual DOM libraries have an abstract concept of components, which offer the ability to "render" a virtual DOM representation that gets mounted to the DOM. Historically, they've been limited to forcing the user to have a "root" for their components mount point. A quick example is shown below (using JSX):

```jsx
function MyComponent() {
  return (
    <div>
      <input name="name" placeholder="Enter name" />
      <input name="age" placeholder="Enter age" />
      <input name="address" placeholder="Enter address" />
    </div>
  );
}
```

This is easy for a virtual DOM library to handle as the component's root DOM node becomes the root node returned by the component function. This root node can be inserted freely into an existing DOM tree at most places without any real problems. It's performant and fits in nicely with the DOM API, as other components can easily interface with this component as long as it has a single root. 

The problem is, the user was force to wrap their three `<input />` nodes in a `<div />`.Ideally the user would be able to simply return an array (fragment) of nodes for the virtual DOM library to mount:

```jsx
function MyComponent() {
  return [
    <input name="name" placeholder="Enter name" />
    <input name="age" placeholder="Enter age" />
    <input name="address" placeholder="Enter address" />
  ];
}
```

Unlike the previous example, there is no longer a root node that can be used. If we attempt to use a DocumentFragment as the root node for the render and thus the component, it falls apart as soon as we append the component to the document. 

We can however build a somewhat complex work-around (like Inferno and React's do) that says components can either have a root node or many root nodes. The problem is that building this logic into the virtual DOM library complicates things and has a knock-on effect on performance and the amount of additional code required to make things work.

In the previous example, we could easily `insertBefore`, `appendChild`, `removeChild` and `replaceChild` (all very common virtual DOM operations) on virtual DOM nodes, as we always carry the a reference to the real DOM node somewhere that relates to our virtual DOM node. When we add in logic that says that reference to the real DOM node may not actually be a real DOM node and might be an abstraction, it means considerbly more logic needs to be added to the virtual DOM library.

With ContainerNode, would should be able to make the component's root node that of the ContainerNode. All the logic to handle `insertBefore`, `appendChild`, `removeChild` and `replaceChild` operations on the ContainerNode and its child nodes is kept at a native DOM level, allowing for interpolation between different libraries and frameworks.

So we can get around this for now like React Fibres and Inferno's VFragments do, but they're not pretty and if we can get a native solution it would be highly beneficial.

### Virtual DOM Children

Virtual DOM libraries can also struggle with nested arrays (this also can link to the last point if a component returns an array render). Below is an example (again, in JSX):

```jsx
function MyComponent() {
  return (
    <table>
      <tr><td>Some data...</td></tr>
      {
        [
          <tr><td>Some data...</td></tr>,
          someRef,
          someItems.map(item => <tr><td>Some data...</td></tr>),
          <SomeComponent />
        ]
      }
      <SomeComponent />
      <tr><td>Some data...</td></tr>
    </table>
  );
}
```

Typically, virtual DOM libraries get around this problem by flattening the children before mounting or patching. On large deep nested children, this can be expensive. Furthermore, what if some of the children in a particular array are keyed* and others are not keyed? One massive advantage that ContainerNodes could offer virtual DOM implementations is in the handle of not only dealing with nested arrays of children, but also the process in re-arraning DOM nodes from one state to another.

Given the child nodes of a ContainerNode can change independently of updating the document, it would allow virtual DOM libraries to re-order, add, remove DOM nodes from the ContainerNode without any reflows/redraw operations occuring. Upon re-attached the ContainerNode, it can do all these operations batched together, offering some solutions to problems when needing to re-order DOM nodes in an optional nature. Here's an example of a problem that virtual DOM libraries have to deal with:

* Keys are a virtual DOM concept that allows for the tracking of DOM ndoes by a common unique ID

```jsx
var lastChildren = [
  <input key="1" />
  <div key="2" />
  <Component key="3" />
  <div key="4" />
  <Component key="5" />
];
```

Upon `lastChildren` being mounted by a virtual DOM library, it could generate a ContainerNode that has the following child nodes:

```[ 
  <input>, 
  <div></div>,
  <span>I am a component render!</span>,
  <div></div>,
  <span>I am a component render!</span>,
]
```

Then virtual DOM library is told an update has occured somewhere and it needs to carry out a patch operation with the given new children:

```jsx
var lastChildren = [
  <Component key="3" />
  <div key="4" />
  <div key="2" />
  <input key="1" />
  <Component key="5" />
];
```

The virtual DOM library might be able to solve this problem in a better way now it can use ContainerNodes. It would iterate through `nextChildren`, using a map and the key of the `nextChildren` child virtual DOM node to get the DOM node reference from `lastChildren`. It would then append the DOM node reference to the newly created ContainerNode. As the new ContainerNode is not attached to the document doing this should incur no DOM operations. Once the virtual DOM library has finished created the new ContainerNode, it can then replace the old ContainerNode with itself, letting the browser carry out all the necessary DOM operations (with an optimised method for doing so) to move the old DOM child nodes to their new positions.

## Can we already do this?

TODO

## Why ContainerNode?

It's doesn't have to be, some have suggested it should be called CollectionNode. I'm not too bothered on the name, just what it brings to those who need these features.
