# Compiler Design Lab – LEX & YACC Practicals 1–11

## 📑 Contents

- [Practical 1: Introduction to LEX tool, metadata and patterns](#p1)
- [Practical 2: Count comments, keywords, identifiers, words, lines and spaces](#p2)
- [Practical 3: Count the number of words starting with “A”](#p3)
- [Practical 4: Introduction to YACC tool and format to write YACC code](#p4)
- [Practical 5: Conversion of lowercase to uppercase and vice versa](#p5)
- [Practical 6: Conversion of decimal to hexadecimal number in a file](#p6)
- [Practical 7: Test lines ending with “COM”](#p7)
- [Practical 8: Postfix expression evaluation](#p8)
- [Practical 9: Desk calculator with error recovery](#p9)
- [Practical 10: Parser for “FOR” loop statements](#p10)
- [Practical 11: Intermediate code generation for an arithmetic expression](#p11)

## One-time setup (Ubuntu / WSL)

```bash
sudo apt update && sudo apt install flex bison gcc gedit -y
```

## Make a folder (do once)

```bash
mkdir cd_lab && cd cd_lab
```

## Note

```bash
LEX only : flex file.l  ->  gcc lex.yy.c -o file
LEX+YACC: bison -d file.y  ->  flex file.l  ->  gcc lex.yy.c file.tab.c -o file
```

---

<a id="p1"></a>

# 📘 Practical 1: Introduction to LEX tool, metadata and patterns

**Step 1: Create file**

```bash
gedit p1.l &
```

**Paste this in p1.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
%}

%%
one|ONE        printf("1");
two|TWO        printf("2");
three|THREE    printf("3");
four|FOUR      printf("4");
five|FIVE      printf("5");
six|SIX        printf("6");
seven|SEVEN    printf("7");
eight|EIGHT    printf("8");
nine|NINE      printf("9");
ten|TEN        printf("10");
prn|PRN        printf("24070521162");
%%

int yywrap() { return 1; }

int main()
{
    yylex();
    return 0;
}
```

**Step 2: Generate C code**

```bash
flex p1.l
```

**Step 3: Compile**

```bash
gcc lex.yy.c -o p1
```

**Step 4: Run**

```bash
echo "one two three ten prn" | ./p1
```

**Expected output**

```text
1 2 3 10 24070521162
```

---

<a id="p2"></a>

# 📘 Practical 2: Count comments, keywords, identifiers, words, lines and spaces

**Step 1: Create file**

```bash
gedit p2.l &
```

**Paste this in p2.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
int comments = 0, keyword = 0, identifier = 0;
int words = 0, lines = 0, spaces = 0;
%}

%%
"/*"([^*]|\*+[^*/])*\*+"/"  { int i; comments++;
                              for (i = 0; i < yyleng; i++)
                                  if (yytext[i] == '\n') lines++; }
"//".*                      { comments++; }
[ \t]+                      { spaces += yyleng; }
\n                          { lines++; }
"int"|"float"|"char"|"double"|"if"|"else"|"while"|"for"|"return"  { keyword++; words++; }
[a-zA-Z_][a-zA-Z0-9_]*      { identifier++; words++; }
[0-9]+                      { words++; }
.                           { /* ignore other characters */ }
%%

int yywrap() { return 1; }

int main()
{
    yylex();
    printf("comments   : %d\n", comments);
    printf("keywords   : %d\n", keyword);
    printf("identifier : %d\n", identifier);
    printf("words      : %d\n", words);
    printf("lines      : %d\n", lines);
    printf("spaces     : %d\n", spaces);
    return 0;
}
```

**Step 2: Create file**

```bash
gedit input.c &
```

**Paste this in input.c, save (Ctrl+S) and close**

```text
/* sample program */
#include <stdio.h>
int main() {
    int a = 10; // counter
    float total = 5.5;
    printf("Hello");
    return 0;
}
```

**Step 3: Generate C code**

```bash
flex p2.l
```

**Step 4: Compile**

```bash
gcc lex.yy.c -o p2
```

**Step 5: Run**

```bash
./p2 < input.c
```

**Expected output**

```text
comments   : 2
keywords   : 4
identifier : 8
words      : 16
lines      : 8
spaces     : 27
```

---

<a id="p3"></a>

# 📘 Practical 3: Count the number of words starting with “A”

**Step 1: Create file**

```bash
gedit p3.l &
```

**Paste this in p3.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
int count = 0;
%}

%%
[Aa][a-zA-Z0-9]*   { count++; }
[a-zA-Z0-9]+       ;
[ \t\n]+           ;
.                  ;
%%

int yywrap() { return 1; }

int main()
{
    printf("Enter a sentence (Ctrl+D to stop):\n");
    yylex();
    printf("\nNumber of words starting with A or a = %d\n", count);
    return 0;
}
```

**Step 2: Create file**

```bash
gedit words.txt &
```

**Paste this in words.txt, save (Ctrl+S) and close**

```text
Apple and banana
Ant eats an Avocado
zebra
```

**Step 3: Generate C code**

```bash
flex p3.l
```

**Step 4: Compile**

```bash
gcc lex.yy.c -o p3
```

**Step 5: Run**

```bash
./p3 < words.txt
```

**Expected output**

```text
Enter a sentence (Ctrl+D to stop):

Number of words starting with A or a = 5
```

---

<a id="p4"></a>

# 📘 Practical 4: Introduction to YACC tool and format to write YACC code

**Step 1: Create file**

```bash
gedit p4.l &
```

**Paste this in p4.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include "p4.tab.h"
%}

%%
[0-9]+     { return NUM; }
[ \t]      ;
\n         { return 0; }
.          { return yytext[0]; }
%%

int yywrap() { return 1; }
```

**Step 2: Create file**

```bash
gedit p4.y &
```

**Paste this in p4.y, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
#include <stdlib.h>
int yylex(void);
int yyerror(char *s);
%}

%token NUM

%%
E : E '+' T
  | T
  ;

T : T '*' F
  | F
  ;

F : '(' E ')'
  | NUM
  ;
%%

int yyerror(char *s)
{
    printf("Invalid Expression\n");
    return 0;
}

int main()
{
    printf("Enter Expression: ");
    if (yyparse() == 0)
        printf("Valid Expression\n");
    return 0;
}
```

**Step 3: Run YACC**

```bash
bison -d p4.y
```

**Step 4: Run LEX**

```bash
flex p4.l
```

**Step 5: Compile**

```bash
gcc lex.yy.c p4.tab.c -o p4
```

**Step 6: Run (valid expression)**

```bash
echo "2+3*(4+5)" | ./p4
```

**Expected output**

```text
Enter Expression: Valid Expression
```

**Step 7: Run (invalid expression)**

```bash
echo "2+*3" | ./p4
```

**Expected output**

```text
Enter Expression: Invalid Expression
```

---

<a id="p5"></a>

# 📘 Practical 5: Conversion of lowercase to uppercase and vice versa

**Step 1: Create file**

```bash
gedit p5.l &
```

**Paste this in p5.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>

char convert_case(char c)
{
    if (c >= 'a' && c <= 'z')
        return c - 'a' + 'A';
    else if (c >= 'A' && c <= 'Z')
        return c - 'A' + 'a';
    return c;
}
%}

%%
[a-zA-Z]    { putchar(convert_case(yytext[0])); }
.|\n        { putchar(yytext[0]); }
%%

int yywrap() { return 1; }

int main()
{
    yylex();
    return 0;
}
```

**Step 2: Generate C code**

```bash
flex p5.l
```

**Step 3: Compile**

```bash
gcc lex.yy.c -o p5
```

**Step 4: Run**

```bash
echo "Hello World 123" | ./p5
```

**Expected output**

```text
hELLO wORLD 123
```

---

<a id="p6"></a>

# 📘 Practical 6: Conversion of decimal to hexadecimal number in a file

**Step 1: Create file**

```bash
gedit p6.l &
```

**Paste this in p6.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>

void decimal_to_hex(int num)
{
    char hex[100];
    int i = 0, j, remainder;

    if (num == 0) {
        printf("0x0");
        return;
    }
    while (num != 0) {
        remainder = num % 16;
        if (remainder < 10)
            hex[i++] = remainder + '0';
        else
            hex[i++] = remainder - 10 + 'A';
        num = num / 16;
    }
    printf("0x");
    for (j = i - 1; j >= 0; j--)
        printf("%c", hex[j]);
}

int string_to_int(char *str)
{
    int i, result = 0;
    for (i = 0; str[i] != '\0'; i++)
        result = result * 10 + (str[i] - '0');
    return result;
}
%}

%%
[0-9]+    { decimal_to_hex(string_to_int(yytext)); }
.|\n      { ECHO; }
%%

int yywrap() { return 1; }

int main()
{
    yylex();
    return 0;
}
```

**Step 2: Create file**

```bash
gedit numbers.txt &
```

**Paste this in numbers.txt, save (Ctrl+S) and close**

```text
255 and 16 and 0
4095
```

**Step 3: Generate C code**

```bash
flex p6.l
```

**Step 4: Compile**

```bash
gcc lex.yy.c -o p6
```

**Step 5: Run**

```bash
./p6 < numbers.txt
```

**Expected output**

```text
0xFF and 0x10 and 0x0
0xFFF
```

---

<a id="p7"></a>

# 📘 Practical 7: Test lines ending with “COM”

**Step 1: Create file**

```bash
gedit p7.l &
```

**Paste this in p7.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
int line_number = 0;
%}

%%
^.*com$    { line_number++;
             printf("Line %d ends with 'com': %s\n", line_number, yytext); }
^.+$       { line_number++; }
\n         { /* ignore new lines */ }
%%

int yywrap() { return 1; }

int main()
{
    yylex();
    return 0;
}
```

**Step 2: Create file**

```bash
gedit lines.txt &
```

**Paste this in lines.txt, save (Ctrl+S), close (press Enter after the last line)**

```text
google.com
hello world
amazon.com
welcome
```

**Step 3: Generate C code**

```bash
flex p7.l
```

**Step 4: Compile**

```bash
gcc lex.yy.c -o p7
```

**Step 5: Run**

```bash
./p7 < lines.txt
```

**Expected output**

```text
Line 1 ends with 'com': google.com
Line 3 ends with 'com': amazon.com
```

---

<a id="p8"></a>

# 📘 Practical 8: Postfix expression evaluation

**Step 1: Create file**

```bash
gedit p8.l &
```

**Paste this in p8.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
#include <stdlib.h>
#include "p8.tab.h"
%}

%%
[0-9]+     { yylval = atoi(yytext); return NUMBER; }
[ \t]+     ;
\n         { return '\n'; }
.          { return yytext[0]; }
%%

int yywrap(void) { return 1; }
```

**Step 2: Create file**

```bash
gedit p8.y &
```

**Paste this in p8.y, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
#include <stdlib.h>
void yyerror(const char *s);
int yylex(void);
%}

%token NUMBER

%%
program:
      /* empty */
    | program line
    ;

line:
      '\n'
    | expr '\n'        { printf("Result: %d\n", $1); }
    ;

expr:
      NUMBER           { $$ = $1; }
    | expr expr '+'    { $$ = $1 + $2; }
    | expr expr '-'    { $$ = $1 - $2; }
    | expr expr '*'    { $$ = $1 * $2; }
    | expr expr '/'    { $$ = $1 / $2; }
    ;
%%

void yyerror(const char *s)
{
    fprintf(stderr, "Error: %s\n", s);
}

int main(void)
{
    printf("Enter postfix expressions (Ctrl+D to stop):\n");
    yyparse();
    return 0;
}
```

**Step 3: Run YACC**

```bash
bison -d p8.y
```

**Step 4: Run LEX**

```bash
flex p8.l
```

**Step 5: Compile**

```bash
gcc lex.yy.c p8.tab.c -o p8
```

**Step 6: Run**

```bash
printf "5 3 +\n5 3 + 2 *\n10 2 8 * + 3 -\n" | ./p8
```

**Expected output**

```text
Enter postfix expressions (Ctrl+D to stop):
Result: 8
Result: 16
Result: 23
```

---

<a id="p9"></a>

# 📘 Practical 9: Desk calculator with error recovery

**Step 1: Create file**

```bash
gedit p9.l &
```

**Paste this in p9.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#define YYSTYPE double
#include <stdlib.h>
#include "p9.tab.h"
%}

%%
[0-9]+(\.[0-9]+)?    { yylval = atof(yytext); return NUMBER; }
"++"                 { return INC; }
"--"                 { return DEC; }
"+"                  { return '+'; }
"-"                  { return '-'; }
"*"                  { return '*'; }
"/"                  { return '/'; }
"%"                  { return '%'; }
"^"                  { return '^'; }
"("                  { return '('; }
")"                  { return ')'; }
\n                   { return '\n'; }
[ \t]+               ;
.                    { return INVALID; }
%%

int yywrap() { return 1; }
```

**Step 2: Create file**

```bash
gedit p9.y &
```

**Paste this in p9.y, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
#include <math.h>
#define YYSTYPE double
int yylex(void);
void yyerror(const char *s);
%}

%token NUMBER
%token INC DEC
%token INVALID
%left '+' '-'
%left '*' '/' '%'
%right '^'

%%
input:
      /* empty */
    | input line
    ;

line:
      expr '\n'     { printf("Result = %g\n", $1); }
    | '\n'          { }
    | error '\n'    { printf("Error: Invalid expression. Please try again.\n");
                      yyerrok; }
    ;

expr:
      NUMBER              { $$ = $1; }
    | expr '+' expr       { $$ = $1 + $3; }
    | expr '-' expr       { $$ = $1 - $3; }
    | expr '*' expr       { $$ = $1 * $3; }
    | expr '/' expr       { if ($3 == 0) { yyerror("Division by zero"); YYERROR; }
                            $$ = $1 / $3; }
    | expr '%' expr       { if ($3 == 0) { yyerror("Modulus by zero"); YYERROR; }
                            $$ = fmod($1, $3); }
    | expr '^' expr       { $$ = pow($1, $3); }
    | INC NUMBER          { $$ = $2 + 1; }
    | DEC NUMBER          { $$ = $2 - 1; }
    | '(' expr ')'        { $$ = $2; }
    ;
%%

void yyerror(const char *s)
{
    printf("Error: %s\n", s);
}

int main()
{
    printf(" SDT DESK CALCULATOR \n");
    printf("Operators: + - * / %% ^ ++ --\n");
    printf("Press Ctrl+D to exit.\n\n");
    yyparse();
    return 0;
}
```

**Step 3: Run YACC**

```bash
bison -d p9.y
```

**Step 4: Run LEX**

```bash
flex p9.l
```

**Step 5: Compile (-lm is needed for pow and fmod)**

```bash
gcc lex.yy.c p9.tab.c -o p9 -lm
```

**Step 6: Run**

```bash
printf "3 + 4 * 2\n(10 - 4) / 3\n2 ^ 3 ^ 2\n10 %% 3\n++5\n3 + * 4\n5 / 0\n" | ./p9
```

**Expected output**

```text
 SDT DESK CALCULATOR 
Operators: + - * / % ^ ++ --
Press Ctrl+D to exit.

Result = 11
Result = 2
Result = 512
Result = 1
Result = 6
Error: syntax error
Error: Invalid expression. Please try again.
Error: Division by zero
Error: Invalid expression. Please try again.
```

---

<a id="p10"></a>

# 📘 Practical 10: Parser for “FOR” loop statements

**Step 1: Create file**

```bash
gedit p10.l &
```

**Paste this in p10.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
#include "p10.tab.h"
%}

%%
"for"               { return FOR; }
"("                 { return LPAREN; }
")"                 { return RPAREN; }
"{"                 { return LBRACE; }
"}"                 { return RBRACE; }
";"                 { return SEMICOLON; }
"++"                { return INC; }
"--"                { return DEC; }
"<="                { return LE; }
">="                { return GE; }
"=="                { return EQ; }
"!="                { return NE; }
"<"                 { return LT; }
">"                 { return GT; }
"="                 { return ASSIGN; }
[a-zA-Z_][a-zA-Z0-9_]*   { return IDENTIFIER; }
[0-9]+              { return NUMBER; }
[ \t\n]+            ;
.                   { printf("Invalid character: %s\n", yytext); }
%%

int yywrap(void) { return 1; }
```

**Step 2: Create file**

```bash
gedit p10.y &
```

**Paste this in p10.y, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
#include <stdlib.h>
void yyerror(const char *s);
int yylex(void);
%}

%token FOR LPAREN RPAREN LBRACE RBRACE SEMICOLON ASSIGN
%token IDENTIFIER NUMBER INC DEC LT GT LE GE EQ NE

%%
program:
    for_loop
    ;

for_loop:
    FOR LPAREN initialization SEMICOLON condition SEMICOLON update RPAREN
    LBRACE body RBRACE
    {
        printf("Valid FOR loop structure\n");
    }
    ;

initialization:
    IDENTIFIER ASSIGN NUMBER
    ;

condition:
    IDENTIFIER relop operand
    ;

relop:
    LT | GT | LE | GE | EQ | NE
    ;

operand:
    IDENTIFIER
    | NUMBER
    ;

update:
    IDENTIFIER INC
    | IDENTIFIER DEC
    ;

body:
    /* empty */
    | body IDENTIFIER ASSIGN operand SEMICOLON
    ;
%%

void yyerror(const char *s)
{
    fprintf(stderr, "Error: %s\n", s);
}

int main(void)
{
    printf("Enter a FOR loop:\n");
    yyparse();
    return 0;
}
```

**Step 3: Run YACC**

```bash
bison -d p10.y
```

**Step 4: Run LEX**

```bash
flex p10.l
```

**Step 5: Compile**

```bash
gcc lex.yy.c p10.tab.c -o p10
```

**Step 6: Run (valid FOR loop)**

```bash
echo "for(i=0;i<10;i++){ x=5; }" | ./p10
```

**Expected output**

```text
Enter a FOR loop:
Valid FOR loop structure
```

**Step 7: Run (invalid FOR loop)**

```bash
echo "for(i=0;i<10){ }" | ./p10
```

**Expected output**

```text
Enter a FOR loop:
Error: syntax error
```

---

<a id="p11"></a>

# 📘 Practical 11: Intermediate code generation for an arithmetic expression

**Step 1: Create file**

```bash
gedit p11.l &
```

**Paste this in p11.l, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
#include <stdlib.h>
#include "p11.tab.h"
%}

%%
[0-9]+      { yylval.num = atoi(yytext); return NUMBER; }
[-+*/()]    { return yytext[0]; }
\n          { return '\n'; }
[ \t]+      ;
.           { printf("Invalid character: %s\n", yytext); }
%%

int yywrap(void) { return 1; }
```

**Step 2: Create file**

```bash
gedit p11.y &
```

**Paste this in p11.y, save (Ctrl+S) and close**

```text
%{
/* Name: Aaryan Kawale ; PRN: 24070521162 */
#include <stdio.h>
#include <stdlib.h>
void yyerror(const char *s);
int yylex(void);

int temp = 0;

char *newTemp(void)
{
    char *t = malloc(16);
    sprintf(t, "t%d", temp++);
    return t;
}
%}

%union {
    int num;
    char *name;
}

%token <num> NUMBER
%type <name> expr

%left '+' '-'
%left '*' '/'

%%
program:
    expr '\n'
    {
        printf("Final result in %s\n", $1);
    }
    ;

expr:
      NUMBER
        { $$ = newTemp(); printf("%s = %d\n", $$, $1); }
    | expr '+' expr
        { $$ = newTemp(); printf("%s = %s + %s\n", $$, $1, $3); }
    | expr '-' expr
        { $$ = newTemp(); printf("%s = %s - %s\n", $$, $1, $3); }
    | expr '*' expr
        { $$ = newTemp(); printf("%s = %s * %s\n", $$, $1, $3); }
    | expr '/' expr
        { $$ = newTemp(); printf("%s = %s / %s\n", $$, $1, $3); }
    | '(' expr ')'
        { $$ = $2; }
    ;
%%

void yyerror(const char *s)
{
    fprintf(stderr, "Error: %s\n", s);
}

int main(void)
{
    printf("Enter an arithmetic expression:\n");
    printf("Three Address Code:\n");
    yyparse();
    return 0;
}
```

**Step 3: Run YACC**

```bash
bison -d p11.y
```

**Step 4: Run LEX**

```bash
flex p11.l
```

**Step 5: Compile**

```bash
gcc lex.yy.c p11.tab.c -o p11
```

**Step 6: Run**

```bash
echo "(2+3)*4" | ./p11
```

**Expected output**

```text
Enter an arithmetic expression:
Three Address Code:
t0 = 2
t1 = 3
t2 = t0 + t1
t3 = 4
t4 = t2 * t3
Final result in t4
```
