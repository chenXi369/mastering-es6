## Promise 你真的用明白了么？

Promise 中只有涉及到状态变更后才需要被执行的回调才算是微任务。像`then`、`catch`、`finally`，其他所有的代码执行的都是宏任务（从上往下，同步执行）



#### question1：这些微任务何时被加入微任务队列？

**·** 如果 Promise 的状态为 pending，那么成功或失败的回调会分别被加入`[[PromiseFulfillReactions]]` 和 `[[PromiseRejectReactions]]` 中，这两个数组存储这些回调函数。

**·** 如果此时 Promise 状态为非 pending 时，回调会成为Promise Jobs，也就是微任务。



#### question2：Promise在链式调用中的执行情况？

```js
Promise.resolve()
    .then(() => {
        console.log("then1")
        Promise.resolve()
            .then(() => {
            console.log("then1-1")
        }).then(() => {
            console.log("then1-2")
        })
    }).then(() => {
        console.log("then2")
    }).then(() => {
        console.log("then3")
    })
```

答案：`then1 -> then1-1 -> then2 -> then1-2 -> then3`。

**结论：链式调用中，只有前一个 `then` 的回调执行完毕后，跟着的 `then` 中的回调才会被加入到微任务队列。**



#### question3：Promise存在多个链式调用时的执行情况？

```js
let p = Promise.resolve();

p.then(() => {
    console.log("then1"); 
    Promise.resolve().then(() => {
        console.log("then1-1");
    });
}).then(() => {
    console.log("then1-2");
});

p.then(() => {
    console.log("then2");
});
```

答案：`then1 -> then2 -> then1-1 -> then1-2`

**结论：每个链式调用的开端会首先依次进入微任务队列。**



#### question3 plus

```js
let q = Promise.resolve().then(() => {
            console.log("then1"); 
            Promise.resolve().then(() => {
                console.log("then1-1");
            });
        }).then(() => {
            console.log("then2");
        });

        q.then(() => {
            console.log("then3");
        });
```

上述代码有个陷阱，`then` 每次都会返回一个新的 Promise，此时的 q 已经不是 `Promise.resolve()` 生成的，而是最后一个 `then` 生成的，故：

答案：`then1 -> then1-1 -> then2 -> then3`

**结论：同一个 Promise 的每个链式调用的开端会首先依次进入到微任务队列。**



#### 延伸

`then1-2` 会在何时打印？

```js
console.log("猜一猜打印顺序")
        Promise.resolve()
            .then(() => {
                console.log("then1")
                Promise.resolve()
                    .then(() => {
                        console.log("then1-1")
                        return 1
                    }).then(() => {
                        console.log("then1-2")
                    })
            })
            .then(() => {
                console.log("then2")
            }).then(() => {
                console.log("then3")
            }).then(() => {
                console.log("then4")
            })
```

```js
console.log("猜一猜打印顺序 plus")
        Promise.resolve()
            .then(() => {
                console.log("then1")
                Promise.resolve()
                    .then(() => {
                        console.log("then1-1")
                        return Promise.resolve()
                    }).then(() => {
                        console.log("then1-2")
                    })
            })
            .then(() => {
                console.log("then2")
            }).then(() => {
                console.log("then3")
            }).then(() => {
                console.log("then4")
            })
```

**Ps:  `then` 返回一个新的 Promise，并且会用这个 Promise 去 `resolve` 返回值。**



当 `Promise resolve` 了一个 Promise 时，会产生一个`NewPromiseResolveThenableJob`，这是属于 Promise Jobs 中的一种，也就是微任务。

> This Job uses the supplied thenable and its then method to resolve the given promise. This process must take place as a Job to ensure that the evaluation of the then method occurs after evaluation of any surrounding code has completed.

并且该 Jobs 还会调用一次 `then` 函数来 `resolve Promise`，这也就又生成了一次微任务。

这就是为什么会触发两次微任务的来源。

