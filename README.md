# COMP 232 Lab 7 - Lexing with Flex

Download [this project](./lab_7_flex.zip). It will serve as a starting point for this lab.

In this lab, we will be using a tool called **flex** to build a lexer which uses regular expressions to recognize token strings.

We will cover a minimal introduction to the use of flex in this lab. [This pdf](./resources/LexAndYaccTutorial.pdf) will provide some more detail, and you will likely want to refer to it throughout the lab.

Consider the simple grammar below.

```
<program> ::= <statement> | <statement> <program>
<statement> ::= <assignStmt> | <repeatStmt> | <printStmt>

<assignStmt> ::= <id> = <expr> ;
<ifStmt> ::= if ( <expr> ) <statement>
<whileStmt> ::= while ( <expr> ) <statement>
<repeatStmt> ::= repeat ( <expr> ) <statement>
<printStmt> ::= print <expr> ;

<expr> ::= <term> | <term> <addop> <expr>
<term> ::= <factor> | <factor> <multop> <term>
<factor> ::= <id> | <number> | <addop> <factor> | ( <expr> )

<number> ::= <int> | <float>
<int> ::= <digit> | <int> <digit>
<float> ::= <digit>. | <digit> <float> | <float> <digit>

<id> ::= <letter> | <id> <letter> | <id> <digit>

<addop> ::= + | -
<multop> ::= * | / | %

<digit> ::= 0 | 1 | 2 | ... | 9
<letter> ::= a | b | ... | z | A | B | ... | Z | _
```

Enumerate all of the token types in this grammar in the `TOKEN` enum in `flex.h`. In `flex.c`, fill out the `tokenTypeStrings` array to match the `TOKEN` enum, so elements of the `TOKEN` enum can be used to index the array and get the corresponding string.

Once you've enumerated all of the `TOKEN` types, you're ready to begin working on the `flex.l`. This file is essentially a configuration file, which flex will use to build a scanner.

It is divided into 3 sections, with a line containing `%%` between each section.

The first section is for **definitions**, the second for **rules**, and the third for **subroutines**. Your task is to complete the first and second sections (the third section will be left empty).

## Definitions

The first section can, in theory, be left empty for what we are doing today, but it should not be. In this section, you can create define shortcuts for regular expressions.

For instance, the lines

```
letter          [a-z]
digit           [0-9]
```

define `letter` to be a shorthand for the regular expression `[a-z]` and `digit` one for `[0-9]`.

Then, the lines

```
float           {digit}+\.{digit}*
id              {letter}+
```

define `float` to be one or more digits, followed by a period and then 0 or more digits, and then define `id` to be 1 or more letters.

Note that when the `digit` and `letter` definitions were used in the `float` and `id` definitions, they were encased in curly braces `{}`. Also note that the `id` definition does not match the one in the grammar, and will therefore need to be rewritten.

## Rules

The second section (after the first `%%`) is for tokenization rules. Each line consists of a regular expression (or a definition from the first section) followed by a block (C syntax) specifying what action is to be taken when a string is encountered which matches said regular expression.

Consider these lines provided in the rules section:

```
if          {return IF_TOKEN;}
{float}     {return FLOAT_TOKEN;}
{id}        {return IDENT_TOKEN;}
```

These lines mean, respectively:

* When the string "if" is encountered, return an `IF_TOKEN`.
* When a string matching the `float` definition is encountered, return a `FLOAT_TOKEN`.
* When a string matching the `id` definition is encountered, return an `IDENT_TOKEN`.

Note that some of the strings which need to be tokenized (such as parenthesis) have meta-meaning in regular expressions, and therefore will need to either be escaped or put in quotes in order to function as literal characters in a regular expression. For instance:

```
(			{return LPAREN_TOKEN;}
```

would not work, but:

```
\(			{return LPAREN_TOKEN;}
```

would work and so would:

```
"("			{return LPAREN_TOKEN;}
```

Order in this section matters. For example, any keyword (print, repeat, etc) will also match the regular expression for an identifier. As such, all of the keyword tokenizations must happen *before* identifier tokenization in the rules section, so keywords match their patterns and return the correct token type before they are ever compared to the `id` definition.

Three more rules have been provided at the bottom of the rules section (and they should be the last three in the rules section when you are done):

```
<<EOF>>     {return EOF_TOKEN;}
[ \n\r\t]   ; //skip whitespace
.           {printf("ERROR: invalid character: >>%s<<\n", yytext);}
```

These rules are to tokenize the end of file, skip whitespace, and catch any invalid characters respectively.

## Output

When you have completed the `TOKEN` enum in `flex.h`, the `tokenTypeStrings` array in `flex.c`, and rules, you are ready to test!

There is a provided sample input, `input.txt`, with the following contents:

```
while (0.4) abc_1_2 = agd + 1;
if (condition) print (hello) ;
```

For this sample input, the output should be:

```
{<while> "while"}
{<lparen> "("}
{<float> "0.4"}
{<rparen> ")"}
{<ident> "abc_1_2"}
{<assign> "="}
{<ident> "agd"}
{<addop> "+"}
{<int> "1"}
{<semicolon> ";"}
{<if> "if"}
{<lparen> "("}
{<ident> "condition"}
{<rparen> ")"}
{<print> "print"}
{<lparen> "("}
{<ident> "hello"}
{<rparen> ")"}
{<semicolon> ";"}
{<eof> ""}
Process finished with exit code 0
```

You do not need to edit the main to match this output; the only change you need to make the `flex.c` is to fill out the `tokenTypeStrings` array.

Note that each token is printed with both its type and the string value that was tokenized. In some cases this is redundant, and that's fine; we will work on more complex tokenization rules which process the string value or simply don't store it in a later lab.

As always, don't forget to rigorously document and test your lab! The input provided in `input.txt` is nowhere near an adequate test; it doesn't even contain every token type, let alone perform bug tests ensureing tokenization is done well indepenent of spacing, etc.


