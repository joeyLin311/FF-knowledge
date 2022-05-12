---
date created: 2022-05-08 23:08
---

## Path 常用API

### basename() 获取路径中的基础名称

```js

console.log(path.basename(__filename))
// 输出: xxx.js  返回具体文件名
console.log(path.basename(__filename, '.js'))
// 第二个参数匹配相关文件后缀名, 不带格式
// 输出: xx
console.log(path.basename('/a/b/c'))
// 输出path的最后一个部分

```

1. 返回的就是接受路径当中的最后一部分
2. 第二参数(可选) 接收扩展名, 如果说没有设置则返回完整的文件名称带后缀
3. 第二个参数作为后缀时, 如果没有在当前路径中匹配到, 那么就会忽略
4. 处理目录路径的时候, 如果参数结尾有分隔符, 那么也会被忽略

### dirname() 获取路径中的目录名(路径)

```js
console.log(path.dirname(__filename))
// C:/Desktop/xxx/xxx
```

1. 返回路径中最后一个部分的上一层目录所在路径

### extname() 获取路径中的扩展名称

```js
console.log(path.extname(__filename))
// 输出: .js  返回文件格式
```

1. 返回 path 路径中相应文件的后缀名
2. 如果 path 路径中存在多个点, 它匹配的是最后一个点到结尾的内容

### isAbsolute() 获取路径是否为绝对路径

### join() 拼接多个路径片段

### resolve() 返回绝对路径

### parse() 解析路径
1. 接受一个路径, 返回一个对象, 包含不同的信息
2. `root`:根  `dir`: 最后一个部分的上一层目录 `base`: 后面完整的内容部分, `ext`: 扩展名 `name`: 文件名称
### format() 序列化路径

### normalize() 规范化路径
