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
![es5继承](/img/es5继承.png)

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

![es5继承-寄生组合式](/img/es5继承-寄生组合式.png)  
<img src="https://github.com/liuhualin95/demo/blob/master/publicImg/es5继承-寄生组合式.png">
![es5继承-寄生组合式](https://github.com/liuhualin95/demo/blob/master/publicImg/es5继承-寄生组合式.png)

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

+ 一个类必须有constructor方法，默认会返回实例对象（即this），没有显示定义会默认添加一个空的constructor方法

----

```javascript
console.log(typeof Animal)	//function
Animal()//Uncaught TypeError: Class constructor Animal cannot be invoked without 'new'
console.log(Object.keys(Animal.prototype))	//[]
console.log(Object.getOwnPropertyNames(Animal.prototype))	
//(2) ["constructor", "says"]
``` 

+ 类的数据类型就是函数，然而不能直接像function一样直接调用，必须通过new命令生成实例，new命令会自动调用constructor方法
+ 通过Object.keys()和Object.getOwnPropertyNames()方法可以发现类的方法都定义在prototype对象上，但是不可枚举 

[slide]

## es5与es6继承的几点不同

+ 子类原型对象的constructor指向不同
+ 子类的原型链指针不同
+ 实质不同（this）

[slide]

## es6继承真的可以继承父类的全部行为吗

+ 组合使用构造函数和原型函数
+ 寄生组合式继承

[slide]
## 使用.class/#id/自定义属性样式
----

```javascript
alert('nodeppt');
```

[slide]

## 主页面样式
### ----是上下分界线
----

nodeppt是基于nodejs写的支持 **Markdown!** 语法的网页PPT，当前版本：1.4.5

Github：https://github.com/ksky521/nodeppt
