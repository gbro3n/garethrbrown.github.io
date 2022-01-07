---
public: true
layout: post
title: "How to create a deep copy of an object in JavaScript/TypeScript"
date: 2021-02-25 00:00 +0000
tags: javascript typescript
---

If you already know the difference between a deep and shallow copy of an object, then you don’t need to read the [very good but quite long articles](https://medium.com/@manjuladube/understanding-deep-and-shallow-copy-in-javascript-13438bad941c) I got from my Google search on the most straight forward way to create a deep copy of an object in JavaScript/TypeScript.

Basically, do this:

```
let deepCopy = JSON.parse(JSON.stringify(objectToDeepCopy));
```

Here is a full working example, which you can play with on [Codepen](https://codepen.io/appsoftware/pen/XWNVByp?editors=1111) if you like.

```javascript
let anArray = [
  { id: 1, name: 'Foo'},
  { id: 2, name: 'Bar'}
];

// Create shallow and deep copies of anArray

let shallowCopyOfArray = anArray;

let deepCopyOfArray = JSON.parse(JSON.stringify(anArray)); // <!-- Like this to create a deep copy!

// Log compare of instance equality

console.log('anArray is same object instance as shallowCopyOfArray', shallowCopyOfArray === anArray); // true

console.log('deepCopyOfArray is same object instance as shallowCopyOfArray', deepCopyOfArray === anArray); // false
```