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