牙医教你 450 行代码自制编程语言
-----------------------------

# Description

没有系统学习过编译原理的同学可能会很好奇编程语言的编译器, Lexer & Parser, 虚拟机是怎么实现的. 而又苦于系统性的教材过于枯燥.  

本教程教大家用 450 行 Go 代码实现一个简单的编程语言, 它的语法是这样的:  

```php
$a = "pen pineapple apple pen."
print($a)
```

看上去很简单是不是? 但是它包含了个手写的递归下降解析器和一个简单的解释器.   

虽然该语言甚至不是图灵完备的. 但写这个语言和教程的主要目的是让编译原理初学者有一个预热, 简单了解一个编程语言是怎么构建的.  

准备好了吗? 让我们开始!  

# PDF 

- [戳这里直接看PDF版本](./pdf/write-a-programming-language-in-450-lines.pdf)

# Markdown

或者你更喜欢看用 Markdown 编写的原始版本:

- [牙医教你 450 行代码自制编程语言 - 1, 从 EBNF 开始](./DOCUMENTS/part-1-start-from-ebnf/part-1-start-from-ebnf.md)
- [牙医教你 450 行代码自制编程语言 - 2, 两个魔法就可以实现永动机](./DOCUMENTS/part-2-two-magic/part-2-two-magic.md)
- [牙医教你 450 行代码自制编程语言 - 3, 实现 Lexer 上篇](./DOCUMENTS/part-3-create-a-lexer/part-3-create-a-lexer.md)
- [牙医教你 450 行代码自制编程语言 - 4, 实现 Lexer 下篇](./DOCUMENTS/part-4-create-a-lexer/part-4-create-a-lexer.md)
- [牙医教你 450 行代码自制编程语言 - 5, 递归下降语法解析器](./DOCUMENTS/part-5-parser/part-5-parser.md)
- [牙医教你 450 行代码自制编程语言 - 6, 后端](./DOCUMENTS/part-6-backend/part-6-backend.md)
- [牙医教你 450 行代码自制编程语言 - 7, 后续该如何学习编译原理](./DOCUMENTS/part-7-how-to-learn/part-7-how-to-learn.md)

# Author

[karminski-牙医](https://github.com/karminski)  

# License 

[CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)

您可以自由地:  

共享 - 在任何媒介以任何形式复制, 发行本作品  

只要你遵守许可协议条款, 许可人就无法收回你的这些权利.  

惟须遵守下列条件： 

署名 - 您必须给出适当的署名, 提供指向本许可协议的链接, 同时标明是否 (对原始作品) 作了修改. 您可以用任何合理的方式来署名, 但是不得以任何方式暗示许可人为您或您的使用背书.  

非商业性使用 - 您不得将本作品用于商业目的.  

禁止演绎 - 如果您 再混合, 转换, 或者基于该作品创作, 您不可以分发修改作品.  

