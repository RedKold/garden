```
lldb: $(NAME)
		lldb ./$(NAME) -- -m maps/map.txt -p 1 --version
```
这样就可以传递给 lldb 一个带参数的程序，中间加-- 隔开以区分是给 lldb 的参数还是 executable file 的参数

gdb 是同理的

在算法竞赛中，更常用的一种情形是用 gdb 调试程序，同时用 `<` 重定向输入文件，这时候可以用

```
gdb ./test.out 	# 进入gdb
(gdb) run < 1.in # 在gdb中重定向
```

