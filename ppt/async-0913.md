title: async-0913
speaker: Henry
url: https://github.com/ksky521/nodeppt
transition: slide3
files: /js/demo.js,/css/demo.css,/js/zoom.js
theme: moon
usemathjax: yes

[slide]

# 没标题

[slide]

## 回调地狱及优化方案
----

* 回调地狱：当业务逻辑复杂的时候，回调的嵌套过多，代码复杂度增加，可读性降低，维护起来也复杂，调试也复杂，这就是回调地狱。  
```javascript
}}}}}}}}}}}}}}}}}}}}}}}}}}}}}}}}
```
* 优化方案：
 *  Promise
 *  Generator
 *  Async/Await
* 将异步操作以同步操作的流程表达出来，避免层层嵌套

[slide]
## Promise
----

* 一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果
* 三种状态，依赖promise对象实例的内部属性[[PromiseState]]，pending、fulfilled(resolved)、rejected
* 状态由异步操作的结果决定，resolve(value)代表进入resolved状态，reject(error)代表进入rejected状态
* 状态改变后不会再变化，任何时候都可以获取状态的结果，对应状态的参数依赖内部属性[[PromiseResult]]

[slide]

## Promise.prototype.then ( onFulfilled, onRejected )
----

* 分别指定resolved状态和rejected状态的回调函数
* resolve函数的参数除了正常的值以外，还可能是另一个Promise实例  
* Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数  

```javascript
var p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

var p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```

[slide]

## Promise.prototype.then
----

```javascript
var p1 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(5), 500)
})

var p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})
p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// 5
```

* 状态由resolve中的promise决定  
http://www.ecma-international.org/ecma-262/7.0/index.html#sec-promise-resolve-functions  

[slide]

## Promise.prototype.then
----

* then方法返回一个新的promise实例，可以链式调用
* then方法的执行过程
 * 设置this值为当前调用then方法的promise对象
 * 获取Promise构造函数，获取当前promise对象的constructor属性，也即Promise构造函数
 ```javascript
 new Promise((resolve, reject) => {resolve(1)})
 ```
 * 创建一个新的promise对象实例
 * 执行then方法中的handler，然后将返回结果在新的promise对象实例中resolve，即reslove(handler中的返回结果)  
 * handler中无返回值时，同样会执行内置的resolve函数将新创建的promise对象的状态置为fulfilled  
http://www.ecma-international.org/ecma-262/7.0/index.html#sec-promise.prototype.then

[slide]

## Promise.prototype.then
----

```javascript
var promise = new Promise((resolve, reject) => {
    resolve(1);
});
promise
  .then(val => val + 1)
  .then(val => console.log(val))
  .then(val => console.log(val));
// 2
// undefined
promise
  .then(val => {
      return new Promise((resolve, reject) => {
          throw new Error('error.')
      })
  })
  .then(val => console.log(val))
  .catch(error => console.log(error));
// Error: error.
```

[slide]

## Promise.prototype.then
----

```javascript
function delay() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(1)
        }, 1000)
    });
}
console.log(
    delay()
        .then((value) => {
            throw new Error('error1')
        })
		.then((value) => {
            throw new Error('error2')
        })
		.catch((error) => {
            console.log(error)
        })
    	//Error: error1
)
//Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
// __proto__:Promise
// [[PromiseStatus]]:"resolved"
// [[PromiseValue]]:undefined
```
[slide]

## Promise.prototype.then
----

```javascript
var promise = new Promise((resolve, reject) => {
    resolve(1);
});
promise
  .then(val => val + 1)
  .then()
  .then(val => console.log(val));
// 2

var promise = new Promise((resolve, reject) => {
    resolve(1);
});
console.log(
    promise
      .then(val => val + 1)
      .then()
);
// Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
// __proto__: Promise
// [[PromiseStatus]]: "resolved"
// [[PromiseValue]]: 2
```

[slide]

## Generator
----

* Generator函数，同样可以理解为一个容器，里面通过yield表达式(或者return)保存多个内部状态
* 调用Genetator函数会返回一个遍历器对象(一个指向内部状态的指针对象)，通过next方法可以遍历内部的状态
* 每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止
* next方法的参数是上一个yield表达式的返回值；next方法自身也有返回值，形式如{value: yield后面表达式的返回值, done: true/false}

[slide]

## Generator
----

```javascript
// 星号表示这是一个 Generator 函数
function* gen() {
    // 第一次 next 只执行到等号右边
    let first = yield (1 + 2)
    // 第二次 next 执行 let first = 和 yield 2
    let second = yield 2
    // 不执行接下来的 next 就卡在上一步了
    let thrid = yield 3
}
let g = gen()
console.log(g.next()) // -> {value: 3, done: false}
console.log(g.next()) // -> {value: 2, done: false}
```

[slide]

## Generator 函数如何用于异步
----

```javascript
function getFirstName () {
    setTimeout(() => gen.next('alex'), 1000);
}

function getSecondName (a) {
    setTimeout(() => gen.next(a + ' perry'), 1000);
}

function* sayHello () {
    var a = yield getFirstName();
    var b = yield getSecondName(a);
    console.log(b); // ->alex perry
}

var gen = sayHello();
gen.next();
```

[slide]

## Generator 函数手动执行
----

```javascript
function getFirstName () {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve('alex'), 1000);
    });
}
function getSecondName (a) {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve(a + ' perry'), 1000);
    });
}
function* sayHello () {
    var a = yield getFirstName();
    var b = yield getSecondName(a);
    console.log(b); // ->alex perry
}
var gen = sayHello();
gen.next().value.then((data) => {
    gen.next(data).value.then((data) => {
        gen.next(data);
    });
});
```

[slide]

## Generator 函数自执行
----

```javascript
function* sayHello () {
    var a = yield getFirstName();
    var b = yield getSecondName(a);
    console.log(b); // ->alex perry
}

function run (genF) {
    var g = genF();
    function next (data) {
        var res = g.next(data);
        if (res.done) return res.value;
        res.value.then((data) => {
            next(data);
        });
    }
    next();
}
run(sayHello);
```

[slide]

## co模块
----

* 自动执行Generator函数
* 前提： yield命令后面需要是promise对象，如果是数组或者对象，则其成员也必须是promise对象
* 返回值为一个promise对象
* 支持并发的异步操作，比如yield命令后面为一个数组或者对象，或者直接使用Promise.all方法都可以

[slide]

## co模块源码
----

```javascript
function co(gen) {
  var ctx = this; // 保存函数的执行上下文对象
  var args = slice.call(arguments, 1) // 传给 gen 函数的参数
  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.apply(ctx, args);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);
    onFulfilled();
    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }
    function next(ret) {
      if (ret.done) return resolve(ret.value); 
      var value = toPromise.call(ctx, ret.value); // 将 value 重新包装成一个 promise 实例
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);      
      return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
        + 'but the following object was passed: "' + String(ret.value) + '"'));
    }
  });
}
```

[slide]

## co模块源码
----

```javascript
function toPromise(obj) {
  if (!obj) return obj;  
// 如果是 Promise 实例，返回这个 promise 实例
  if (isPromise(obj)) return obj; 
// 如果是 generator 函数或者 一个generator
  if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj); 
// 如果是 thunk 函数
  if ('function' == typeof obj) return thunkToPromise.call(this, obj); 
// 如果是一个数组
  if (Array.isArray(obj)) return arrayToPromise.call(this, obj); 
// 如果是一个 plain object
  if (isObject(obj)) return objectToPromise.call(this, obj); 
// 如果是原始值，则返回这个原始值
  return obj; 
}
```

[slide]

## co模块源码
----

* 判断一个对象是否是一个 promise 实例
```javascript
function isPromise(obj) {
  	return 'function' == typeof obj.then;
}
```
* 判断一个对象时候是 generator 实例
```javascript
function isGenerator(obj) {
  	return 'function' == typeof obj.next && 'function' == typeof obj.throw;
}
```
* 判断是否是一个 generator 函数
```javascript
function isGeneratorFunction(obj) {
  	var constructor = obj.constructor;
  	if (!constructor) return false;
  	if ('GeneratorFunction' === constructor.name) return true;
  	return isGenerator(constructor.prototype);
}
```

[slide]

## co模块源码
----

```javascript
function arrayToPromise(obj) {
  return Promise.all(obj.map(toPromise, this)); 
}
```
```javascript
function objectToPromise(obj){
  var results = new obj.constructor();
  var keys = Object.keys(obj);
  var promises = []; 
  for (var i = 0; i < keys.length; i++) {
    var key = keys[i];
    var promise = toPromise.call(this, obj[key]); 
    if (promise && isPromise(promise)) defer(promise, key); 
    else results[key] = obj[key];
  }
  return Promise.all(promises).then(function () {
    return results; // 将 results 作为 onFulfilled 函数的参数
  });
  function defer(promise, key) {
    results[key] = undefined;
    promises.push(promise.then(function (res) {
      results[key] = res;
    }));
  }
}
```

[slide]

## co并发处理异步操作
----

允许某些操作同时进行，等到它们全部完成，才进行下一步

```javascript
// 数组的写法
co(function* () {
  var res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res);
}).catch(onerror);

// 对象的写法
co(function* () {
  var res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2),
  };
  console.log(res);
}).catch(onerror);
```

[slide]

## Generator
----

![generator对象关系图](/img/generator-objects-relationship.png)

[slide]

## Async/Await
----

* Generator函数语法糖，在写法上async函数将generator函数的星号(*)替换成async，将yield替换成await
```javascript
const sayHello = async function () {
    var a = await getFirstName();
    var b = await getSecondName(a);
    console.log(b); // ->alex perry
}
sayHello();
```
* async函数内置执行器，会自动执行到函数末尾，除非碰到return或者抛出错误
* co模块限制了yield命令后面只能是promise对象(thunk不提，后续co版本不再支持)，async函数中的await命令后面可以是promise对象和原始类型的值
* 返回值是一个promise对象

[slide]

## Async/Await实现原理
----

```javascript
async function fn(args) {
  // ...
}

// 等同于

function fn(args) {
  return co(function* () {
    // ...
  });
}
```

[slide]

## Async/Await多个异步请求并发执行
----

```javascript
function getFirstName () {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve('alex'), 1000);
    });
}

function getSecondName () {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve('perry'), 500);
    });
}
async function dbFuc (db) {
  let promises = [getFirstName(), getSecondName()];
  let results = await Promise.all(promises);
  console.log(results);
}
dbFuc();
```

[slide]

## bind函数
----

```javascript
var c = 3;
function d(...args){
    var c = 1;
    console.log(this.c);
    console.log(args);
}
d.bind({c:3}).call({c:4});  //3 []
d.bind({c:3}, 1, 2).bind({c:5}, 3, 4).call({c:4}, 5);  //3 [1, 2, 3, 4, 5]
```

[slide]

## bind函数的polyfill
----

```javascript
Function.prototype.bind = function (context) {
    var self = this,
        args = [].slice.call(arguments, 1),
        F = function () {},
        bound = function () {
            bound.prototype = this instanceof F ?
                new F() : bound.prototype;
            return self.apply(
                this instanceof F ?
                    this : context || this,
                args.concat([].slice.call(arguments))
            );
        };
    //Function.prototype doesn't have a prototype property
    if (this.prototype) {
        F.prototype = this.prototype;
    }
    return bound;
}
```

[slide]

## THANKS