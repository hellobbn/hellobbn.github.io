---
title: Write a simple compiler [Part 3]
date: 2020-04-29 1
category:
  - Programming
tags:
  - Compiler
  - LLVM
---

Building a C- compiler using bison, flex and LLVM, Part 3: Parser

<!-- more -->

A syntax analyzer analyzes code aginst the production rules to detect errors. In this part we take the input from the lexical analyzer in Lab1 and uses bison to construct the syntax tree.

## 1. Bison

As stated in `bison`'s manual:

> Bison is a parser generator in the style of _yacc_.

And we are going to use `bison` to generate our parser, construct the syntax tree for our program.

### 1.1 Format

From `bison`'s manual, an input file's structure is like the following:

```yacc
%% {
  C Declarations
% }
  Bison Declarations
%%
  Grammar Rules
%%
  Additional C code
```

The main part of using bison for syntax analyze is the Grammar Rules part which defines the grammar.

### 1.2 C Declarations

The _C Declarations_ part defines functions and variables from `flex` like `yylex()`, `yyparse()`, etc... It also includes files needed to construct a syntax tree.

### 1.3 Bison Declarations

This part defines a `%union` which is used as `yylval` in the Lexer code. __\[1\]__

It also defines `%token` (terminal symbol) and `%type` (non-terminal symbol). And it should be noted that we are going to use those token for the Lexer!! I will talk about this part later.

### 1.4 Grammar rules

This part defines fules for the parser. And we are going to construct Syntax tree in the code section of this part.

Take function declaration for the C- as an example:

```c
/** A simple function declaration:
 * int -- token: type-specifier
 * main -- token: identifier
 * ( -- token: left parenthese
 * void -- token: params
 * ) -- token: right parenthese
 * { [...] } -- token: compound-stmt
 */
int main(void) {

}
```

A grammar rule for bison looks like this:

```c
fun-declaration: type-specifier tok_identifier tok_l_parenthese params tok_r_parenthese compound-stmt // we specify the grammar rules
    {
        $$ = newSynTreeNode("fun-declaration"); // new tree node
        // insert child node below
        synTreeNodeAddChild($$, $1);
        synTreeNodeAddChild($$, $2);
        synTreeNodeAddChild($$, $3);
        synTreeNodeAddChild($$, $4);
        synTreeNodeAddChild($$, $5);
        synTreeNodeAddChild($$, $6);
    };
```

And for the lexer, we need to make tree node for each token. For example, in the lexer file:

```c
"while" {
  #ifndef LEX_TEST
    yylval.pnode = newSynTreeNode("while");
  #endif
    return tok_while;
}
```

We use _$?_ to represent the child node because the node is already allocated for them:

1. For the non-terminal symbol, the node is already allocated before during parsing.
2. For the terminal symbol, the node is allocated in the lexical anaysis part. 

Finally, when reaching the start symbol, we make the tree root point to the tree generated:

```c
gt->root = tmp
```

### 1.5 Additional code

We place our test code here.

## 2. Writing the Lexer And Parser

### 2.1 Communication between bison and flex

#### 2.1.1 token declaration

As is noted before, the parser defines tokens that can be used by the lexer. In fact, when using `bison` to compile the _parser.y_ file, it will generate a header file that defines tokens like this:

```c
#ifndef YYTOKENTYPE
# define YYTOKENTYPE
  enum yytokentype
  {
    tok_error = 258,
    //.....
  }
#endif
```

And our lexer includes the header file in the first section, return the nodes that are predefined in the header:

```c
%{
  #include "parser.tab.h"
}
%%

. {
  return tok_error;
}
```

#### 2.1.2 yylval and node type

In the lexer file, we use `yylval` to save the tree node pointer and return the token. As explained [here](https://www.epaperpress.com/lexandyacc/pry1.html), the function yylex has a return type of int that returns a token. Values associated with the token are returned by lex in variable yylval.
 For example when lex returns an INTEGER token yacc shifts this token to the parse stack. At the same time the corresponding yylval is shifted to the value
In this case, yylval.pnode is associated with a node that contains a name. The type of yylval is by default int but changed in the bison %union
section. When lex returns the token, yyval is also pushed to the stack, and we can use $1, $2... to use yylval.

In our case, we define our tokens of type `struct tree_node* pnode`. And for each all tokens, we use the following convention:

```c
%type <pnode> tok_error tok_add tok_sub ....
```

And in the Lexer part, we use the following code the pass the pnode value:

```c
"+" {
  #ifndef LEX_TEST
    yylval.pnode = newSynTreeNode("+"); // tree node
  #endif

  return tok_add;
}
```

### 2.2 Syntax tree

There is not much to say about the syntax tree: it is just a normal tree structure.

### 2.3 Test section

As you may notice in the lexer file, I define a macro `LEX_TEST` for indicating whether the code is compiled for testing or not. And the macro is passed in the compile time using the `-D` flag of the compiler.

Using `cmake` for this can be quite easy, though.

The testing main files reside in the `test_src` folder, and the building process automatically generates the test binaries for them.

## 3. Source Code

You can find the source code of this part [here](https://github.com/hellobbn/cminusc/tree/dafb2c514d9494c74384f9b80642835ed10f3d93)
