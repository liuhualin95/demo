title: polyfill.io-1020
speaker: liuhualin
plugins:
    - echarts

<slide class="bg-black-blue aligncenter" image="https://source.unsplash.com/C1HhAQrbykQ/ .dark">

#  polyfill.io{.text-serif.text-shadow}

By liuhualin {.text-serif}

<slide :class="size-50">
> A [polyfill](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill) is a piece of code (usually JavaScript on the Web) used to provide modern functionality on older browsers that do not natively support it.
> {.text-quote}

> For example, [click me](https://caniuse.com/promises).



<slide class="">

## 过去 

#### shim + sham
` `
- shim
- sham
- [es5-shim](https://github.com/es-shims/es5-shim)
- [es6-shim](https://github.com/es-shims/es6-shim)

<slide>


那么，shim 和 sham 是啥？又有什么区别？

- shim 是能用的补丁  
- sham 顾名思义，是假的意思，所以 sham 是一些假的方法，只能使用保证不出错，但不能用。至于为啥会有 sham，因为有些方法的低端浏览器里根本实现不了

-----

:::{.fadeIn.animated.delay-2800}
- shim 完美模拟了所有 ES5 中可以被完美模拟的方法。有点绕，就是说 ES5 中有些方法，是可以在旧 JS 引擎中完美模拟的，那么 shim 就完美模拟了它们。

- ES5 中其他无法被完美模拟的方法，就由 sham 承包了。 sham 只承诺你用的时候代码不会崩溃，至于对应的方法是不是起作用它就不保证了，它只是尽力模拟(as close as possible)

--- 
- sham 依赖 shim

<slide>



A monkey patch (also known as "duck punching") is a way for a program to extend or modify supporting system software locally (affecting only the running instance of the program).


monkey patch指的是在运行时动态替换，一般是在startup的时候。
es5-shim属于monkey path。



<slide>
# 现在{.lightSpeedIn}
` `

[@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill){.fadeIn.animated.delay-300}

:::{.fadeIn.animated.delay-800}

### Size{.text-serif}

The polyfill is provided as a convenience but you should use it with @babel/preset-env and the useBuiltIns option so that it doesn't include the whole polyfill which isn't always needed. Otherwise, we would recommend you import the individual polyfills manually.


es6、es7 特性随便写

但缺点是，babel-polyfill 包含所有补丁，不管浏览器是否支持，也不管你的项目是否有用到，都全量引入

<slide>

```
import "core-js/stable";
import "regenerator-runtime/runtime";
```

` `

> Since now @babel/polyfill is deprecated in favor of separate core-js and regenerator-runtime inclusion, we can optimize regenerator-runtime import. For this reason, regenerator-runtime import will be removed from the source code when targeting browsers that supports generators natively.

<slide>

# 优化
` `

## @babel/preset-env + useBuiltins + targets

` `

@babel/polyfill 包含所有补丁，那假如只需要支持某些浏览器的某些版本，是否有办法只包含这些浏览器的补丁？

<slide>

### @babel/preset-env

` `

@babel/preset-env is a smart preset that allows you to use the latest JavaScript without needing to micromanage which syntax transforms (and optionally, browser polyfills) are needed by your target environment(s). This both makes your life easier and JavaScript bundles smaller!

@babel/preset-env 会根据目标环境来进行编译和打补丁

<slide>


### useBuiltins

` `

@babel/preset-env has 2 different modes, which can be enabled with the useBuiltIns option: entry and usage, which optimize imports of core-js in different ways.


- useBuiltIns = entry 的含义是找到入口文件里引入的 @babel/polyfill，并替换为 targets 浏览器/环境需要的补丁列表。  
- useBuiltIns = usage 的含义是根据不同文件所需的补丁来引入对应的补丁，能一定程度上减少不必要的补丁。

```
入口处引入
import '@babel/polyfill';
----->
替换后的内容，比如：
import "core-js/modules/es7.string.pad-start";
import "core-js/modules/es7.string.pad-end";
...
```
> corejs2.0是@babel/polyfill，corejs3.0是替换core-js/stable  
> 了解[corejs](https://github.com/zloirock/core-js/blob/master/docs/2019-03-19-core-js-3-babel-and-a-look-into-the-future.md#babelpreset-env)




<slide>


### targets

` `

指定应用需要支持的浏览器环境，语法参考[browserslist](https://github.com/browserslist/browserslist)

```
  presets: [
    [
      '@babel/preset-env',
      {
        'useBuiltIns': 'entry',
        'corejs': 3,
        // 支持chrome 72+ 及 IE 11+
        'targets': {
          chrome: '72',
          ie: '11',
        }
      },
    ],
  ]

```

<slide>

源代码引入如下

```
import "core-js/stable";
import "regenerator-runtime/runtime";
```

When targeting chrome 72, it will be transformed by @babel/preset-env to

```
import "core-js/modules/es.array.unscopables.flat";
import "core-js/modules/es.array.unscopables.flat-map";
import "core-js/modules/es.object.from-entries";
import "core-js/modules/web.immediate";
```

when targeting chrome 73 (which completely support ES2019 standard library), it will just become a single smaller import:

```
import "core-js/modules/web.immediate";
```

<slide>

# 再优化一步

` `

[@babel/plugin-transform-runtime](https://babeljs.io/docs/en/babel-plugin-transform-runtime)

当应用引入了多个第三库，并且它们存在公用的 helper 方法时，通过该plugin的引入来减少打包体积

<slide>

# 问题{.swing}

` `

:::{.fadeIn.animated.delay-500}

1. 特性列表是按浏览器整理的，那怎么知道哪些特性我用了，哪些没有用到，没有用到的部分也引入了是不是也是冗余？@babel/preset-env 有提供 exclude 的配置，如果我配置了 exclude，后面是否得小心翼翼地确保不要用到 exclude 掉的特性
2. 补丁是打包到静态文件的，如果我配置 targets 为 chrome: 62, ie: 9，那意味着 chrome 62 也得载入 ie 9 相关的补丁，这也是一份冗余

` `

:::{.fadeIn.animated.delay-800}
总结来讲为两点：

:::{.fadeIn.animated.delay-1000}
1. 应用使用的特性列表  
2. 浏览器差异导致的按需加载


<slide>

手动引入确实可以解决问题1，虽然低效，但直观有用。不过，在支持某些引入特性如promise的浏览器上，补丁的引入还是冗余的，因此有了根据浏览器特性动态打补丁的方案。

----


### What is [Polyfill.io](https://polyfill.io/v3/)?

` `

It's a service which accepts a request for a set of browser features and returns only the polyfills that are needed by the requesting browser.


<slide>

!![](https://sf1-dycdn-tos.pstatp.com/obj/eden-cn/upshbobps/ad_share/111.png .size-50.aligntop)

```
// example
// IE 7.0
curl https://polyfill.io/v3/polyfill.min.js -H "User-Agent:Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0)"

// IE 8.0
curl https://polyfill.io/v3/polyfill.min.js -H "User-Agent:Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0)"
```
<slide>

业务线直接在客户端访问 https\://polyfill.io/v3/polyfill.min.js，polyfill service会识别请求头中的UA，下发不同的 polyfill，做到按需加载 polyfill


polyfill.io每次访问的 time 大概是0.5s，而且必须前置加载，影响首屏性能，因此需要挂载到 CDN 服务

<slide>

!![](https://sf1-dycdn-tos.pstatp.com/obj/eden-cn/upshbobps/ad_share/222.png .size-50.alignright)
### 方案一
` `

1. 客户端直接访问服务地址，如https\://cdn.bytedance.com/cdnpoly/
2. 服务解析请求头中的ua，然后将ua编码后，通过query携带的方式返回给客户端一个js请求地址，如https\://s1.pstatp.com/cdnpoly/?ua=xxxx
3. 客户端加载https\://s1.pstatp.com/cdnpoly/?ua=xxxx此地址，完成polyfill的js加载


------

缺点\: 步骤1的qps可能比较高，考虑预留机器；两次域名解析，无法避免  
优点\: 客户端使用简便

<slide>

!![](https://sf1-dycdn-tos.pstatp.com/obj/eden-cn/upshbobps/ad_share/333.png .size-50.alignright)

### 方案二
` `
1. 客户端直接访问服务地址，如https\://cdn.bytedance.com/cdnpoly/
2. 服务解析请求头中的ua，然后将ua编码后，在服务端请求CDN服务，如https\://s1.pstatp.com/cdnpoly/?ua=xxxx，紧接着将结果返回给客户端加载js


------
缺点\: 步骤1的qps可能比较高，考虑预留机器；两次域名解析，内置服务可以使用服务发现，降低至一次  
优点\: 客户端使用简便

<slide>
!![](https://sf1-dycdn-tos.pstatp.com/obj/eden-cn/upshbobps/ad_share/444.png .size-50.alignright)

### 方案三
` `
1. 客户端直接访问服务地址，如https\://s1.pstatp.com/cdnpoly/?ua=xxxx

` `

缺点：
1. 客户端操作比较麻烦\:  
a. query中的ua需要客户端自行获取，比如navigator获取，然后encodeURIComponent  
b. 如果ua获取不到，需要携带一个时间戳，防止cdn缓存，这里不携带应该也没问题
2. 客户端获取ua能否获取到？即是否在加载polyfill前因为兼容性问题导致崩溃？  
https://developer.mozilla.org/zh-CN/docs/Web/API/NavigatorID/userAgent  
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date/getTime
---
优点：
1. 服务qps应该不会比较高  
2. 仅一次cdn域名解析，缓存后基本无开销

<slide>

### Source code  
` `
- [polyfill-service](https://github.com/financial-times/polyfill-service)  
- [polyfill-library](https://github.com/Financial-Times/polyfill-library)

` `
  
  
https\://cdn.bytedance.com/poly/

<slide class="aligncenter">

## THANKS{.text-serif.animated.bounceIn}
