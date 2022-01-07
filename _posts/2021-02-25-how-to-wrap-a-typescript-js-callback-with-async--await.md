---
public: true
layout: post
title: "How to wrap a TypeScript / JS callback with async / await"
date: 2021-02-25 00:00 +0000
tags: javascript typescript async
---

I find that at times the standard TypeScript/JavaScript callback pattern can make code hard to read and manage. To alleviate this problem, we can wrap callbacks with promises to synchronise the call. Going a step further, we can implement async / await to further improve code readability. This is especially true if like me, your primary programming language is OO like C# or Java.

In the below example, I’m working with NeDB, as JS database. Operations in NeDB follow the standard callback pattern. Standard callback pattern in the docs looks like this:

```javascript
db.findOne({'name': 'draupnir'}, function (err, doc) {  
    console.log('findOne callback: err, doc', [err, doc]);  
});
```

In the async / await version below, we create an async function that wraps a promise, allowing us to call await to synchronise on the result. To allow for error handling in case promise reject() is called, we wrap the await in a try-catch-finally block.

In the standard promise version, we simply use `findOnePromise.then()` to inspect the result.

So, depending on your preference, you have some options …

```javascript
///////////////////////
// async / await vesion
///////////////////////

let findOnePromiseFn = async (obj):Promise<any> => {

    return new Promise((resolve, reject) => {

        db.findOne(obj, function (err, doc) {

            if(err)
            {
                reject(err);
            }
            else
            {
                resolve(doc);
            }
        });
    })
}

// Trigger the function

let findOnePromise = findOnePromiseFn({'name': 'draupnir'});

// Potentially do other stuff ...

// Await the result

try
{
    let findOnePromiseResult = await findOnePromise;

    console.log('findOnePromiseResult', findOnePromiseResult);
}
catch (e)
{
    console.log('findOnePromiseResult (Error)', e);
}
finally
{
    // Any clean up
}
  
/////////////////////////////////
// standard promise based version
/////////////////////////////////
  
let findOnePromisFn = () => {

    return new Promise((resolve, reject) => {

        db.findOne({'name': 'draupnir'}, function (err, doc) {

            if(err)
            {
                reject(err)
            }
            else
            {
                resolve(doc);
            }
        });
    })
}

findOnePromisFn().then(
    value => {

        // Success

        console.log('findOnePromiseResult', value);
    }, 
    reason => {

        // Error

        console.error('findOnePromiseResult (Error)', reason);
    }
);

//////////////////////////////////
// Standard async callback version
//////////////////////////////////

db.findOne({'name': 'draupnir'}, function (err, doc) {

    console.log('findOne callback: err, doc', [err, doc]);
});
```