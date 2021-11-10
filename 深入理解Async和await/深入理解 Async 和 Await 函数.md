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

我们来详细说明一下 async / await 的作用。await 操作符后面可以是任意值，当是Promise 对象的时候，会暂停 async function 执行。即 必须要等待 await 后面的 Promise 处理完才能继续：

```js
function testAsy(x){
   return new Promise(resolve=>{setTimeout(() => {
       resolve(x);
     }, 3000)
    }
   )
}
async function testAwt(){    
  let result =  await testAsy('hello world');
  console.log(result);    // 3秒钟之后出现hello world
}
testAwt();
```

await 表达式的运算结果取决于它等的东西。

如果它等到的不是一个 Promise 对象，那 await 表达式的运算结果就是它等到的东西。

如果它等到的是一个 Promise 对象，await 就忙起来了，它会阻塞后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果。

```js
function testAsy(x) {
	return new Promise(resolve => {
        setTimeout(() => {
			resolve(x)
        }, 3000)
    })
}
async function testAwt() {
	let result = await testAsy("hello world")
    console.log(result)
    console.log("lcc")
}
testAwt()
console.log("lisir")
```



**3. async / await 的简单应用**

上面已经说明了 async 会将其后的函数（函数表达式或 Lambda）的返回值封装成一个 Promise 对象，而 await 会等待这个 Promise 完成，并将其 resolve 的结果返回出来。

现在举例，用 setTimeout模拟耗时的异步操作，先来看看不用 async/await 会怎么写

```js
function takeLongTime() {
    return new Promise(resolve => {
        setTimeout(() => resolve("long_time_value"), 1000);
    });
}

takeLongTime().then(v => {
    console.log("got", v); //一秒钟后输出got long_time_value
});
```

如果改用 async/await 呢，会是这样

```js
function takeLongTime() {
	return new Promise(resolve => {
        setTimeout(() => {
			resolve("long_time_value")
        },1000)
    })
}

async function test() {
    let v = await takeLongTime()
    console.log(v)
}

test()
```

tankLongTime()本身就是返回的 Promise 对象，所以加不加 async结果都一样。



**4. async / await 来处理 then 链**

前言，async 和 await 是处理 then 链的语法糖，现在我们来对比一下具体是如何实现的：

假设一个业务，分多个步骤完成，每一步都是异步的，且依赖上一步的结果。我们分别用 Promise.then() 和 async / await 来简单模拟一下：

```js
/**
 * 传入参数 n，表示这个函数执行的时间（毫秒）
 * 执行的结果是 n + 200，这个值将用于下一步骤
 */
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}
```

首先是 Promise.then() ：

```js
function dolt() {
    console.time("dolt")
    let time1 = 300
    step1(time1).then((time2) => {
        step2(time2).then((time3) => {
            step(time3).then((res) => {
                console.log(`res is ${res}`)
                console.timeEnd("doIt");
            })
        })
    })
}

dolt()
```

 然后是 async / await ：

```js
async function Dolt() {
	console.time("Dolt")
    let time1 = 300
    let time2 = await step1(time1)
    let time3 = await step2(time2)
    let result = await step3(time3)
    console.log(`result is ${result}`)
    console.timeEnd("Dolt")
}
```



**5. Promise 处理结果为 rejected**

await 命令后面的 Promise 对象，运行结果可能是 rejected，所以最好把 await 命令放在 try ... catch 代码块中：

两种写法：

```js
async function myFunction() {
	try {
        await somethingThatReturnAPromise()
    } catch (err) {
		// some err handle
    }
}

// 另一种写法
async function myFunction2() {
    await somethingThatReturnAPromise().catch(err => {
        // some err handle
    })
}
```

