---
date created: 2022-05-08 23:28
date updated: 2022-05-09 00:16
---

## 概念

Buffer 让 JS 可以操作二进制

### 二进制数据

NodeJS 平台下 JS 可实现 IO 操作, 处理的就是二进制数据

### 流操作 Stream

数据流操作, 配合管道实现数据分段传输

### Buffer

- 无须 require 的一个全局变量
- 实现 NodeJS 平台下的二进制数据操作
- 不占据V8内存空间
- 内存的使用由Node控制, 由V8的GC回收
- 一般配合 Stream 流使用, 充当数据缓冲区

## 创建 Buffer

Buffer 是 NodeJS 的内置类

### 三种创建方法(API)

- alloc: 创建指定字节大小的 buffer
- allocUnsafe: 创建指定大小的 buffer (不安全)
- form: 接收数据, 创建 buffer

## Buffer 实例方法

### fill: 使用数据填充buffer

### write: 向 buffer 中写入数据

### toString: 从 buffer 中提取数据

### slice: 截取 buffer

### indexOf: 在 buffer 中查找数据

### copy: 拷贝 buffer 中的数据

## Buffer 静态方法

### concat: 将多个 buffer 拼接成一个新的 buffer

### isBuffer: 判断当前数据是否为 buffer

## 实现 Buffer.split API
- [ ] 写一个算法