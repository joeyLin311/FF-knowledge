## plop 简介

Plop 是一个基于 nodeJS 开发的小工具, 主要作业婆娘个是根据模板代码生成目标代码, plop 对于模板代码的处理选择了 `handlebars` 作用模板

**作用是通过自动化工具减少开发中重复代码的书写**

一般我们使用 plop 是继承在某个项目中, 用来自动化创建一些同类型的, 同HTML结构的页面文件

>举个例子

```bash
在多页面开发的情况下, 经常会碰到相同的文件目录或者相同结构的页面文件. 我们用通过命令行去快速生成相关的 html, css, js 文件, 甚至是通用的公共文件引入. plop 主要有点我觉得在于对现有已经搭建完成的项目进行非侵入性的增量修改: 比如我们并不需要整体重新去搭建一个够, 而是在项目基础上集成添加公共模板文件的生成
```

### plop 使用

安装完成之后, 在项目根目录中创建 `plopfile.js` 在该文件中, 我们可以使用 `plop` 的相关API 编写相关文件的生成函数.

例子:

```jsx
module.export = plop => {
  // plop 提供了生成页面文件的API
  // 第一个参数是 `Generator` 命令名称
  plop.setGenerator('my-page', {
    description: '使用plop创建文件命令', // 描述命令
    prompts: [{
      type: 'input', // 用来在终端输入文本
      name: 'name', // 在 actios 内 可以通过{{name}} 取到这里输入的值，在模板内也可以通过{{name}}取到这个值
      message: '请输入函数组件的名字（必填）', // 会显示在控制台
      validate(val) {
        // 如果验证不通过，需要返回字符串，字符串将会被展示在控制台
        // 验证通过，则返回true
        return true;
      },
      default: "default", // 默认值
    }],
    actions:[{
        // 添加文件 src/components/{{name}}/index.js 文件内容为templateFile指定的内容
          type: 'add',
          path: 'src/components/{{name}}/index.js',
          templateFile: 'plop-templates/functionComponentWithLessPage.hbs',
        // 如果模板文件内容少，可以直接使用template指定
        // template:""
        // 在将文件输出到硬盘前会调用transform函数
        // transform:()=>""
        // 当运行这个actions，data会合并到prompt answers。
        // data:{}
        //...其他不常用参数请查看文档
    }，{
      // 修改src/index.js文件中的内容，将 pattern 匹配到的内容，修改为 template 的内容
      type: 'modify',
      path: 'src/index.js',
      pattern: /(\/\* injectinfo \*\/)/,
      template: '$1\nconsole.log(111);',
    },{
    // 修改src/index.js文件中的内容，在 pattern 匹配到的内容后追加 template 的内容
      type: 'append',
      path: 'src/route.config/index.js',
      pattern: /\/\* inject \*\//,
      template: 'console.log(666);',
    }],
  })
}

```

### prompts

配置输入类型, 可以是 `input` `number` `confirm` `list` `rawlist` `expand` `checkbox` `passwrod` `editor` , 并且支持 `inquirer` 插件.

### actions

- `actions` 数组内的type, 可选: `ADD` `AddMany` `Modify` `Append`
- 如果 `actions` 需要根据 `prompts` 的 answer 来决定, actions 可以接受一个参数, 实现动态action

```jsx
actions(data) {
  if(data.needLess) {
    return [...something]
  } else {
    retrurn [...others]
  }
}
```

## 创建模板

```html
<!DOCTYPE html>
<html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <link href="./{{name}}.css">
    </head>

    <body>
        <p>{{name}}</p>

    </body>

</html>
```

使用:  `npm plop <name>`  携带的参数就会进入模板文件绑定的变量中
