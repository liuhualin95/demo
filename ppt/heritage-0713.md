title: heritage-0713
speaker: Henry
url: https://github.com/ksky521/nodeppt
transition: slide3
files: /js/demo.js,/css/demo.css,/js/zoom.js
theme: moon
usemathjax: yes

[slide]

# ES5、ES6继承浅析

[slide]

# 主要内容 
----

* es5继承 {:&.rollIn}
* es6继承
* es5与es6继承对比
* es6继承可以继承父类的全部行为，吗

[slide]

## es5继承

+ 组合使用构造函数和原型函数  

----

```javascript
function A(type) {
	this.type = type
}
A.prototype.sayType = function() {
	return this.type
}
function B(type) {
	A.call(this, type)
}
B.prototype = new A('')
B.prototype.say = function() {
	return 'my type is B'
}
var b = new B('B')
```

[slide]

+ 组合使用构造函数和原型函数  

----

![es5继承](/img/es5-heritage-01.png)

[slide]

## es5继承

+ 寄生组合式继承  

----

```javascript
function A(type) {
	this.type = type
}
A.prototype.sayType = function() {
	return this.type
}
function B(type) {
	A.call(this, type)
}
B.prototype = Object.create(A.prototype)	//只有这里不同。
B.prototype.say = function() {
	return 'my type is B'
}
var b = new B('B')
``` 

[slide]

+ 寄生组合式继承    

----

![es5继承-寄生组合式](/img/es5-heritage-02.png)

[slide]

## es6继承

* class

----
```javascript
class Animal {
	constructor() {
		this.type = 'animal'
	}
	says(say) {
		console.log(this.type + ' says ' + say)
	}
}
```
```javascript
class Animal {
	constructor() {
		return Object.create(null)
	}
	says(say) {
		console.log(this.type + ' says ' + say)
	}
}
console.log(new Animal() instanceof Animal)
console.log(new Animal() instanceof Object)
```

+ 一个类必须有constructor方法，默认会返回实例对象（即this），没有显示定义会默认添加一个空的constructor方法

[slide]

## es6继承

* class

----

```javascript
console.log(typeof Animal)	
//function
Animal()
//Uncaught TypeError: Class constructor Animal cannot be invoked without 'new'

console.log(Object.keys(Animal.prototype))	
//[]
console.log(Object.getOwnPropertyNames(Animal.prototype))	
//(2) ["constructor", "says"]
``` 

+ 类的数据类型就是函数，然而不能直接像function一样直接调用，必须通过new命令生成实例，new命令会自动调用constructor方法
+ 通过Object.keys()和Object.getOwnPropertyNames()方法可以发现类的方法都定义在prototype对象上，但是不可枚举 

[slide]

```javascript
class Animal {
	constructor() {
		this.type = 'animal'
	}
	says(say) {
		console.log(this.type + ' says ' + say)
	}
}
let animal = new Animal()
class Cat extends Animal {
	constructor() {
		super()	//相当于Animal.prototype.constructor.call(this)
	}
	eat() {
		console.log('eat')
	}
}
let cat = new Cat()
```
* 声明了继承的class，必须先执行super表达式，从而对this进行绑定，才能使用this。
* super表达式的执行过程
 + result = new GetSuperConstructor
 + 给当前词法环境绑定this值为result

[slide]

## 为什么要先执行super表达式呢

* 先创建父类的实例对象this，然后再用子类的构造函数修改this  

----
![es6-this-01](/img/es6-this-01.png)


http://www.ecma-international.org/ecma-262/7.0/index.html#sec-runtime-semantics-classdefinitionevaluation


[slide]

![es6-this-02](/img/es6-this-02.png)

http://www.ecma-international.org/ecma-262/7.0/index.html#sec-ecmascript-function-objects

[slide]

![es6-this-03](/img/es6-this-03.png)

http://www.ecma-international.org/ecma-262/7.0/index.html#sec-ecmascript-function-objects-construct-argumentslist-newtarget

[slide]

## super

----
* super作为函数调用时，只能用在子类的构造函数之中，代表父类的构造函数
* super作为对象使用时，在普通方法中，指向父类的原型对象(prototype)，在静态方法中，指向父类

[slide]

```javascript
class A {
	constructor() {
		this.x = 1
	}
	print() {
		console.log(this.x)
	}
}
A.prototype.x = 2
class B extends A {
	constructor() {
		super()
		this.x = 3
	}
	printX() {
		super.print()
	}
}
let b = new B()
b.printX()
```

* 通过super调用父类的方法的时候，会绑定到子类的this {:&.rollIn}

[slide]

```javascript
class A {
	constructor() {
		this.x = 1
	}
}
A.prototype.x = 2
class B extends A {
	constructor() {
		super()
		this.x = 3
		super.x = 4
		console.log(super.x)
		console.log(this.x)
	}
}
let b = new B()
```

* 因为super绑定到子类的this {:&.rollIn}
 + 所以如果是通过super对某个属性赋值，这时候super就是this，赋值的属性也会变成子类实例的属性
 + 而进行取值操作的时候，super仍会指向父类的原型对象(prototype)

[slide]

![es6-super-get](/img/es6-super-get.png)

http://www.ecma-international.org/ecma-262/7.0/index.html#sec-makesuperpropertyreference

[slide]

![es6-super-set](/img/es6-super-set.png)

http://www.ecma-international.org/ecma-262/7.0/index.html#sec-assignment-operators-runtime-semantics-evaluation

[slide]

+ es6继承    

----

![es6继承](/img/es6-heritage.png)

[slide]

## es5与es6继承的几点不同
----
+ 子类原型对象的constructor指向不同
+ 子类的原型链指针不同
+ 实质不同（this）

[slide]


![es5-constructor](/img/es5-constructor.png)

----

* es5
 * 子类的原型对象__prototype__在不指定constructor的情况下，会指向父类构造函数


[slide]

![es6-constructor](/img/es6-constructor.png)
----

* es6
 * 子类的原型对象__prototype__的constructor，会指向子类构造函数

[slide]

![es5-__proto__](/img/es5-__proto__.png)
----
* es5
 * 子类的原型链指针依旧指向Function.prototype  
 * 子类构造函数和父类构造函数之间不存在关系

[slide]

![es6-__proto__](/img/es6-__proto__.png)
----
* es6
 * 子类构造函数的原型链指针指向父类构造函数
 * 父类构造函数的原型链指针指向Function.prototype

[slide]

![__proto__](/img/__proto__.png)

[slide]

* es5继承创建的子类实例

----

```javascript
function MyArray() {
	Array.apply(this, arguments)
}
MyArray.prototype = Object.create(Array.prototype)
let arr = new MyArray()

arr[0] = 'hello'
console.log(arr.length)	//0

console.log(arr)	//MyArray {0: "hello"}

console.log(Object.keys(arr))	//["0"]
console.log(Object.getOwnPropertyNames(arr))	//["0"]

console.log(arr.hasOwnProperty('length'))	//false
console.log(MyArray.prototype.hasOwnProperty('length'))	//false
console.log(Array.prototype.hasOwnProperty('length'))	//true

```
[slide]

* 正常array实例

----
```javascript
let array = new Array()

console.log(Object.keys(array))		//[]

console.log(Object.getOwnPropertyNames(array))	//["length"]

console.log(array.hasOwnProperty('length'))	//true
```

[slide]

* 原生构造函数会忽略apply(call)方法传入的this，也就是说，原生构造函数的this无法绑定，导致拿不到内部属性。

----
![construtor-function-new-super](/img/construtor-function-new-super.png)

[slide]


* es6继承创建的子类实例

----

```javascript
class AnotherArray extends Array {
	constructor() {
		super()
	}
}
let _arr = new AnotherArray()

_arr[0] = 'hello'
console.log(_arr.length)	//1

console.log(Object.keys(_arr))	//["0"]
console.log(Object.getOwnPropertyNames(_arr))	//["0", "length"]

console.log(_arr.hasOwnProperty('length'))	//true
```

[slide]

* 子类是否可以继承调用isArray方法

----

```javascript
console.log(MyArray.isArray(arr))	
//Uncaught TypeError: MyArray.isArray is not a function
console.log(AnotherArray.isArray(_arr))	
//true

console.log(Object.getOwnPropertyNames(AnotherArray))	
//(3) ["length", "prototype", "name"]
console.log(Object.getOwnPropertyNames(Array))	
//(8) ["length", "name", "arguments", "caller", "prototype", "isArray", "from", "of"]
```
[slide]


## 实质不同


* es5 先新建子类的实例对象this，再将父类的属性添加到子类上，由于父类的属性无法获取，导致无法继承原生的构造函数  

 >例如Array构造函数有一个内部属性[[DefineOwnProperty]]，用来定义新属性时更新length属性，这个属性无法在子类获取  

----

* es6 先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为可以继承

[slide]

## es6继承真的可以继承父类的全部行为，吗

[slide]

```javascript
class MyError extends Error {
	constructor(){
		super(...arguments)
	}
}
let errorInstance = new MyError('myerror-test')
console.log(errorInstance.message === 'myerror-test')

class NewObj extends Object {
	constructor(){
		super(...arguments)
	}
}
let newObjInstance = new NewObj({attr: true})
console.log(newObjInstance.attr === true)

let objectInstance = new Object({attr: true})
console.log(objectInstance.attr === true)
```
[slide]

+ NewObj继承了Object类，但是却无法通过super参数向父类Object传递参数 
* 因为ES6改变了Object构造函数的行为，只有通过new Object()这种形式调用才有作用，否则会忽略传递的参数 {:&.rollIn}

[slide]
# THANKS