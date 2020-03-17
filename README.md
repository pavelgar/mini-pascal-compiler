# Mini-Pascal compiler

Compiler for Mini-Pascal language. This is an assignment for University of Helsinki course on Code Generation.

## Context-free grammar

```
// Program

<program>           => "program" <id> ";" {<procedure> | <function>} <main-block> "."

<procedure>         => "procedure" <id> "(" parameters ")" ";" <block> ";"

<function>          => "function" <id> "(" parameters ")" ":" <type> ";" <block> ";"

<var-declaration>   => "var" <id> {"," <id>} ":" <type>

<parameters>        => ["var"] <id> : <type> {"," ["var"] <id> ":" <type>}
                    |  <empty>

<type>              => <simple type>
                    |  <array type>

<array type>        => "array" "[" [<integer expr>] "]" "of" <simple type>

<simple type>       => <type id>

<block>             => "begin" <statement> {";" <statement>} [";"] "end"

<statement>         => <simple statement>
                    |  <structured statement>
                    |  <var-declaration>

<empty>             =>

----------------------------------------------------------------------------------------
// Simple statements

<simple statement>  => <assignment statement>
                    |  <call>
                    |  <return statement>
                    |  <read statement>
                    |  <write statement>
                    |  <assert statement>

<assignment statement> => <variable> ":=" <expr>

<call>              => <id> "(" <arguments> ")"

<arguments>         => expr { "," expr }
                    |  <empty>

<return statement>  => "return" [ expr ]

<read statement>    => "read" "(" <variable> { "," <variable> } ")"

<write statement>   => "writeln" "(" <arguments> ")"

<assert statement>  => "assert" "(" <boolean expr> ")"

----------------------------------------------------------------------------------------
// Structured statements

<structured statement> => <block>
                       |  <if statement>
                       |  <while statement>

<if statement>      => "if" <boolean expr> "then" <statement>
                    |  "if" <boolean expr> "then" <statement> "else" <statement>

<while statement>   => "while" <boolean expr> "do" <statement>

----------------------------------------------------------------------------------------
// Expressions

<expr>              => <simple expr>
                    |  <simple expr> <relational operator> <simple expr>

<simple expr>       => [ <sign> ] <term> { <adding operator> <term> }

<term>              => <factor> { <multiplying operator> <factor> }

<factor>            => <call>
                    |  <variable>
                    |  <literal>
                    |  "(" <expr> ")"
                    |  "not" <factor>
                    |  < factor> "." "size"

<variable>          => <variable id> [ "[" <integer expr> "]" ]

----------------------------------------------------------------------------------------
// Operators

<relational operator> => "=" | "<>" | "<" | "<=" | ">=" | ">"

<sign>              => "+" | "-"

<negation>          => "not"

<adding operator>   => "+" | "-" | "or"

<multiplying operator> => "*" | "/" | "%" | "and"
```

## Lexical grammar

```
// Variable name
<id>              => <letter> { <letter> | <digit> | "_" }

----------------------------------------------------------------------------------------
// Literals
<literal>         => <integer literal> | <real literal> | <string literal>

<integer literal> => <digits>

<digits>          => <digit> { <digit> }

<real literal>    => <digits> "." <digits> [ "e" [ <sign> ] <digits>]

<string literal>  => "\"" { < a char or escape char > } "\""

----------------------------------------------------------------------------------------
// Symbols
<letter>          => a | b | c | d | e | f | g | h | i | j | k | l | m | n | o
                  |  p | q | r | s | t | u | v | w | x | y | z | A | B | C | D
                  |  E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S
                  |  T | U | V | W | X | Y | Z

<digit>           => 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

----------------------------------------------------------------------------------------
// Special symbols and keywords

<special symbol or keyword> => "+" | "-" | "*" | "%" | "=" | "<>" | "<" | ">"
                            |  "<=" | ">=" | "(" | ")" | "[" | "]" | ":=" | "."
                            |  "," | ";" | ":" | "or" | "and" | "not" | "if"
                            |  "then" | "else" | "of" | "while" | "do" | "begin"
                            |  "end" | "var" | "array" | "procedure" | "function"
                            |  "program" | "assert" | "return"

<predefined id>   => "Boolean" | "false" | "integer" | "read"
                  |  "real" | "size" | "string" | "true" | "writeln"
```

## Tokens

|     token      | regex or token                   | explanation                       |
| :------------: | :------------------------------- | :-------------------------------- |
|  **Keywords**  |
|       OR       | `or`                             | Comparsion of two booleans        |
|      AND       | `and`                            | Comparsion of two booleans        |
|      NOT       | `not`                            | Negation                          |
|       IF       | `if`                             | Start of if statement             |
|      THEN      | `then`                           | Part of if statement              |
|      ELSE      | `else`                           | Part of if statement              |
|       OF       | `of`                             | Array type assignment             |
|     WHILE      | `while`                          | Start of while loop               |
|       DO       | `do`                             | Part of while loop                |
|     BEGIN      | `begin`                          | Start of code block               |
|      END       | `end`                            | End of code block                 |
|      VAR       | `var`                            | Variable declaration              |
|     ARRAY      | `array`                          | Array declaration                 |
|   PROCEDURE    | `procedure`                      | Start of a procedure              |
|    FUNCTION    | `function`                       | Start of a function               |
|    PROGRAM     | `program`                        | Start of the program              |
|     ASSERT     | `assert`                         | Assert statement                  |
|     RETURN     | `return`                         | Return statement                  |
| **Arithmetic** |
|      ADD       | `+`                              | Addition and string concatenation |
|      SUB       | `-`                              | Subtraction                       |
|      MULT      | `*`                              | Multiplication                    |
|      MOD       | `%`                              | Modulo (remainder)                |
|      DIV       | `/`                              | Division                          |
| **Relational** |
|       EQ       | `=`                              | Equality comparsion               |
|       NE       | `<>`                             | Not equal comparsion              |
|       LT       | `<`                              | Less than comparsion              |
|       GT       | `>`                              | Greater than comparsion           |
|      LTE       | `<=`                             | Less than or equal comparsion     |
|      GTE       | `=>`                             | Greater than or equal comparsion  |
|  **Literals**  |
|     IDENT      | r`[a-zA-Z][a-zA-Z0-9_]*`         | Identifier                        |
|    INTEGER     | r`[0-9]+`                        | Integer literal                   |
|      REAL      | r`[0-9]+\.[0-9]+(e[+-]?[0-9]+)?` | Real number literal               |
|     STRING     | r`\"(\\.|[^"\\])*\"`             | String literal                    |
|   **Other**    |
|   LEFT_PAREN   | `(`                              | Begin nested expression           |
|  RIGHT_PAREN   | `)`                              | End nested expression             |
|  LEFT_BRACKET  | `[`                              | Begin array                       |
| RIGHT_BRACKET  | `]`                              | End array                         |
|     ASSIGN     | `:=`                             | Variable assignment               |
|      DOT       | `.`                              | Integer-decimal separator         |
|     COMMA      | `,`                              | Array element separator           |
|   SEMICOLON    | `;`                              | End of statement                  |
|     COLON      | `:`                              | Variable type assignment          |
|   SCAN_ERROR   | _Anything not matching above_    | Unexpected token                  |
|      EOF       |                                  | End of file                       |
