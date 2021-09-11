## 哈希也有讲究

webpack给我们提供了三种哈希值计算方式，分别是hash、chunkhash和contenthash。那么这三者有什么区别呢？

-   hash：跟整个项目的构建相关，构建生成的文件hash值都是一样的，只要项目里有文件更改，整个项目构建的hash值都会更改。
-   chunkhash：根据不同的入口文件(Entry)进行依赖文件解析、构建对应的chunk，生成对应的hash值。
-   contenthash：由文件内容产生的hash值，内容不同产生的contenthash值也不一样。

显然，我们是不会使用第一种的。改了一个文件，打包之后，其他文件的hash都变了，缓存自然都失效了。这不是我们想要的。

那chunkhash和contenthash的主要应用场景是什么呢？在实际在项目中，我们一般会把项目中的css都抽离出对应的css文件来加以引用。如果我们使用chunkhash，当我们改了css代码之后，会发现css文件hash值改变的同时，js文件的hash值也会改变。这时候，contenthash就派上用场了。