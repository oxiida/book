# Cheatsheet

### Builtin Tasks and Operations

| **Category**            | **Description**                                                                                               |
|------------------------ |--------------------------------------------------------------------------------------------------------------|
| Binary operations       | `+`, `-`, `*`, `/`, `^`                                                                                      |
| Unary operations        | `+`(number/string), `-`(numbers only), `!`(booleans)                                                      |
| Comparison operations   | `>`, `>=`, `<`, `<=`, `==`, `!=`                                                                             |
| Logic operations        | `and`, `or` (short-circuit)                                                                                  |
| `shell`                 | Run shell command                                                                                            |
| `shellpipe`             | Run shell pipeline (syntax sugar for `shell`)                                                                |

---

### Builtin Types

| **Type**   | **Description**                                                                                                                                              |
|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Number     | internal representation will be a `f64` (for both integers and floats)                                                                                    |
| Boolean    | either `true` or `false`                                                                                                                                  |
| String     | Anything inside double quotes                                                                                                                                |
| Array      | Same type items in `[]`, separated by `,` (e.g., `["monday", "sunday"]`)                                                                                    |
| Mapping    | Not directly defined or passed to functions, but can be returned. Access attributes using `.` by key (e.g., `object.key`).                                  |

---

## Grammar Rules

The detail grammar rules, only come here when something is really hard to figure out and you want to figure it out by yourself.
Otherwise, join and ask me in [Oxiida Zulip](http://bit.ly/44wK00z).

### Program structure

| Non-terminal | Production |
|--------------|------------|
| **Program**  | `Statement*` |

---

### Statements

| Non-terminal | Production |
|--------------|------------|
| **Statement** | `SimpleStmt ";"`  \|  `Block`  \|  `IfStmt`  \|  `WhileStmt`  \|  `ForStmt` |
| **SimpleStmt** | `Expression`  \|  `"print" Expression`  \|  `"require" IdentifierList` |
| **Block** | `"seq"  "{" Statement* "}"`<br>`"para" "{" Statement* "}"`<br>`"{" Statement* "}"` |
| **IfStmt** | `"if" "(" Expression ")" Statement`<br>`"if" "(" Expression ")" Statement "else" Statement` |
| **WhileStmt** | `"while" "(" Expression ")" Statement` |
| **ForStmt** | `"for" Expression "in" Array Statement` |
| **IdentifierList** | `identifier ( "," identifier )*` |

---

### Expressions — precedence (low → high)

| Level | Non-terminal | Production |
|-------|--------------|------------|
| 8 | **Assignment** | `Shell "=" Expression`  \|  `Logic` |
| 7 | **Logic** | `Equality { ( "and" \| "or" ) Equality }` |
| 6 | **Equality** | `Compare { ( "==" \| "!=" ) Compare }` |
| 5 | **Compare** | `Term { ( ">" \| ">=" \| "<" \| "<=" ) Term }` |
| 4 | **Term** | `Factor { ( "+" \| "-" ) Factor }` |
| 3 | **Factor** | `Power { ( "*" \| "/" ) Power }` |
| 2 | **Power** | `Unary { "^" Unary }` |
| 1 | **Unary** | `( "+" \| "-" \| "!" ) Unary`  \|  `Attribute` |

---

### Primary & postfix

| Non-terminal | Production |
|--------------|------------|
| **Attribute** | `CallPostfix { "." identifier }` |
| **CallPostfix** | `Primary { "(" [ Expression { "," Expression } ] ")" }` |
| **Primary** | `"true"` \| `"false"` \| `"nil"`<br>`number` \| `identifier` \| `string`<br>`"(" Expression ")"`<br>`Array` \| `Shell` \| `PipeShell` |

---

### Composite literals & shell constructs

| Non-terminal | Production |
|--------------|------------|
| **Array** | `"[" [ Shell { "," Shell } ] "]"` |
| **PipeShell** | `"shellpipe" "{" PipeShellExpr "}"` |
| **PipeShellExpr** | `PipeShellExpr "|" string string*`<br>`ShellUnit` |
| **ShellUnit** | `string string*` |
| **Shell** | `"shell" "{" Term [, "[" Term { "," Term } "]"] [, Shell] "}"`  \|  `Logic` |

---

### Tokens / Lexical categories

| Category | Lexemes (examples) |
|----------|-------------------|
| **Keywords** | `if`, `else`, `while`, `for`, `seq`, `para`, `print`, `require`, `and`, `or`, `in`, `true`, `false`, `nil`, `shell`, `shellpipe` |
| **Operators** | `=`  `==`  `!=`  `<`  `<=`  `>`  `>=`  `+`  `-`  `*`  `/`  `^`  `!`  `.` |
| **Delimiters** | `(  )  {  }  [  ]` |
| **Literals** | `number` (f64), `string` |
| **Identifiers** | `identifier` |

---

### Operator-precedence recap

| Precedence | Operators |
|------------|-----------|
| 1 (highest) | `+  -  !` (unary) |
| 2 | `^` |
| 3 | `*  /` |
| 4 | `+  -` (binary) |
| 5 | `<  <=  >  >=` |
| 6 | `==  !=` |
| 7 | `and  or` |
| 8 (lowest) | `=` |
