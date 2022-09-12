# 目录

1. 简单的搭建一个环境
2. loader 的分类和执行顺序
3. loader 组成
4. 实现 babel-loader
5. 实现一个 loader 功能：在代码顶部加入 注释
6. 实现图片模块 loader
7. style-loader css-loader less-loader 实现

loaderJs1,loaderJs2,loaderJs3 console 得输出不一样改成 loader1,2,3

```js
function loader(source) {
    console.log("loader1~~~~~~~~~");
    return source;
}
module.exports = loader;

```

webpack.config.js

```js
let path = require("path");

module.exports = {
    mode: "development",
    entry: {
        home: "./src/index.js",
    },
    output: {
        filename: "index.js",
        path: path.resolve(__dirname, "dist"),
    },
    // 专门针对解析Loader的
    resolveLoader: {
        // 匹配下方Loader 先去node_modules找，没有再在loader下找
        modules: ["node_modules", path.resolve(__dirname, "loader")],
        // 别名
        // alias: {
        //     loaderJs: path.resolve(__dirname, "loader", "loaderJs")
        // }
    },
    module: {
        // loader
        rules: [
            {
                test: /\.js$/,
                use: ["loaderJs2", "loaderJs1"],
            },
        ],
    },
};
```

npx webpack  
![](https://img-blog.csdnimg.cn/20210121105944724.png)

针对上面代码分析,  
`resolveLoader` 是专门针对 `loader` 的配置，可以使用别名的方式，更方便点就是配置 `modules`: 将 rules 中 use 的 `loader` 优先匹配 `modules` 找不到，再在根目录下 `loader` 文件夹中查找。也可以直接在 use 中写 `path.resolve(__dirname, “loader”, “loaderJs”)`，推荐配置 modules

## 2.loader 的分类和执行顺序  

执行顺序由 右往左，由下到上。  
由下到上验证：改变 use 的写法，可以单个独自写  
![](https://img-blog.csdnimg.cn/20210121110333949.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTM1NTEy,size_16,color_FFFFFF,t_70)  
loader 的分类：顺序：pre 前置 +normal 普通的 + inline 行内 loader+ post 后置  
通过 enforce 设置

```js
   {
        test: /\.js$/,
        use: "loaderJs1",
    },
    {
        test: /\.js$/,
        use: "loaderJs2",
        enforce: "pre",
    },
    {
        test: /\.js$/,
        use: "loaderJs3",
        enforce: "post",
    },

```

![](https://img-blog.csdnimg.cn/20210121110700927.png)  
inline 行内 loader，在 loader 文件夹下 新建 loaderJs-inline.js, 里面跟其他 loader 一样，打印 `console.log(“loader-inline~~~~~~~~~”)`  
用法：在 js 中用  
index.js, 这里新建一个 a.js

```js
//require("-!loaderJs-inline!./a.js");
let str = require("loaderJs-inline!./a.js");

console.log("index.js");

```

npx webpack 验证顺序：pre 前置 +normal 普通的 + inline 行内 loader+ post 后置  
![](https://img-blog.csdnimg.cn/20210121112050306.png)  
`require("-!loaderJs-inline!./a.js"); -! `不会让 pre normal 执行了 只执行它自己和 post  
`require("!loaderJs-inline!./a.js");!` 不会让 normal 执行了  
`require("!!loaderJs-inline!./a.js");!!` 什么都不要

## 3.loader 组成  

默认两部分：`pitch`  `normal`  
use:[loader3,loader2,loader1];  
loader 先执行 pitch 方法，3=》2=》1 顺序执行，然后获取资源，再返回 1，2，3  
![](https://img-blog.csdnimg.cn/2021012111250896.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTM1NTEy,size_16,color_FFFFFF,t_70)  
如果 loader 有返回值 就会直接返回，（阻断功能）  
![](https://img-blog.csdnimg.cn/20210121112759505.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTM1NTEy,size_16,color_FFFFFF,t_70)  
如果我们在我们的 loaderJs1.js 中加入 picth 只执行了 loaderJs2,loaderJs3.js

```js
function loader(source) {
    console.log("loader1~~~~~~~~~");
    return source;
}

loader.pitch = function (params) {
    return "1111";
};
module.exports = loader;

```

![](https://img-blog.csdnimg.cn/20210630155832843.png)

## 4. 实现 babel-loader

> babel-loader  
> ------------------es6 转换 ----------------------  
> @babel/core  
> @babel/preset-env  
> 安装这 3 个，然后配置解析 js 的规则

```js
resolveLoader: {
    // 匹配下方Loader 先去node_modules找，没有再在loader下找
    modules: ["node_modules", path.resolve(__dirname, "loader")],
    // 别名
    // alias: {
    //     loaderJs: path.resolve(__dirname, "loader", "loaderJs")
    // }
},
module:
rules: [
    {
        test: /\.js$/,
        use: {
            loader: "babel-loader",
            options: {
                // 插件库，预设
                presets: [
                    "@babel/preset-env", //js es6语法转换
                ],
            },
        },
        include: path.resolve(__dirname, "src"), //只在src下找
    },
],

```

index.js 中写个 es6 的语法

```js
class A {
    constructor(param) {
        console.log("es6");
    }
}

```

这里比较基础 不多说了，能编译成功！ npx webpack

接下来我们将 loader: “babel-loader”, 改为 loader: “my-babel-loader”, 在 loader 文件夹下新建 my-babel-loader

```js
my-babel-loader.js
// 用来处理es6的语法转换
let babel = require("@babel/core");
let loaderUtils = require("loader-utils"); //loader的工具类
function loader(source) {
    //this 里面包含很多loader的内容包括有多少个Loader等 可以打印看下
    // console.log(this);
    console.log("babel-loader~~~~~~~~~");
    // 利用工具类的方法获取 预设 presets
    let options = loaderUtils.getOptions(this);
    // 内部的 同步的变量，如果是异步 就放在异步回调里面
    let cb = this.async();
    // 转化
    babel.transform(
        source,
        {
            ...options,
            // 方便调试的时候 需要在webpack。config.js中设置de
            sourceMap: true,
            filename: this.resourcePath.split("/").pop(), //文件名 this.resourcePath完整的文件路径 ，我们这里可以取她的文件名
        },
        function (err, result) {
            // 异步回调
            cb(err, result.code, result.sourceMap); //异步
        }
    );
}

module.exports = loader;
```

webpack,config.js 配置下 devtool:‘source-map’  
npx webpack  
![npx webpack ](https://img-blog.csdnimg.cn/20210121143919541.png)  

## 5. 实现一个 loader 功能：在代码顶部加入 注释  

配置文件

```js
{
    test: /\.js$/,
    use: {
        loader: "text-loader",
        options: {
            text: "一脸云",
            filename: path.resolve(__dirname, "src", "text.js"),
        },
    },
    include: path.resolve(__dirname, "src"), //只在src下找
},

```

新建 loader 中 text-loader

> npm i loader-utils schema-utils -D

```js
let loaderUtils = require("loader-utils"); //loader的工具类
let { validate } = require("schema-utils"); //骨架校验，验证写的对不对  json 对象等
let fs = require("fs");
function loader(source) {
    console.log("text-loader~~~~~~~~~");
    // 不写默认的，是否缓存  this.cacheable(false) 不要缓存
    this.cacheable && this.cacheable();
    let options = loaderUtils.getOptions(this);
    let schema = {
        text: {
            type: String,
        },
        filename: {
            type: String,
        },
    };
    validate(schema, options, "text-loader"); //第三个参数是报错信息
    // 内部的 同步的变量，如果是异步 就放在异步回调里面
    let cb = this.async();
    if (options.filename) {
        // 打包实时监听的时候，引入文件的内容修改时不会实时监听的，需要用这个方法将这个文件关联到我们的依赖关系中去
        this.addDependency(options.filename);
        fs.readFile(options.filename, "utf8", function (err, data) {
            // 这里是异步
            cb(err, `/**${data}**/${source}`);
        });
    } else {
        cb(null, `/**${options.text}**/${source}`);
    }
}

module.exports = loader;
```

npx webpack , 注意上面的 addDependency 是将文件关联进依赖，这样监听打包的时候改变文件就能实时更新  
![](https://img-blog.csdnimg.cn/20210121183832852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTM1NTEy,size_16,color_FFFFFF,t_70)

## 6. 实现图片模块 loader  

原本我们用的是 url-loader 和 file-loader。 url-loader 会做判断 limit 大小，小于这个 limit，会自已将二进制转化成 base64 输出，反之，交给 file-loader 去处理  
![](https://img-blog.csdnimg.cn/20210122113248333.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTM1NTEy,size_16,color_FFFFFF,t_70)  
下面我们手写下  
配置文件

```js
{
    test: /\.jpg$/,
    // use: {
    //     loader: "my-file-loader",
    // },
    // my-url-loade
    use: {
        loader: "my-url-loader", //第一步 让my-file-loader处理路径  第二步 转base64
        options: {
            limit: 1000 * 1024,
        },
    },
    include: path.resolve(__dirname, "src"), //只在src下找
},

```

loader 文件夹下新建 my-url-loader.js

```js
//第一步 让my-file-loader处理路径  第二步 转base64
let loaderUtils = require("loader-utils"); //loader的工具类
let mime = require("mime"); //获取图片类型的
function loader(source) {
    // 获取参数
    let { limit } = loaderUtils.getOptions(this);
    console.log(source, limit);
    if (limit && limit > source.length) {
        return `module.exports="data:${mime.getType(
            this.resourcePath
        )};base64,${source.toString("base64")}"`;  //这里有点不太明白这个二进制转base64的写法，不知道是不是固定这么写
    } else {
        return require("./my-file-loader").call(this, source);
    }
}

// 图片返回的是二进制 ，将source代码转化成二进制
loader.raw = true;
module.exports = loader;
```

loader 文件夹下新建 my-file-loader.js

```js
// 目的是根据图片生成一个md5 发射倒dist目录下，还会返回当前的图片路径
let babel = require("@babel/core");
let loaderUtils = require("loader-utils"); //loader的工具类
function loader(source) {
    console.log("my-file-loader~~~~~~~~~", source);
    // 获取文件名
    let filename = loaderUtils.interpolateName(this, "[hash].[ext]", {
        content: source,
    });
    // 发射文件
    this.emitFile(filename, source);
    return `module.exports="${filename}"`;
}

// 图片返回的是二进制 ，将source代码转化成二进制
loader.raw = true;
module.exports = loader;
```

效果，可以试下 改下 limit 我试过了，还是可以的，这里就不单独截图了。  
![](https://img-blog.csdnimg.cn/2021012211365933.png)  
**7.style-loader css-loader less-loader 实现**  
配置文件，css-loader 主要解决 @import 背景图片等

```js
resolveLoader 这里就不说了 上面统一配置了
{
    test: /\.less$/,
    // use: {
    //     loader: "my-file-loader",
    // },
    // my-url-loade
    use: ["style-loader", "css-loader", "less-loader"],
        include: path.resolve(__dirname, "src"), //只在src下找
    },

```

在 loader 新建三个文件  
less-loader: 这个文件就是将 less 转成 css

```js
let less = require("less");
// less 转css 功能
function loader(source) {
    let css = "";
    // 可以在Node中调用编译器  '.class { width: (1 + 1) }'  输出 .class { width: 2 }
    less.render(source, function (err, c) {
        css = c.css;
    });
    return css;
}
module.exports = loader;
```

css-loader: 处理 @import 和 url 下面写的是简单的处理 url，@import 替换成 require(“-!css-loader!./global.css”) 来处理。

> 返回的是一个数组，数组中是 js 代码片段（字符串），需要动态的执行 js，style-loader 无法将 js 字符串转化成 css 代码。下面截图是 css-loader 返回的 js 代码片段，为了获取 CSS 样式，我们会在 style-loader 中直接通过 require 来获取，可以通过 pitch 方法来实现。

```js
// 转义@import 或者 url ('') 转移成require
function loader(source) {
    // 匹配url
    let reg = /url\((.+?)\)/g;
    let pos = 0;
    let current;
    let arr = ["var list=[]"];
    // reg.exec(source)这个返回结果打印我们可以知道 是一个数组  0和1 对应的是“url('./mmm.jpg')”  “'./mmm.jpg'”
    while ((current = reg.exec(source))) {
        // 将代码分开 单独 将url('./mmm.jpg') 转成require
        let [a, b] = current;
        let last = reg.lastIndex - a.length;
        arr.push(`list.push(${JSON.stringify(source.slice(pos, last))})`);
        pos = reg.lastIndex;
        // 将./mmm.jpg 转成require
        arr.push(`list.push('url('+require(${b})+')')`);
    }
    // 将剩余代码放进去
    arr.push(`list.push(${JSON.stringify(source.slice(pos))})`);
    arr.push(`module.exports = list.join("")`);
    console.log(arr.join("\r\n"));
    return arr.join("\r\n");
}
module.exports = loader;
```

![](https://img-blog.csdnimg.cn/20210126102342852.png)  
style-loader: 将 css 内联到 html 中。

```js
let loaderUtils = require("loader-utils");
// css内联
function loader(source) {
    console.log(source);
    return loader;
}

// remainingRequest 剩余的请求  less-loader!css-loader!./index.less
loader.pitch = function (remainingRequest) {
    // remainingRequest 打印发现是一个绝对路径需要处理
    // loaderUtils.stringifyRequest 这个方法能将remainingRequest 处理成相对路径，通过行内loader处理好css文件后返回的就是css-loader中的数组代码片段，通过require的方式引入，就能拿到他的css代码片段
    let str = `
        let style=document.createElement('style');
        style.innerHTML=require(${loaderUtils.stringifyRequest(
            this,
            "!!" + remainingRequest //行内loader去处理 css
        )})
        document.head.append(style)
    `;
    return str;
};

module.exports = loader;
```

上面说过了 pitch 这个方法，有返回值的话 ，将会被阻断。正常的顺序  
style-loader-pitch=>css-loader-pitch=>less-loader-pitch=>less-loader=>css-loader=>style-loader;

> 现在在第一步 style-loader-pitch 有返回值，直接阻断，自己都不会执行，直接执行 pitch 中的内容，里面通过 require 的方式使用行内 loader 的处理方式重新链式处理 css，!! 是为了防止死循环，只执行这里的 Loader 的意思，最后成功，下面的 url，是因为我配置了 url-loader 转成 base64 了

![](https://img-blog.csdnimg.cn/20210126104113690.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTM1NTEy,size_16,color_FFFFFF,t_70)  
思考：我这里有个问题，为什么执行顺序 不能正常进行，一定要用 style-loader 中的 pitch 方法。  
主要是还是 css-loader 这里返回的是 js 代码片段，因为这里处理 css 的代码片段，需要拼接字符串，最后再合并，直接输出这个字符串（尝试过，发现他的换行符 / n 会比较乱，最终也没找到实现的方法）有明白得可以给我留言哈。
