##### babel编译流程
babel 的编译流程就是 parse、transform、generate 3步， parse 是把源码转成 AST，transform 是对 AST 的转换，generate 是把 AST 转成目标代码，并且生成 sourcemap。在 transform 阶段，会应用各种内置的插件来完成 AST 的转换。内置插件做的转换包括两部分，一是把不支持的语法转成目标环境支持的语法来实现相同功能，二是不支持的 api 自动引入对应的 polyfill。

###### babel6的问题
es 的标准一年一个版本，也就意味着 babel 插件要实时的去跟进，babel6有一堆的preset-stage*要更新，另外 手动引入polyfill ，比较麻烦。

###### babel7变化
babel 7 废弃了 preset-20xx 和 `preset-stage-x` 的 preset 包，而换成了 preset-env，`preset-env 默认会支持所有 es 标准的特性`，如果没进入标准的，不再封装成 preset，需要手动指定 `plugin-proposal-xxx`。

###### babel7实现 自动引入 polyfill
```
{
    "presets": [["@babel/preset-env", { 
        "targets": "> 0.25%, not dead",
        "useBuiltIns": "usage",// or "entry" or "false"
        "corejs": 3
    }]],
    "plugins": [
        "@babel/plugin-transform-runtime",
    ]
}

说明：
通过 preset-env + corejs + useBuiltIns 实现自动引入polyfill
corejs 就是 babel 7 所用的 polyfill，需要指定下版本，corejs 3 才支持实例方法（比如 Array.prototype.fill ）的 polyfill；
useBuiltIns 就是使用 polyfill （corejs）的方式，是在入口处全部引入（entry），还是每个文件引入用到的（usage），或者不引入（false）；
```

###### babel 中插件的应用顺序
先 plugin 再 preset，plugin 从左到右，preset 从右到左

###### babel7的问题
@babel/plugin-transform-runtime 是不支持配置 targets 的，因为不知道目标环境支持啥，它只能全部做转换。按照babel的应用顺序来说 plugin-transform-runtime 是在 preset-env 前面执行的；等 @babel/plugin-transform-runtime 转完了之后，再交给 preset-env 这时候已经做了无用的转换了。


###### babel8
babel8 的 @babel/polyfills 包就解决了 babel 7 的 @babel/plugin-transform-runtime 的遗留问题；useBuiltIns变成了entry、usage、pure三种方式；所以到了 babel8 的阶段， @babel/preset-env 才是功能完备的。