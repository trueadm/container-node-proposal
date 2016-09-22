## What?

I propose the addition of a new DOM node called the ContainerNode. ContainerNodes would share some similarities to DocumentFragments, in that they are buckets for other DOM nodes; yet there would be some key differences. 

- Like DocumentFragments, ContainerNodes *do not* get inserted into the document, rather their contents get inserted upon the ContainerNode being inserted.

- ContainerNodes will *not* lose their children upon being inserted into the document.

- Children nodes within the ContainerNode will always remain as children nodes until explicitly removed on the ContainerNode itself. 

- If children nodes have been inserted at other position in the document, upon the ContainerNode being inserted (even if it was already inserted previously) all children nodes will return back into the position under the ContainerNode, in the order as they appear in the ContainerNode’s children NodeList.

- Even though ContainerNode is not actually in the document, it should be possible to call `removeChild` on a parent that has had a DocumentNode previously attached. Only the contents of the ContainerNode that exist as child nodes on the parent should be removed.

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

document.body.removeChild(container);
console.log(container.childNodes); // [ "text", "more text" ]
console.log(document.body.childNodes); // []
```

## Why?

TODO

## Can we already do this?

TODO
