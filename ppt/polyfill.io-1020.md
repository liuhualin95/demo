title: polyfill.io-1020
speaker: liuhualin
plugins:
    - echarts

<slide class="bg-black-blue aligncenter" image="https://source.unsplash.com/C1HhAQrbykQ/ .dark">

#  polyfill.io{.text-serif.text-shadow}

By liuhualin {.text-serif}

<slide>

A [polyfill](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill) is a piece of code (usually JavaScript on the Web) used to provide modern functionality on older browsers that do not natively support it.

For example, [click me](https://caniuse.com/promises).

<slide>

# 过去

### shim + sham

- shim
- sham
- [es5-shim](https://github.com/es-shims/es5-shim)
- [es6-shim](https://github.com/es-shims/es6-shim)

那么，shim 和 sham 是啥？又有什么区别？

shim 是能用的补丁
sham 顾名思义，是假的意思，所以 sham 是一些假的方法，只能使用保证不出错，但不能用。至于为啥会有 sham，因为有些方法的低端浏览器里根本实现不了

<slide>

shim 完美bai模拟了所有 ES5 中可以被完美模拟的方法。du有点绕，就zhi是说 ES5 中有些方法，是dao可以在旧 JS 引擎中完美模拟了，那么 shim 就完美模拟了它们。

ES5 中其他无法被完美模拟的方法，就由 sham 承包了。 sham 只承诺你用的时候代码不会崩溃，至于对应的方法是不是起作用它就不保证了，它只是尽力模拟(as close as possible)

sham 依赖 shim

A monkey patch (also known as "duck punching") is a way for a program to extend or modify supporting system software locally (affecting only the running instance of the program).


monkey patch指的是在运行时动态替换，一般是在startup的时候。
es5-shim属于monkey path。



<slide>


[@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill)



### Size{.text-serif}

The polyfill is provided as a convenience but you should use it with @babel/preset-env and the useBuiltIns option so that it doesn't include the whole polyfill which isn't always needed. Otherwise, we would recommend you import the individual polyfills manually.


es6、es7 特性随便写

但缺点是，babel-polyfill 包含所有补丁，不管浏览器是否支持，也不管你的项目是否有用到，都全量引入

<slide>

```
import "core-js/stable";
import "regenerator-runtime/runtime";
```

> Since now @babel/polyfill is deprecated in favor of separate core-js and regenerator-runtime inclusion, we can optimize regenerator-runtime import. For this reason, regenerator-runtime import will be removed from the source code when targeting browsers that supports generators natively.

<slide>

# 现在

## @babel/preset-env + useBuiltins + targets

@babel/polyfill 包含所有补丁，那假如只需要支持某些浏览器的某些版本，是否有办法只包含这些浏览器的补丁？

<slide>

### @babel/preset-env

@babel/preset-env is a smart preset that allows you to use the latest JavaScript without needing to micromanage which syntax transforms (and optionally, browser polyfills) are needed by your target environment(s). This both makes your life easier and JavaScript bundles smaller!

@babel/preset-env 会根据目标环境来进行编译和打补丁

<slide>


### useBuiltins

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
[@babel/plugin-transform-runtime](https://babeljs.io/docs/en/babel-plugin-transform-runtime)

当应用引入了多个第三库，并且它们存在公用的 helper 方法时，通过该plugin的引入来减少打包体积

<slide>

# 问题
1. 特性列表是按浏览器整理的，那怎么知道哪些特性我用了，哪些没有用到，没有用到的部分也引入了是不是也是冗余？@babel/preset-env 有提供 exclude 的配置，如果我配置了 exclude，后面是否得小心翼翼地确保不要用到 exclude 掉的特性
2. 补丁是打包到静态文件的，如果我配置 targets 为 chrome: 62, ie: 9，那意味着 chrome 62 也得载入 ie 9 相关的补丁，这也是一份冗余

总结来讲为两点：
1. 应用使用的特性列表  
2. 浏览器差异导致的按需加载


<slide>

手动引入确实可以解决问题1，虽然低效，但直观有用。不过，在支持某些引入特性如promise的浏览器上，补丁的引入还是冗余的，因此有了根据浏览器特性动态打补丁的方案。

### What is [Polyfill.io](https://polyfill.io/v3/)?

It's a service which accepts a request for a set of browser features and returns only the polyfills that are needed by the requesting browser.


<slide>