# AST: 抽象语法树
## 一.解析
### 1.acorn
用 JavaScript 实现的、小型且快速的 JavaScript 解析器
### 工作流程
- 词法分析
- 语法分析
- 得到AST


![[acorn 工作流程图.png]]

**acorn 将词法分析和语法分析交替进行，只需要扫描一遍代码即可得到最终 AST 结果**。

-   type 表示当前 Token 类型；
    
-   pos 表示当前 Token 所在源代码中的位置；
    
-   startNode 方法返回当前 AST 节点；
    
-   nextToken 方法从源代码中读取下一个 Token；
    
-   parseTopLevel 方法实现递归向下组装 AST 树。

### 词法分析
1.  明确需要分析哪些 Token 类型。
-   关键字：import，function，return 等
    
-   变量名称
    
-   运算符号
    
-   结束符号

2.状态机：简单来讲就是消费每一个源代码中的字符，对字符意义进行状态机判断

## 其他
[AST explorer](https://astexplorer.net/)，在这个平台中，可以实时看到 JavaScript 代码转换为 AST 之后的产出结果