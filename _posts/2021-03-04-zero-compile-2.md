---
title: '从零开始编译器-2'
layout: post
subtitle: ''
tags:
  - SimpleLang, compiler
---

# 语法分析

## 什么是语法分析

语法分析把词法分析的结果组装成一颗语法树，我也简单介绍下，同样如果你懂编译原理的画可以跳过这一段。

比方说上面词法分析提到的代码：

```c
int a = 1234 + b;
```

经过词法分析以后，会转换为符号串（见词法表1），而语法分析就是分析这串符号串是否符合语法。

上面这段变量定义的语法我们这么定义

```
类型 变量 [= 表达式] ;
```

*中括号表示可选*

**类型**允许是**类型保留字**，或者**单词**（针对自定义类型）

**变量**允许是**单词**

**表达式**允许是**数字**、**单词**、或者通过符号连接的**表达式**

注意这里规则有个比较特别的地方，表达式的定义里出现了自己，这是因为语法定义中，表达式必须在特定的地方出现，所以语法分析器是可以通过上下文分析出递归的符号的，因此我们可以在语法规则里用递归。

## 语法分析工具: bison

我用的语法分析工具是 bison, 它一般和 flex 一起使用，同样是把你定义的语法文件翻译为 c 文件，你可以把它生成的文件加入你的编译器工程，你就完成了语法分析工作。

bison 文件的第一部分，是通过 `%{  %}` 括起来的C/C++源码，同样会加入生成文件的头部。

第二部分是一个定义：

```
%union{
	AstType		*tp;
	AstNode		*expr;
	char*		str;
	int			type;
}
```

这实际上是定义了一个 union 的临时变量，保存当前的节点数据。

接下来通过 %right, %left 来定义操作符：

```
%right '=' 
%left '+' '-'
%left '*' '/'
```

%right 表示右结合，会先匹配右方，因此一般用在 = 这种赋值操作上，右方的表达式计算完毕，才最后赋值到左边，%left 当然是相反的。

特别的，这里定义的顺序是有影响的，优先级低的写在前面，优先级高的写在下面，上面加减在前乘除在后，就能实现表达式里先乘除后加减了。

再接下来，就是 Token 的定义

```
%token 			FUNC DOTS '}'
%token<type>	ITYPE 
```

这实际上是给词法分析器返回的 Token 定义分类，这个分类在生成的 C 代码里实际是一个 `enum yytokentype` ，因此按 C 的命名习惯用了全大写，而它从256开始定义，因此也可以用 `%token '}'` 这样的代码把右括号这样的字符单独分类，不会冲突，如果这个Token有额外数据，还要指示它被保存在上面 union临时变量的那个字段里，比如 `%token<type>	ITYPE ` 表示 ITYPE 类型的额外数据被保存在 int 型的 type 字段里。 

再然后我们通过 `%type<expr>	def` 这样的代码来指定语法节点要把额外数据保存在那个字段里，好处就是下面如果出现了被定义的节点，我们不需要临时指定字段，否则就需要通过 `$<expr>1` 这样的形式每次单独指定这个节点的存储字段（其实我后来倒是觉得全部临时指定代码可读性更高些？）。

定义完成后用 `%%` 分割，下面写语法规则。

语法规则我们一般使用[巴科斯范式](https://baike.baidu.com/item/%E5%B7%B4%E7%A7%91%E6%96%AF%E8%8C%83%E5%BC%8F/1849549?fromtitle=BNF&fromid=7328753&fr=aladdin)来表示，而我使用的语法分析工具 bison 用接近巴科斯范式的语法来定义语法，比如上面这个 c 语言变量定义的语法定义大致是这样：

```
%type<expr> def

%%

def
    : type IDENTIFIER			{ $$=makeType($2); }
    | type IDENTIFIER = expr	{ $$=makeType($2, $4); }
    ;
type
    : TYPEKEY		
    | IDENTIFIER
    ;
expr 						// 它的定义里出现了自己
	: NUMBER							// 数字不用做处理，所以不用写代码，可以理解为 {$$=$1;}
	| expr operator_binary expr			{ $<expr>$=makeBinary($<type>2,$<expr>1,$<expr>3); }
	;
operator_binary
	: '+'	{ $<type>$='+'; }
	| '-'	{ $<type>$='-'; }
...
    
```

大括号里同样是当完成匹配时需要做的事情，$$ 表示当前节点。

而上面用到的函数在 C/C++ 里定义，在 bison 文件的第一部分 %{ %} 里 #include 进来，比如上面的 `makeBinary` 函数这样定义:

```c
AstNode* makeBinary( int op, AstNode* left, AstNode* right );
```

类型必须和成员变量的类型对应上，因为生成代码实际就是把这个节点的成员变量往里塞。需要注意的是，由于临时变量用完就会被改写，所以你必须马上把你需要的内容复制下来，否则过了就没了。

写完语法规则就再用 `%%` 分割最后的块，可以在块里写 C/C++ 代码，同样会附在生成文件的尾部。

好了，现在我们已经学会了语法分析器怎么弄，让我们来试一试写自己语言的语法分析器吧！（加减乘除都学会了，开始解微积分吧！）

*（未完待续)*

