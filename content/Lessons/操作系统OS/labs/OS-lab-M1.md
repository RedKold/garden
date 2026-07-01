## Use getopt_long to parse the args
> [!Question] the third param optstring's ": " is what?
In `optstring`, `:` after a letter means **that option requires an argument**.

```
"vf:"
 │└─ 'f' requires an argument  → -f value  or  -fvalue
 └── 'v' has no argument       → -v
```

**Positions matter:**

|Pattern|Meaning|
|---|---|
|`"f:"`|`-f` requires an argument|
|`"f::"`|`-f` has an optional argument (GNU extension)|
|`":f:"`|leading `:` — return `':'` instead of `'?'` on missing arg|

The leading `:` is special: it changes error behavior so you can distinguish "unknown option" (`'?'`) from "missing argument" (`':'`).


In `getopt_long`, we need to form a struct array

```c
struct option {
   char *name;
   int has_arg;
   int *flag;
   int val;
};
```

> [!Note] 关于 options 结构体数组的细节
 > The last element of the longopts array has to be filled with zeroes.
 > 否则程序可能无法终止


> [!Question] 怎么检测多余的参数？或者缺少的参数？
> In our problem, we wish user type `--version`  for version info, but if user type `--version 233`, we want to detect this type of error, to tell user you we don't expected a argument for `--version`.
> `getopt` and `getopt_long` all provide an extern variable to do it. `optind`, it's the *index* of *opt* in the parsing process.  
> In a normal parsing, we expected when the parse done, *optind* is equal to *argc*.
> So if `argc` is not equal to `optind` , instead, greater than, we detect a no-need argument.
> ```c
> if(argc > optind){
> 	return 1;
> }
> else{
> 	printVersion();
> }
> ```

# Use testkit

You can type the following command to test your code:
```sh
TK_RUN=1 ./labyrinth  
```

