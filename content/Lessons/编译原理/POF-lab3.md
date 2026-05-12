中间代码生成

| Instruction             | Description                          |
| ----------------------- | ------------------------------------ |
| `LABEL x:`              | Define label `x`                     |
| `FUNCTION f:`           | Define function `f`                  |
| `x := y`                | Assign `y` to `x`                    |
| `x := y + z`            | Arithmetic operations (+, -, *, /)   |
| `x := &y`               | Take address of `y`                  |
| `x := *y`               | Dereference pointer `y`              |
| `*x := y`               | Write to address in `x`              |
| `GOTO x`                | Unconditional jump                   |
| `IF x [relop] y GOTO z` | Conditional jump                     |
| `RETURN x`              | Return value from function           |
| `DEC x [size]`          | Allocate memory (for arrays/structs) |
| `ARG x`                 | Pass argument to function            |
| `x := CALL f`           | Call function and store result       |
| `PARAM x`               | Declare function parameter           |
| `READ x`                | Read integer from console            |
| `WRITE x`               | Write integer to console<br>         |
|                         |                                      |
