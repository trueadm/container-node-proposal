**This document is a work-in-progress**

## What?

I propose the addition of a new DOM node called the ContainerNode. ContainerNodes have similarities with DocumentFragments, in that they are buckets for other DOM nodes and do not get inserted to the document. This is a basic overview of DocumentFragments:

- Like DocumentFragments, ContainerNodes *do not* get inserted into the document, rather their contents get inserted upon the ContainerNode being inserted.

- ContainerNodes will *not* lose their children upon being inserted into the document.

- Child nodes within the ContainerNode will always remain as child nodes until explicitly removed on the ContainerNode itself. 

- If a child node gets inserted at another position in the document, upon the ContainerNode being inserted (even if it was already inserted previously) all children nodes will return back into the position under the ContainerNode, in the order as they appear in the ContainerNode’s children NodeList.

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

TODO

## Can we already do this?

TODO
