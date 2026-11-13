# Compiler Design Lab

LEX & YACC Practicals 1--10 -- Step-by-Step Execution Guide

## Before you start: Setup

### Option A -- Linux / Ubuntu / WSL (recommended)

1.  Install the tools once: sudo apt update && sudo apt install flex
    bison gcc -y
2.  Check: flex --version, bison --version, gcc --version

### Option B -- Windows

1.  Install WinFlexBison (gives win_flex and win_bison) and MinGW-w64
    GCC, then add both folders to PATH.
2.  Wherever this guide says flex use win_flex, and for bison use
    win_bison. Everything else is identical.
3.  To end keyboard input on Windows press Ctrl+Z then Enter (on Linux:
    Ctrl+D).

### Golden order of commands

``` bash
LEX only program: flex file.l  &&  gcc lex.yy.c -o file  &&  ./file
```

``` bash
LEX + YACC program: bison -d file.y  &&  flex file.l  &&  gcc lex.yy.c file.tab.c -o file  &&  ./file
```

> Tip: YACC must run before LEX because the lexer includes the token
> header (file.tab.h) that Bison creates. All programs below were
> compiled and tested before putting them in this file.

### Practical list at a glance

  -----------------------------------------------------------------------
  S.No                    Experiment              Tools
  ----------------------- ----------------------- -----------------------
  1                       Introduction to LEX     LEX
                          tool, metadata and      
                          patterns                

  2                       Count comments,         LEX
                          keywords, identifiers,  
                          words, lines and spaces 

  3                       Count words starting    LEX
                          with "A"                

  4                       Introduction to YACC    LEX + YACC
                          tool and format to      
                          write YACC code         

  5                       Lowercase to uppercase  LEX
                          and vice versa          

  6                       Decimal to hexadecimal  LEX
                          number in a file        

  7                       Test lines ending with  LEX
                          "COM"                   

  8                       Postfix expression      LEX + YACC
                          evaluation              

  9                       Desk calculator with    LEX + YACC
                          error recovery          

  10                      Parser for "FOR" loop   LEX + YACC
                          statements              
  -----------------------------------------------------------------------

## Practical 1: Introduction to LEX tool, metadata and patterns

**Aim:** To understand the LEX tool, the structure of a LEX program, and
to recognise simple patterns. **Concept:** A LEX file has 3 sections
separated by %%: (1) Definitions -- C headers, variables inside %{ %};
(2) Rules -- pattern + action; (3) User code -- main() and yywrap().
yytext holds the matched text, yylex() starts scanning, yywrap() tells
LEX there is no more input.

### Execution steps

1.  Open the terminal and create a folder: mkdir cd_lab && cd cd_lab
2.  Create the LEX file: nano p1.l (or use any editor), paste the
    program given below, then save.
3.  Generate the C code: flex p1.l → creates lex.yy.c
4.  Compile: gcc lex.yy.c -o p1
5.  Run: ./p1 (type text such as x = 5 + y then press Ctrl+D)

**p1.l**

``` c
%{
#include <stdio.h>
%}
%%
[0-9]+                    { printf("NUMBER     : %s\n", yytext); }
[a-zA-Z_][a-zA-Z0-9_]*    { printf("IDENTIFIER : %s\n", yytext); }
[+\-*/=]                  { printf("OPERATOR   : %s\n", yytext); }
[ \t\n]                   ;
.                         { printf("OTHER      : %s\n", yytext); }
%%
int yywrap() { return 1; }
int main() {
    printf("Enter text (Ctrl+D to stop):\n");
    yylex();
    return 0;
}
```

    Enter text (Ctrl+D to stop):
    IDENTIFIER : x
    OPERATOR   : =
    NUMBER     : 5
    OPERATOR   : +
    IDENTIFIER : y

## Practical 2: Count comments, keywords, identifiers, words, lines and spaces

**Aim:** To count comments, keywords, identifiers, words, lines and
spaces in an input file. **Concept:** Keywords are listed before the
identifier rule: when both match the same length, LEX picks the rule
written first. The block comment pattern also counts the lines/spaces
inside the comment.

### Execution steps

1.  Open the terminal and create a folder: mkdir cd_lab && cd cd_lab
2.  Create the LEX file: nano p2.l (or use any editor), paste the
    program given below, then save.
3.  Create the input file: nano input.c and paste the sample input
    below.
4.  Generate the C code: flex p2.l → creates lex.yy.c
5.  Compile: gcc lex.yy.c -o p2
6.  Run: ./p2 input.c

**p2.l**

``` c
%{
#include <stdio.h>
int comments = 0, keywords = 0, identifiers = 0;
int words = 0, lines = 0, spaces = 0;
%}
%%
"//".*                       { comments++; }
"/*"([^*]|\*+[^*/])*\*+"/"   { comments++;
                               for (int i = 0; yytext[i]; i++) {
                                   if (yytext[i] == '\n') lines++;
                                   else if (yytext[i] == ' ') spaces++;
                               } }
int|float|char|double|long|short|void|if|else|for|while|do|switch|case|break|continue|return|main|printf|scanf|include|define   { keywords++; words++; }
[a-zA-Z_][a-zA-Z0-9_]*       { identifiers++; words++; }
[0-9]+(\.[0-9]+)?            { words++; }
\n                           { lines++; }
[ ]                          { spaces++; }
\t                           { spaces++; }
.                            ;
%%
int yywrap() { return 1; }
int main(int argc, char *argv[]) {
    if (argc < 2) { printf("Usage: %s inputfile\n", argv[0]); return 1; }
    yyin = fopen(argv[1], "r");
    if (!yyin) { perror("Cannot open file"); return 1; }
    yylex();
    printf("Comments    : %d\n", comments);
    printf("Keywords    : %d\n", keywords);
    printf("Identifiers : %d\n", identifiers);
    printf("Words       : %d\n", words);
    printf("Lines       : %d\n", lines);
    printf("Spaces      : %d\n", spaces);
    fclose(yyin);
    return 0;
}
```

**input.c (sample input file)**

``` text
/* sample program */
#include <stdio.h>
int main() {
    int a = 10; // counter
    float total = 5.5;
    printf("Hello");
    return 0;
}
```

    Comments    : 2
    Keywords    : 7
    Identifiers : 5
    Words       : 15
    Lines       : 8
    Spaces      : 30

> Note: the counts depend on your input file and on the keyword list in
> the program; add more keywords to the list if your file uses them.

## Practical 3: Count the number of words starting with "A"

**Aim:** To count the words that start with the letter A in a file.
**Concept:** Rule 1 matches any word beginning with A/a. Rule 2 swallows
all other words so they are not counted. For capital A only, change rule
1 to A\[a-zA-Z0-9\]\*

### Execution steps

1.  Open the terminal and create a folder: mkdir cd_lab && cd cd_lab
2.  Create the LEX file: nano p3.l (or use any editor), paste the
    program given below, then save.
3.  Create the input file: nano words.txt and type some lines of text
    (sample below).
4.  Generate the C code: flex p3.l → creates lex.yy.c
5.  Compile: gcc lex.yy.c -o p3
6.  Run: ./p3 words.txt

**p3.l**

``` c
%{
#include <stdio.h>
int count = 0;
%}
%%
[Aa][a-zA-Z0-9]*      { count++; printf("Found: %s\n", yytext); }
[a-zA-Z0-9]+          ;
.|\n                  ;
%%
int yywrap() { return 1; }
int main(int argc, char *argv[]) {
    if (argc < 2) { printf("Usage: %s inputfile\n", argv[0]); return 1; }
    yyin = fopen(argv[1], "r");
    if (!yyin) { perror("Cannot open file"); return 1; }
    yylex();
    printf("Total words starting with 'A' = %d\n", count);
    fclose(yyin);
    return 0;
}
```

**words.txt (sample input file)**

``` text
Apple and banana
Ant eats an Avocado
zebra
```

    Found: Apple
    Found: and
    Found: Ant
    Found: an
    Found: Avocado
    Total words starting with 'A' = 5

## Practical 4: Introduction to YACC tool and format to write YACC code

**Aim:** To understand the YACC tool, its program format, and how LEX
and YACC work together (simple calculator). **Concept:** A YACC file
also has 3 sections separated by %%: (1) Declarations -- %{ %}, %token,
%left (precedence); (2) Grammar rules -- production + { action } using
\$\$, \$1, \$2...; (3) User code -- yyerror() and main() which calls
yyparse(). LEX sends tokens to YACC through yylex() and the value via
yylval.

### Execution steps

1.  Open the terminal and go to your lab folder: cd cd_lab
2.  Create the LEX file calc.l and the YACC file calc.y and paste the
    two programs given below.
3.  Run YACC/Bison first (creates the parser and the token header):
    bison -d calc.y → creates calc.tab.c and calc.tab.h
4.  Run LEX: flex calc.l → creates lex.yy.c
5.  Compile both together: gcc lex.yy.c calc.tab.c -o calc
6.  Run: ./calc (type 2+3\*4 and press Enter)

**calc.l (LEX file)**

``` text
%{
#include <stdlib.h>
#include "calc.tab.h"
%}
%%
[0-9]+    { yylval = atoi(yytext); return NUMBER; }
[ \t]     ;
\n        { return '\n'; }
.         { return yytext[0]; }
%%
int yywrap() { return 1; }
```

**calc.y (YACC file)**

``` text
%{
#include <stdio.h>
int yylex(void);
void yyerror(const char *s);
%}
%token NUMBER
%left '+' '-'
%left '*' '/'
%%
input : /* empty */
      | input line
      ;
line  : '\n'
      | expr '\n'    { printf("Result = %d\n", $1); }
      ;
expr  : expr '+' expr   { $$ = $1 + $3; }
      | expr '-' expr   { $$ = $1 - $3; }
      | expr '*' expr   { $$ = $1 * $3; }
      | expr '/' expr   { $$ = $1 / $3; }
      | '(' expr ')'    { $$ = $2; }
      | NUMBER
      ;
%%
void yyerror(const char *s) { printf("Error: %s\n", s); }
int main() {
    printf("Enter expressions (Ctrl+D to stop):\n");
    yyparse();
    return 0;
}
```

    Enter expressions (Ctrl+D to stop):
    2+3*4
    Result = 14
    (2+3)*4
    Result = 20

### YACC format at a glance

%{ C declarations %} %token / %left / %right <- tokens and precedence
%% grammar rules with actions expr : expr '+' expr { \$\$ = \$1 + \$3; }
%% C code: yyerror(), main() -> yyparse()

## Practical 5: Conversion of lowercase to uppercase and vice versa

**Aim:** To convert lowercase letters to uppercase and uppercase letters
to lowercase in a file. **Concept:** \[a-z\] and \[A-Z\] rules swap the
case using toupper()/tolower(); every other character is printed
unchanged using ECHO.

### Execution steps

1.  Open the terminal and create a folder: mkdir cd_lab && cd cd_lab
2.  Create the LEX file: nano p5.l (or use any editor), paste the
    program given below, then save.
3.  Create the input file: nano case.txt (sample below).
4.  Generate the C code: flex p5.l → creates lex.yy.c
5.  Compile: gcc lex.yy.c -o p5
6.  Run: ./p5 case.txt

**p5.l**

``` c
%{
#include <stdio.h>
#include <ctype.h>
%}
%%
[a-z]    { putchar(toupper(yytext[0])); }
[A-Z]    { putchar(tolower(yytext[0])); }
.|\n     { ECHO; }
%%
int yywrap() { return 1; }
int main(int argc, char *argv[]) {
    if (argc < 2) { printf("Usage: %s inputfile\n", argv[0]); return 1; }
    yyin = fopen(argv[1], "r");
    if (!yyin) { perror("Cannot open file"); return 1; }
    yylex();
    fclose(yyin);
    return 0;
}
```

**case.txt (sample input file)**

``` text
Hello World 123
Compiler Design
```

    hELLO wORLD 123
    cOMPILER dESIGN

## Practical 6: Conversion of decimal to hexadecimal number in a file

**Aim:** To read a file and replace every decimal number in it by its
hexadecimal value. **Concept:** The rule \[0-9\]+ matches a decimal
number; strtol() converts the text to a number and printf("%lX") prints
it in hexadecimal. All other text is copied as it is.

### Execution steps

1.  Open the terminal and create a folder: mkdir cd_lab && cd cd_lab
2.  Create the LEX file: nano p6.l (or use any editor), paste the
    program given below, then save.
3.  Create the input file: nano dec.txt (sample below).
4.  Generate the C code: flex p6.l → creates lex.yy.c
5.  Compile: gcc lex.yy.c -o p6
6.  Run: ./p6 dec.txt

**p6.l**

``` c
%{
#include <stdio.h>
#include <stdlib.h>
%}
%%
[0-9]+    { printf("0x%lX", strtol(yytext, NULL, 10)); }
.|\n      { ECHO; }
%%
int yywrap() { return 1; }
int main(int argc, char *argv[]) {
    if (argc < 2) { printf("Usage: %s inputfile\n", argv[0]); return 1; }
    yyin = fopen(argv[1], "r");
    if (!yyin) { perror("Cannot open file"); return 1; }
    yylex();
    fclose(yyin);
    return 0;
}
```

**dec.txt (sample input file)**

``` text
Number 10
Value 255 and 16
Total 4096
```

    Number 0xA
    Value 0xFF and 0x10
    Total 0x1000

## Practical 7: Test lines ending with "COM"

**Aim:** To check which lines of a file end with the string COM.
**Concept:** In LEX, $ matches the end of a line, so .*COM$ matches a
whole line that ends in COM. The second rule discards all the other
lines. Make sure the last line of the file ends with Enter.

### Execution steps

1.  Open the terminal and create a folder: mkdir cd_lab && cd cd_lab
2.  Create the LEX file: nano p7.l (or use any editor), paste the
    program given below, then save.
3.  Create the input file: nano com.txt (sample below).
4.  Generate the C code: flex p7.l → creates lex.yy.c
5.  Compile: gcc lex.yy.c -o p7
6.  Run: ./p7 com.txt

**p7.l**

``` c
%{
#include <stdio.h>
int count = 0;
%}
%%
.*COM$     { count++; printf("Ends with COM : %s\n", yytext); }
.*         ;
\n         ;
%%
int yywrap() { return 1; }
int main(int argc, char *argv[]) {
    if (argc < 2) { printf("Usage: %s inputfile\n", argv[0]); return 1; }
    yyin = fopen(argv[1], "r");
    if (!yyin) { perror("Cannot open file"); return 1; }
    yylex();
    printf("Total lines ending with COM = %d\n", count);
    fclose(yyin);
    return 0;
}
```

**com.txt (sample input file)**

``` text
www.google.COM
hello world
abc.com
mail.yahoo.COM
```

    Ends with COM : www.google.COM
    Ends with COM : mail.yahoo.COM
    Total lines ending with COM = 2

> Note: matching is case-sensitive. To accept com / Com too, change the
> rule to .*\[cC\]\[oO\]\[mM\]$

## Practical 8: Postfix expression evaluation

**Aim:** To evaluate a postfix (reverse Polish) expression using LEX and
YACC. **Concept:** In postfix the operator comes after its operands, so
the grammar is expr : expr expr '+' . YACC's own parse stack does the
work of the evaluation stack. Separate the numbers with spaces.

### Execution steps

1.  Open the terminal and go to your lab folder: cd cd_lab
2.  Create the LEX file post.l and the YACC file post.y and paste the
    two programs given below.
3.  Run YACC/Bison first (creates the parser and the token header):
    bison -d post.y → creates post.tab.c and post.tab.h
4.  Run LEX: flex post.l → creates lex.yy.c
5.  Compile both together: gcc lex.yy.c post.tab.c -o post
6.  Run: ./post (type 2 3 4 \* + and press Enter)

**post.l (LEX file)**

``` text
%{
#include <stdlib.h>
#include "post.tab.h"
%}
%%
[0-9]+    { yylval = atoi(yytext); return NUMBER; }
[ \t]     ;
\n        { return '\n'; }
.         { return yytext[0]; }
%%
int yywrap() { return 1; }
```

**post.y (YACC file)**

``` text
%{
#include <stdio.h>
int yylex(void);
void yyerror(const char *s);
%}
%token NUMBER
%%
input : /* empty */
      | input line
      ;
line  : '\n'
      | expr '\n'    { printf("Result = %d\n", $1); }
      ;
expr  : expr expr '+'   { $$ = $1 + $2; }
      | expr expr '-'   { $$ = $1 - $2; }
      | expr expr '*'   { $$ = $1 * $2; }
      | expr expr '/'   { $$ = $1 / $2; }
      | NUMBER
      ;
%%
void yyerror(const char *s) { printf("Error: %s\n", s); }
int main() {
    printf("Enter postfix expressions (e.g. 2 3 4 * +):\n");
    yyparse();
    return 0;
}
```

    Enter postfix expressions (e.g. 2 3 4 * +):
    2 3 4 * +
    Result = 14
    5 6 2 + *
    Result = 40
    10 2 /
    Result = 5

## Practical 9: Desk calculator with error recovery

**Aim:** To build a calculator that supports + − \* / ( ), decimals and
unary minus, and that recovers from errors instead of stopping.
**Concept:** The rule line : error '\n' tells YACC to skip the
bad input up to the next newline and continue; yyerrok resets the error
state. Division by zero is also checked inside the action.

### Execution steps

1.  Open the terminal and go to your lab folder: cd cd_lab
2.  Create the LEX file desk.l and the YACC file desk.y and paste the
    two programs given below.
3.  Run YACC/Bison first (creates the parser and the token header):
    bison -d desk.y → creates desk.tab.c and desk.tab.h
4.  Run LEX: flex desk.l → creates lex.yy.c
5.  Compile both together: gcc lex.yy.c desk.tab.c -o desk
6.  Run: ./desk (try 2+3*4, 5/0, 3+*4, -5+2, 7/2)

**desk.l (LEX file)**

``` text
%{
#include <stdlib.h>
#include "desk.tab.h"
%}
%%
[0-9]+(\.[0-9]+)?   { yylval.dval = atof(yytext); return NUMBER; }
[ \t]               ;
\n                  { return '\n'; }
.                   { return yytext[0]; }
%%
int yywrap() { return 1; }
```

**desk.y (YACC file)**

``` text
%{
#include <stdio.h>
int yylex(void);
void yyerror(const char *s);
%}
%union { double dval; }
%token <dval> NUMBER
%type  <dval> expr
%left '+' '-'
%left '*' '/'
%right UMINUS
%%
input : /* empty */
      | input line
      ;
line  : '\n'
      | expr '\n'     { printf("Result = %g\n", $1); }
      | error '\n'    { printf("Recovered. Enter the next expression.\n"); yyerrok; }
      ;
expr  : expr '+' expr   { $$ = $1 + $3; }
      | expr '-' expr   { $$ = $1 - $3; }
      | expr '*' expr   { $$ = $1 * $3; }
      | expr '/' expr   { if ($3 == 0) { yyerror("division by zero"); $$ = 0; }
                          else $$ = $1 / $3; }
      | '-' expr %prec UMINUS { $$ = -$2; }
      | '(' expr ')'    { $$ = $2; }
      | NUMBER
      ;
%%
void yyerror(const char *s) { printf("Error: %s\n", s); }
int main() {
    printf("Desk Calculator (Ctrl+D to stop):\n");
    yyparse();
    return 0;
}
```

    Desk Calculator (Ctrl+D to stop):
    2+3*4
    Result = 14
    5/0
    Error: division by zero
    Result = 0
    3+*4
    Error: syntax error
    Recovered. Enter the next expression.
    -5+2
    Result = -3
    7/2
    Result = 3.5

## Practical 10: Parser for "FOR" loop statements

**Aim:** To write a parser that checks whether a given FOR loop
statement is syntactically valid. **Concept:** LEX returns tokens such
as FOR, ID, NUM, ++, \<=. YACC checks the structure for ( init ;
condition ; update ) body. Nested for loops and { } blocks are also
accepted.

### Execution steps

1.  Open the terminal and go to your lab folder: cd cd_lab
2.  Create the LEX file for.l and the YACC file for.y and paste the two
    programs given below.
3.  Run YACC/Bison first (creates the parser and the token header):
    bison -d for.y → creates for.tab.c and for.tab.h
4.  Run LEX: flex for.l → creates lex.yy.c
5.  Compile both together: gcc lex.yy.c for.tab.c -o for
6.  Run: ./for (type a FOR statement and press Enter)

**for.l (LEX file)**

``` text
%{
#include "for.tab.h"
%}
%%
"for"                      { return FOR; }
[a-zA-Z_][a-zA-Z0-9_]*     { return ID; }
[0-9]+                     { return NUM; }
"<="                       { return LE; }
">="                       { return GE; }
"=="                       { return EQ; }
"!="                       { return NE; }
"++"                       { return INC; }
"--"                       { return DEC; }
[ \t\n]                    ;
.                          { return yytext[0]; }
%%
int yywrap() { return 1; }
```

**for.y (YACC file)**

``` text
%{
#include <stdio.h>
int yylex(void);
void yyerror(const char *s);
%}
%token FOR ID NUM LE GE EQ NE INC DEC
%left '+' '-'
%left '*' '/'
%%
program  : program for_stmt
         | for_stmt
         ;
for_stmt : FOR '(' init ';' cond ';' update ')' body
           { printf("Valid FOR loop\n"); }
         ;
init     : ID '=' expr
         | /* empty */
         ;
cond     : expr relop expr
         | /* empty */
         ;
relop    : '<' | '>' | LE | GE | EQ | NE ;
update   : ID INC
         | ID DEC
         | ID '=' expr
         | /* empty */
         ;
body     : stmt
         | '{' stmtlist '}'
         | for_stmt
         ;
stmtlist : stmtlist stmt
         | stmtlist for_stmt
         | /* empty */
         ;
stmt     : ID '=' expr ';'
         | ';'
         ;
expr     : expr '+' expr
         | expr '-' expr
         | expr '*' expr
         | expr '/' expr
         | '(' expr ')'
         | ID
         | NUM
         ;
%%
void yyerror(const char *s) { printf("Invalid FOR statement (%s)\n", s); }
int main() {
    printf("Enter a FOR statement (Ctrl+D to stop):\n");
    yyparse();
    return 0;
}
```

    for(i=0;i<10;i++) x=x+i;
    Valid FOR loop
    for(i=0;i<10 i++) x=1;          <- missing semicolon
    Invalid FOR statement (syntax error)
    for(i=0;i<5;i++){ x=1; for(j=0;j<=3;j--) y=2; }     <- nested
    Valid FOR loop
    Valid FOR loop

> Note: after the first syntax error the parser stops. Re-run the
> program to test another statement.
