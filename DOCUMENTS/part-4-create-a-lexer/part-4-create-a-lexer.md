牙医教你 450 行代码自制编程语言 - 4, 实现 Lexer 下篇.md
----------------------------------------------------

@version    20210104:1  
@author     karminski <work.karminski@outlook.com>  


上一篇  [牙医教你 450 行代码自制编程语言 - 3, 实现 Lexer 上篇](https://zhuanlan.zhihu.com/p/341840788), 讲了 Lexer 的 Token 识别方法, 接下来我们就要把它拼装起来了!

本教程的所有代码都可以在 [https://github.com/karminski/pineapple](https://github.com/karminski/pineapple) 找到.  


再次推荐这本书, 本教程就是类似这本书的简化版本, 想要仔细学习的话可以考虑看原作:  

- [《自己动手实现Lua：虚拟机、编译器和标准库》](https://union-click.jd.com/jdc?e=jdext-1331174943460048896-0&p=AyIGZRhfEQAUAlEZWBAyEgZUGF4SAhIFUBJaEQQiQwpDBUoyS0IQWhkeHAxfEE8HCllHGAdFBwsCEwZWHlwVAhACXBpfEx1LQglGa2lVWnpcTwhRYXZHBkIzFXRIXT1jGHUOHjdVElsXChMGVRxYJQITBlUfXhYBFAZlK1sQMlNpXBhdFAUaN1QrWxICEwdRHFIXCxYPUitbHQYi0fuPjp29y7fwzfG715%2B3gJLwwbyUN2UrWCVZR1McXkcVABAHVR1eHQcQAlIaWhALGw9SB1olAhMGVx9ZFAUaBzseWxQDEwNdGlkXbBAGVBlaFAAVAVYrWyUBIlk7GggVUhVVAEw1T1lTBxAeWxdsEgRdHFwRBBA3VxpaFwA%3D)


# 糖糖糖!

到目前为止, 我们已经实现了 Lexer 的核心功能, 接下来我们需要一些糖, 将他们包裹起来, 作为一种抽象, 帮助我们后续更好地构建 Parser.  

抽象能力是很重要的能力, 他能直接能决定构建上层逻辑的复杂度. 抽象得越好, 上层实现越简洁, 越易于理解. 原作这一点实现的很好. 好的代码是易于理解的, 而不是只有编写者自己才能看懂的.  

我们首先要实现的就是 ```GetNextToken()```:   

```go
func (lexer *Lexer) GetNextToken() (lineNum int, tokenType int, token string) {
    // next token already loaded
    if lexer.nextTokenLineNum > 0 {
        lineNum                = lexer.nextTokenLineNum
        tokenType              = lexer.nextTokenType
        token                  = lexer.nextToken
        lexer.lineNum          = lexer.nextTokenLineNum
        lexer.nextTokenLineNum = 0
        return
    }
    return lexer.MatchToken()

}
```

该函数其实就是上一篇中 ```MatchToken()``` 的封装, 返回值是一样的.   

但不同之处在于, 我们判断了一下 ```nextTokenLineNum``` 的值, 这个值表面上是下一个 Token 的行号. 但实际上, 这个值一旦大于 0, 即意味着 nextTokenType 已经加载了, 我们获取下一个 Token 的时候 (即调用 ```GetNextToken()```) 就可以直接返回上次已经加载好的值. 避免了再次调用 ```MatchToken()``` 浪费性能.  

举例来讲, 比如我们调用了 ```LookAhead()``` 来获取下一个 Token 是什么, 判断是我们要的 Token 之后, 必然会使用 ```NextTokenIs(TOKEN)``` 进行断言.  

例如解析 Variable (```Variable ::= "$" Name Ignored```), 需要先判断是美元符号 ```$```, 然后是 ```Name```:  

```go
token := LookAhead()
if token == TOKEN_VAR_PREFIX {
    NextTokenIs(TOKEN_VAR_PREFIX)
    parseName()
}
```

这样在 ```LookAhead()``` 中调用了 ```GetNextToken()```, 调用 ```NextTokenIs(TOKEN_VAR_PREFIX)``` 的时候, 就不用再次调用 ```GetNextToken()``` 了. 存储了上一次的执行结果, 提升了性能.  


# 最后三个函数  

至此, 所有的基础函数做完了, 我们要实现三个用于抽象的函数. 首先是 ```NextTokenIs()```:  

```go
func (lexer *Lexer) NextTokenIs(tokenType int) (lineNum int, token string) {
    nowLineNum, nowTokenType, nowToken := lexer.GetNextToken()
    // syntax error
    if tokenType != nowTokenType {
        err := fmt.Sprintf("NextTokenIs(): syntax error near '%s', expected token: {%s} but got {%s}.", tokenNameMap[nowTokenType], tokenNameMap[tokenType], tokenNameMap[nowTokenType]) 
        panic(err)
    }
    return nowLineNum, nowToken
}
```

这个函数用于断言下一个 Token 是什么. 并且由于内部执行了 ```GetNextToken()```, 所以游标会自动向前移动.  

然后是 ```LookAhead()```:  

```go
func (lexer *Lexer) LookAhead() int {
    // lexer.nextToken* already setted
    if lexer.nextTokenLineNum > 0 {
        return lexer.nextTokenType
    }
    // set it
    nowLineNum                := lexer.lineNum
    lineNum, tokenType, token := lexer.GetNextToken()
    lexer.lineNum              = nowLineNum
    lexer.nextTokenLineNum     = lineNum
    lexer.nextTokenType        = tokenType
    lexer.nextToken            = token
    return tokenType
}
```

我们也见过很多次了, 这个函数用于返回下一个 Token 是什么, 不过它并不会将游标向前移动 (准确的说是移动了, 然后又移了回来, 在 ```lineNum, tokenType, token := lexer.GetNextToken()``` 后面那一堆代码.).  

头部的 ```if lexer.nextTokenLineNum > 0 {``` 则像我们之前说的, 检测是否曾经获取过下一个 Token, 如果获取过, 直接返回 ```lexer``` 结构体内部缓存好的结果.  

最后是 ```LookAheadAndSkip()```:  

```go
func (lexer *Lexer) LookAheadAndSkip(expectedType int) {
    // get next token
    nowLineNum                := lexer.lineNum
    lineNum, tokenType, token := lexer.GetNextToken()
    // not is expected type, reverse cursor
    if tokenType != expectedType {
        lexer.lineNum              = nowLineNum
        lexer.nextTokenLineNum     = lineNum
        lexer.nextTokenType        = tokenType
        lexer.nextToken            = token
    }
}
```

它相对于 ```LookAhead()``` 多了个参数, 仍然是先看一下下一个 Token 是什么, 如果跟输入的相同就会跳过去, 如果不同则不会跳过去. 这个函数是为了 ```Ignored``` Token 特别定制的.   


# 趁热打铁

好的, 接下来我们来迅速熟悉下我们定义的这些函数的使用方法, 我们直接来看 ```Print``` 语句是如何解析的:  

```go
// Print ::= "print" "(" Ignored Variable Ignored ")" Ignored
func parsePrint(lexer *Lexer) (*Print, error) {
    var print Print 
    var err   error

    print.LineNum = lexer.GetLineNum()
    lexer.NextTokenIs(TOKEN_PRINT)
    lexer.NextTokenIs(TOKEN_LEFT_PAREN)
    lexer.LookAheadAndSkip(TOKEN_IGNORED)
    if print.Variable, err = parseVariable(lexer); err != nil {
        return nil, err
    }
    lexer.LookAheadAndSkip(TOKEN_IGNORED)
    lexer.NextTokenIs(TOKEN_RIGHT_PAREN)
    lexer.LookAheadAndSkip(TOKEN_IGNORED)
    return &print, nil
}
```

我们先不要管看不懂的细节, 直接来看使用了我们刚才定义的那些函数的部分, 首先 ```Print``` 语句的开头必须是字符 ```"print"```, 我们已经定义好了 Token (```TOKEN_PRINT```), 对于这种确定的场景, 直接用 ```NextTokenIs()``` 来进行断言即可.  

同样, 然后的左括号 (```TOKEN_LEFT_PAREN```) 也直接断言, 然后就是 ```Ignored``` 了, 我们的语法允许函数里面有空格或者换行. 即这样写也是合法的:  

```php
print(
    $a
)
```

那么, 这里我们就需要用 ```LookAheadAndSkip(TOKEN_IGNORED)``` 来判断接下来是不是 ```Ignored```, 如果是就跳过, 如果不是就不跳过.  

剩下就没有新鲜的了. 我们已经重复很多次了.  

# 到此为止 

全部的 lexer.go 文件见 Github: [https://github.com/karminski/pineapple/blob/main/src/lexer.go](https://github.com/karminski/pineapple/blob/main/src/lexer.go).    


