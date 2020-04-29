---
title: Write a simple compiler [Part 2]
date: 2020-04-27 2
category:
  - Programming
tags:
  - Compiler
  - LLVM
---

Building a C- compiler using bison, flex and LLVM, Part 2: Lexer

<!-- more -->

In this part, we are going to use _flex_ to build the lexer of the compiler.

## 1. Flex

### 1.1 Intro to flex

"flex" is also called the fast lexical analyser generator. As quoted in its description:

> `flex` is a tool for generating _scanners_: programs which recognized lexical patterns in text.

And `flex` accepts files with extension _.l_, which consists of the following sections:

```
definitions
%%
rules
%%
user code
```

where `%%` is treated as seperator.

Here we present an example illustrating the usage of `flex`, for more detailed docs, you should refer to the flex docs

### 1.2 Flex example (from flex doc)

Here is a simple example that uses flex for a scanner of a toy Pascal-like language:

```flex
%{
  /* C like includes and definitions should start here */
  /* This is needed for atof() below */
  #include <math.h>
}

/* Rules start here : in regex */
{DIGIT}+    {
            printf( "An integer: %s (%d)\n", yytext,
                    atoi( yytext ) );
            }

{DIGIT}+"."{DIGIT}*        {
            printf( "A float: %s (%g)\n", yytext,
                    atof( yytext ) );
            }

if|then|begin|end|procedure|function        {
            printf( "A keyword: %s\n", yytext );
            }

{ID}        printf( "An identifier: %s\n", yytext );

"+"|"-"|"*"|"/"   printf( "An operator: %s\n", yytext );

"{"[^}\n]*"}"     /* eat up one-line comments */

[ \t\n]+          /* eat up whitespace */

.           printf( "Unrecognized character: %s\n", yytext );

%%

/* User code: functions, etc */
int main(int argc, char** argv) {
  /* yyin: the stream that flex uses */
  if (argc > 1)
    yyin = fopen(argv[1], "r");
  else
    yyin = stdin;

  /* start lexer here */
  yylex();
}
```

When a pattern matches the regex in the _rules_ part, the lexer will exec the code in the braces.

### 1.3 Use flex

To compile a flex file, use the following command:

```bash
-> flex wc.l  # wc.l is the file name
              # this command will generate `lex.yy.c`
-> gcc lex.yy.c # compile the c file generated by flex
-> ./a.out  # run
```

__Note:__ we can run the program because in the user code section, we define a `main` function!

## 2. C- Lexical conventions

1. keywords:
   `else`, `if`, `int`, `return`, `void`, `while`
2. preserved symbols:
  \+ - * / < <= >= > == != ; , ( ) [ ] /* */
3. Identifier, number:
   ID = letter letter*
   NUM = digit digit*
   letter = a | ... | z | A | .... | Z
   digit = 0 | ... | 9
4. Comment: /\*.....\*/

## 3. Write the code!

You can find the source code [here](https://github.com/hellobbn/cminusc/commit/e0d7c1da6e2680ed5e582348959069881c0c017b)

## 4. Extra notes (From lab report)

### 4.1 Regex for tokens

Most regex for the tokens are very simple, such as `">="` for `GTE`, but some are a bit difficult, such as the regex expression for `COMMENT` and for `ARRAY`.

The "definition" of `ARRAY` is a bit unclear, for example, is the `ARRAY` token for `a[10]` `[10]` while `a` is treated as `IDENTIFIER` or the entire `a[10]`?
In `lexical_analyzer`, I use the latter pattern.

The regex expression for the token `COMMENT` is difficult due to the `greedy match` of `flex`'s regex.
According to [here](https://stackoverflow.com/questions/4166194/how-do-i-write-a-non-greedy-match-in-lex-flex), using `[^*/]` can prohibit having a `*/` in
the pattern, and accept any other characters.

### 4.2 Find files

The program need to fetch all files ended with `.cminus` in `testcase` folder, and I use the `dirent.h` header file for this. In `getAllTestCase()`, the function
finds all files ended with `.cminus` and put their names in the array `filenames`.
Then it cuts the `cminus` part and appends the `.tokens` for the output filename. The program saves the tokens in the `./tokens` folder.

### 4.3 Scanner position

To find where the tokens are located, I used a `flex` option `yylineno` for the line number and `strlen` to calculate the `start` and `end` position in the line.
The source code is as follows:

```c
/* Init Pointer */
pos_start = 1;
pos_end = 1;

/* For each read, move scanner pointer */
while(token = yylex()) {
  lines += yylineno - prev_line;

  for (int i = 0; i < yyleng; ++ i) {
    if (yytext[i] != '\n') {
      pos_end ++;
    } else {
      pos_start += 1;
      pos_end = 1;
    }
  }
}
```