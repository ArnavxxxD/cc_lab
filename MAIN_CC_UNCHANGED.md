Compiler Design Lab – LEX & YACC Practicals 1–10
One-time setup (Ubuntu / WSL)
sudo apt update && sudo apt install flex bison gcc gedit -y
Make a folder (do once)
mkdir cd_lab && cd cd_lab
Note
LEX only : flex file.l  ->  gcc lex.yy.c -o file
LEX+YACC: bison -d file.y  ->  flex file.l  ->  gcc lex.yy.c file.tab.c -o file

Practical 1: Introduction to LEX tool, metadata and patterns
Step 1: Create file
gedit p1.l &
Paste this in p1.l, save (Ctrl+S) and close
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
Step 2: Generate C code
flex p1.l
Step 3: Compile
gcc lex.yy.c -o p1
Step 4: Run
echo "x = 5 + y" | ./p1
Expected output
Enter text (Ctrl+D to stop):
IDENTIFIER : x
OPERATOR   : =
NUMBER     : 5
OPERATOR   : +
IDENTIFIER : y

Practical 2: Count comments, keywords, identifiers, words, lines and spaces
Step 1: Create file
gedit p2.l &
Paste this in p2.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit input.c &
Paste this in input.c, save (Ctrl+S) and close
/* sample program */
#include <stdio.h>
int main() {
    int a = 10; // counter
    float total = 5.5;
    printf("Hello");
    return 0;
}
Step 3: Generate C code
flex p2.l
Step 4: Compile
gcc lex.yy.c -o p2
Step 5: Run
./p2 input.c
Expected output
Comments    : 2
Keywords    : 7
Identifiers : 5
Words       : 15
Lines       : 8
Spaces      : 30

Practical 3: Count the number of words starting with “A”
Step 1: Create file
gedit p3.l &
Paste this in p3.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit words.txt &
Paste this in words.txt, save (Ctrl+S) and close
Apple and banana
Ant eats an Avocado
zebra
Step 3: Generate C code
flex p3.l
Step 4: Compile
gcc lex.yy.c -o p3
Step 5: Run
./p3 words.txt
Expected output
Found: Apple
Found: and
Found: Ant
Found: an
Found: Avocado
Total words starting with 'A' = 5

Practical 4: Introduction to YACC tool and format to write YACC code
Step 1: Create file
gedit calc.l &
Paste this in calc.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit calc.y &
Paste this in calc.y, save (Ctrl+S) and close
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
Step 3: Run YACC
bison -d calc.y
Step 4: Run LEX
flex calc.l
Step 5: Compile
gcc lex.yy.c calc.tab.c -o calc
Step 6: Run
printf '2+3*4\n(2+3)*4\n' | ./calc
Expected output
Enter expressions (Ctrl+D to stop):
Result = 14
Result = 20

Practical 5: Conversion of lowercase to uppercase and vice versa
Step 1: Create file
gedit p5.l &
Paste this in p5.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit case.txt &
Paste this in case.txt, save (Ctrl+S) and close
Hello World 123
Compiler Design
Step 3: Generate C code
flex p5.l
Step 4: Compile
gcc lex.yy.c -o p5
Step 5: Run
./p5 case.txt
Expected output
hELLO wORLD 123
cOMPILER dESIGN

Practical 6: Conversion of decimal to hexadecimal number in a file
Step 1: Create file
gedit p6.l &
Paste this in p6.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit dec.txt &
Paste this in dec.txt, save (Ctrl+S) and close
Number 10
Value 255 and 16
Total 4096
Step 3: Generate C code
flex p6.l
Step 4: Compile
gcc lex.yy.c -o p6
Step 5: Run
./p6 dec.txt
Expected output
Number 0xA
Value 0xFF and 0x10
Total 0x1000

Practical 7: Test lines ending with “COM”
Step 1: Create file
gedit p7.l &
Paste this in p7.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit com.txt &
Paste this in com.txt, save (Ctrl+S) and close
www.google.COM
hello world
abc.com
mail.yahoo.COM
Step 3: Generate C code
flex p7.l
Step 4: Compile
gcc lex.yy.c -o p7
Step 5: Run
./p7 com.txt
Expected output
Ends with COM : www.google.COM
Ends with COM : mail.yahoo.COM
Total lines ending with COM = 2

Practical 8: Postfix expression evaluation
Step 1: Create file
gedit post.l &
Paste this in post.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit post.y &
Paste this in post.y, save (Ctrl+S) and close
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
Step 3: Run YACC
bison -d post.y
Step 4: Run LEX
flex post.l
Step 5: Compile
gcc lex.yy.c post.tab.c -o post
Step 6: Run
printf '2 3 4 * +\n5 6 2 + *\n10 2 /\n' | ./post
Expected output
Enter postfix expressions (e.g. 2 3 4 * +):
Result = 14
Result = 40
Result = 5

Practical 9: Desk calculator with error recovery
Step 1: Create file
gedit desk.l &
Paste this in desk.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit desk.y &
Paste this in desk.y, save (Ctrl+S) and close
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
Step 3: Run YACC
bison -d desk.y
Step 4: Run LEX
flex desk.l
Step 5: Compile
gcc lex.yy.c desk.tab.c -o desk
Step 6: Run
printf '2+3*4\n5/0\n3+*4\n-5+2\n7/2\n' | ./desk
Expected output
Desk Calculator (Ctrl+D to stop):
Result = 14
Error: division by zero
Result = 0
Error: syntax error
Recovered. Enter the next expression.
Result = -3
Result = 3.5

Practical 10: Parser for “FOR” loop statements
Step 1: Create file
gedit for.l &
Paste this in for.l, save (Ctrl+S) and close
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
Step 2: Create file
gedit for.y &
Paste this in for.y, save (Ctrl+S) and close
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
Step 3: Run YACC
bison -d for.y
Step 4: Run LEX
flex for.l
Step 5: Compile
gcc lex.yy.c for.tab.c -o for
Step 6: Run (valid + invalid FOR)
printf 'for(i=0;i<10;i++) x=x+i;\nfor(i=0;i<10 i++) x=1;\n' | ./for
Step 7: Run (nested FOR)
printf 'for(i=0;i<5;i++){ x=1; for(j=0;j<=3;j--) y=2; }\n' | ./for
Expected output
Valid FOR loop
Invalid FOR statement (syntax error)
 
Valid FOR loop
Valid FOR loop
