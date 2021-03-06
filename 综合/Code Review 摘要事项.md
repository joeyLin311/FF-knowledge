---
date created: 2021-12-09 22:57
date updated: 2021-12-17 16:05
---

#日常记录
Code Review 存在的意义在于：

- **让团队其它成员熟悉代码**。这样保证了其他人都可以快速上手工作、解决代码问题
- **提高代码质量**。这点毫无疑问，一方面是主动地提升代码质量，比如代码需要被其他人 review，会自觉地提高代码质量，另一方面，我们可以学习和发现他人代码中优秀的写法和发现对方存在的不足，提醒自己避免出现类似错误。
- **检查或提高新成员的编程水平。**新人加入时，对项目代码、业务、编码风格还不熟悉，通过 Code Review 可以更好地帮助新人融入团队，保持代码的一致性，也是一个很好的技术交流相互学习的机会。

## 规范方面

- 是否易于理解
- 是否遵循编码规范
- 是否有无用的注释代码
- 函数或者类太大吗？如果是，函数或者类是否承担太多职责
- 行为是否统一？比如缓存是否统一，错误处理是否统一，错误提示是否统一，弹出框是否统一等等
- 同一逻辑、同一行为有没有走同一的 Code Path？
  - 低质量程序的另一个特征是：同一行为、同一逻辑，因为出现在不同的地方或者被不同的方法触发，没有走同一 Code Path 或者各处有一份 copy 的实现，导致非常难以维护

## 设计方面

- 职责是否单一？
  - 这是经常被违背的原则。一个函数或类只能做一件事情，比较常见的违背是一个函数或类既做了 UI 的事情，又做了逻辑的事情。
- 代码有没有对其它模块强耦合？
- 代码是否重复？
  - 主要看有没有把公用组件、可重复的代码、函数这些可抽象的给抽象出来
- 代码易于扩展吗？
  - 要面向接口编程而不是面向实现编程。主要就是看有没有进行合适的抽象，把一些行为抽象为接口
- 代码足够健壮吗？
  - 对一些边界条件有没有考虑完整，逻辑是否健壮，有没有存在的 bug

## 安全方面

- 有没有内存泄漏风险？
- 有没有考虑线程安全性、数据访问的一致性？
- 有没有很好的 Error Handling？比如网络错误、IO 错误
- 新的改动是通过打补丁让代码质量继续恶化，还是对代码质量进行了修复？

## 性能方面

- 客户端程序对频繁消息和较大数据等耗时操作是否处理得当？
- 关键算法的时间复杂度是多少？有没有可能存在的性能瓶颈？
