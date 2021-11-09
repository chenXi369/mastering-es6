## 深入理解Async 和 Await 函数



**async 和 await 存在的意义：**

​       当我们要调用多个then链才能达到要求，有没有可以使得代码更简洁更易懂的方法来达到then链相同的结果呢？async 和 await 就很好的解决了这个问题。

​        首先用 async 声明一个异步函数，然后再用 await 等待异步结果，把以前的 then链的结果直接放在await，非常方便。



**1. async / await 是什么？**

async / await 其实是Promise 的语法糖，它能实现的效果都能用 then 链来实现，它就是为了优化 then 链而开发出来的，增强了代码的可读性。从字面来看，async 是“异步”的简写，await 译为等待。所以可以很好的理解 async 声明 function 是异步的，await等待某个操作完成。语法上强制规定 await 只能出现在 async 函数中。

```js
async function testAsy() {
    rerurn "hello world"
}
let result = testAsy()
console.log(result)
```

猜猜看控制台打印了什么？

这个 async 声明的异步函数把 return 后面直接通过 Promise.resolve() 返回 Promise 对象，可以直接通过 then 链式直接调用：

```js
async function testAsy(){
   return 'hello world'
}
let result = testAsy() 
console.log(result)
result.then(v=>{
    console.log(v) 
})
```

联想一下Promise特点——异步无等待，所以当没有await语句执行async函数，它就会立即执行，返回一个Promise对象，非阻塞，与普通的Promise对象函数一致。

重点就在await，它等待什么呢？

==按照[语法说明](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await)，await等待的是一个Promise对象，或者是其他值（也就是说可以等待任何值），如果等待的是Promise对象，则返回Promise的处理结果；如果是其他值，则返回该值本身。并且await会暂停当前async function的执行，等待Promise的处理完成。若Promise正常处理（fulfillded），其将回调的resolve函数参数作为await表达式的值，继续执行async function；若Promise处理异常（rejected），await表达式会把Promise异常原因抛出；另外如果await操作符后面的表达式不是一个Promise对象，则返回该值本身。==



**2. 深入理解 async / await**

我们来详细说明一下 async / await 的作用。await 操作符后面可以是任意值，当是Promise 对象的时候，会暂停 async function 执行。即 必须要等待 await 后面的 Promise 处理完才能继续