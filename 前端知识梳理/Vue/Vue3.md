# Vue3.x
## 一.新特性
### 1.Composition Api
组合式Api
-   替代mixins混入方案，避免命名冲突和变量的来历不清楚
-   助推复杂业务逻辑用例
-   自然完善的TS支持
-   为封装解耦组织业务逻辑而生
### 2.支持TS
-   源码开发 - Internal
    -   更好的编辑器支持 - 重构代码更加的方便而安全
    -   提高代码贡献者提交信心
-   业务编写 - External
    -   满足日益增长的TS Vue使用者需求
    -   自动类型推断 - 一致性检查 降低维护成本
### 3.Tree-shaking-aware 
global API 全局API可按需剪裁

### 4.编译推断型虚拟Dom - 静态提升
静态虚拟DOM不加入计算比较
只保留必要的动态部分的渲染功能，减少了运行时的运算量。

### 5.Fragment
支持代码碎片，比如在组件中渲染一个弹窗，弹窗在根节点上而不组件上，这样就不用考虑z-index的层级