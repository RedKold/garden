## 内容 1-词法分析实践

> Lexical Analysis

首先需要阅读 C--文法

only `INT, FLOAT, ID` 3 unit need us to write regex expression

Then we use Flex (Fast Lexical Analyzer Generator) to implement it .


 Flex files have a .l extension and follow a 3-part structure:
  - Definitions: C declarations (e.g., includes, macros)
  - Rules: Token patterns (regular expressions) → actions (C code to run when matched)
  - User Code: Main function or helper functions to call the scanner

3 parts are separated with `%%`, as 
```
{definition}
%%
{rules}
%%
{user subroutines}
```

- In the definition part, we often use the pattern:
```
name definition
```
- definition is any regex expression

- In the *rule* part, we have the format as 
	- `pattern {action}`
	- where `{action}` is a C code


A simple example:****
```c
/* calc_scanner.l */
 %{
 #include <stdio.h>
 #include <stdlib.h>

 // Token types for a calculator
 typedef enum {
     TOKEN_INT,    // Integer (e.g., 123)
     TOKEN_PLUS,   // +
     TOKEN_MINUS,  // -
     TOKEN_MUL,    // *
     TOKEN_DIV,    // /
     TOKEN_LPAREN, // (
     TOKEN_RPAREN, // )
     TOKEN_EOF     // End of input
 } TokenType;

 // Structure to hold token info (type + value)
 typedef struct {
     TokenType type;
     int value; // For integers only
 } Token;

 Token nextToken; // Global token to return to parser
 %}

 /* Definitions */
 DIGIT [0-9]
 INTEGER {DIGIT}+

 %%

 /* Rules */
 {INTEGER} {
     nextToken.type = TOKEN_INT;
     nextToken.value = atoi(yytext); // Convert string to integer
     return TOKEN_INT;
 }

 "+" { nextToken.type = TOKEN_PLUS; return TOKEN_PLUS; }
 "-" { nextToken.type = TOKEN_MINUS; return TOKEN_MINUS; }
 "*" { nextToken.type = TOKEN_MUL; return TOKEN_MUL; }
 "/" { nextToken.type = TOKEN_DIV; return TOKEN_DIV; }
 "(" { nextToken.type = TOKEN_LPAREN; return TOKEN_LPAREN; }
 ")" { nextToken.type = TOKEN_RPAREN; return TOKEN_RPAREN; }

 [ \t\n] { /* Ignore whitespace */ }

 . { printf("ERROR: Unknown character '%c'\n", yytext[0]); exit(1); }

 %%

 /* User Code */
 int main(void) {
     Token t;
     printf("Calculator Scanner (Ctrl+D to exit):\n");
     while ((t.type = yylex()) != TOKEN_EOF) {
         switch (t.type) {
             case TOKEN_INT: printf("Token: INT, Value: %d\n", nextToken.value); break;
             case TOKEN_PLUS: printf("Token: PLUS\n"); break;
             case TOKEN_MINUS: printf("Token: MINUS\n"); break;
             case TOKEN_MUL: printf("Token: MUL\n"); break;
             case TOKEN_DIV: printf("Token: DIV\n"); break;
             case TOKEN_LPAREN: printf("Token: LPAREN\n"); break;
             case TOKEN_RPAREN: printf("Token: RPAREN\n"); break;
         }
     }
     return 0;
 }
```