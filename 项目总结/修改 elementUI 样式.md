
#### 1. 新建全局样式表

新建 global.css 文件，并在 main.js 中引入。 global.css 文件一般都放在 src->assets 静态资源文件夹下的 style 文件夹下，在 main.js 的引用写法如下：

```
import "./assets/style/global.css";


```

在 global.css 文件中写的样式，无论在哪一个 vue 单页面都会覆盖 ElementUI 默认的样式。

#### 2. 在当前 vue 单页面中添加一个新的`style`标签

在当前的 vue 单页面的`style`标签后，添加一对新的`style`标签，新的`style`标签中不要添加`scoped`属性。在有写`scoped`的`style`标签中书写的样式不会覆盖 ElementUI 默认的样式。

#### 3. 使用 `/deep/` 深度修改标签样式

找到需要修改的 ElementUI 标签的类名，然后在类名前加上 `/deep/` ，可以强制修改默认样式。这种方式可以直接用到有 `scoped` 属性的 style 标签中。

```
// 修改级联选择框的默认宽度
/deep/ .el-cascader {
  width: 100%;
}


```

#### 4. 通过内联样式 或者 绑定类样式覆盖默认样式

通过内联样式 style ，绑定类样式的方式，可以在**某些标签**中可以直接覆盖默认样式，不是很通用。具体实例如下：  
内联样式`style`的方式：

```
<el-button :style="selfstyle">默认按钮</el-button>
<script>
    export default {
      data() {
        return {
            selfstyle: {
                color: "white",
		marginTop: "10px",
		width: "100px",
		backgroundColor: "cadetblue"
            }
        };
      }
    }
</script>


```

通过绑定修改样式方式修改：

```
<el-button :class="[selfbutton]">默认按钮</el-button>
<script>
  export default {
    data() {
      return {
        selfbutton: "self-button"
      };
    } 
  }
</script>
<style lang="stylus" rel="stylesheet/stylus" scoped>
.self-button {
    color: white;
    margin-top: 10px;
    width: 100px;
    background-Color: cadetblue;
}
</style>


```

#### 5. 总结

第一种全局引入 css 文件的方式，适合于对 elementUI 整体的修改，比如整体配色的修改；  
第二种添加一个 style 标签的形式，也能够实现修改默认样式的效果，但实际上因为是修改了全局的样式，因此在不同的 vue 组件中修改同一个样式有可能会有冲突。  
第三种方式通过 /deep/ 的方式可以很方便的在 vue 组件中修改默认样式，也不会于其他页面有冲突。  
第四种方式局限性比较大，可以使用，但不推荐使用。

--------------------------------- 分割线 - 7 月 20 日更新 ----------------------------------------

#### elementUI 修改样式介绍内容更新

第三种方法 `/deep/` 更新：

今天在做样式修改的时候，突然发现谷歌浏览器报了一个警告，说 ** /deep/ combinator is no longer supported in CSS dynamic profile.** 保险起见我决定不用 `/deep/` 这种方法来修改 elementUI 的样式了。

一番搜寻得知，可以使用 `>>>` 来深度修改样式。如下面的例子：

```
<style scoped>
  .a >>> .b { /* ... */ }
</style>


```

上面的代码将会被解析成如下格式，可在浏览器中查看：

```
.a[data-v-f3f3eg9] .b { /* ... */ }


```

**但是，** 粗心的我一开始没注意在 `<style> </style>` 标签中并没有声明类似于 less，scss 等预处理语言，因此：** 这种 `>>>` 方式只能用在原生 CSS 语法中，不能在 css 预处理器如 less scss 等直接使用 **

**如何在 css 预处理器中使用 `>>>` 深度修改 elementUI 样式呢？**  
`用变量代替 >>> 符号`，如下代码示例：

```
<style scoped lang='less'>
    @deep: ~'>>>';
      .box {
        @{deep} .title {
            ...
        }
      }
</style>
**~ 表示转义**
转义允许您将任意字符串用作属性或变量值。除插值外，里面的任何东西 ~"anything" 或 ~'anything' 原样使用。
```css
.weird-element {
  content: ~"^//* some horrible but needed css hack";
}


```

编译为以下内容：

```
.weird-element {
  content: ^//* some horrible but needed ss hack;
}


```

**当然，我们也可以在全局样式表中为 `>>>` 取别名，那么就可以直接在页面任何 style 标签中使用其别名如 @{data} 来修改页面样式了**

**注意：**我在实际中发现，多个 @{data} 可以同级使用，但不能相互嵌套，否则将不会生效。如下图，虽然 `el-input__inner` 在 `el-input` 标签内部，但却不可以直接嵌套使用。这一部分内容，参考[这篇文章](https://blog.csdn.net/mochenangel/article/details/105571447)  
![](https://img2020.cnblogs.com/blog/2091839/202007/2091839-20200720230112118-1968793676.png)

参考文章：  
[https://blog.csdn.net/bamboozjy/article/details/81629381](https://blog.csdn.net/bamboozjy/article/details/81629381)  
[https://blog.csdn.net/zeping891103/article/details/84961225](https://blog.csdn.net/zeping891103/article/details/84961225)  
[https://blog.csdn.net/weixin_42204698/article/details/101757080](https://blog.csdn.net/weixin_42204698/article/details/101757080)