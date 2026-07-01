## 进程间通信- `pipe`

- get knowledge by:
```
man -P cat 2 pipe | claude summarize
```
then you get:


⏺ Summary of pipe (2) System Call

  Purpose: pipe () creates a unidirectional communication channel (pipe) for interprocess communication, returning a pair of file descriptors for reading and writing.

  Interface:
```c
  #include <unistd.h>
  int pipe (int filedes[2]);
  - filedes[0]: points to the read end of the pipe
  - filedes[1]: points to the write end of the pipe
```
  Key Behavior:
  - Data written to the write end (`filedes[1]`) can be read from the read end (` filedes[0]`), enabling output from one process to be passed directly as input to another.
  - The pipe persists in the system until all associated file descriptors are closed.
  - A pipe with one end closed is called "widowed":
    - Writing to a widowed pipe (read end closed) generates a `signal`  to the writing process (this can be disabled via fcntl `F_SETNOSIGPIPE`).
    - Reading from a widowed pipe (write end closed) returns EOF (0 count) after all buffered data is consumed.

  Return Values:
  - Returns 0 on successful pipe creation
  - Returns -1 on error and sets errno to indicate the error type

  Possible Errors:
  - EFAULT: The provided filedes buffer points to an invalid memory address
  - EMFILE: Too many open file descriptors for the current process
  - ENFILE: The system-wide file table is full

  Context: First introduced in Version 6 AT&T UNIX. Related functions: sh (1), fork (2), read (2), write (2), fcntl (2), socketpair (2).



# 正则表达式 (regex. h)

```c
typedef struct {
              regoff_t rm_so;
              regoff_t rm_eo;
          } regmatch_t;
```
   Each  `rm_so` element that is not -1 indicates the start offset of the next largest substring match within the string.  The relative `rm_eo` element indicates the end offset of the match, which is the offset of the first character after the match‐ ing text.

这里的 rm_so 是 match 的开始下标，rm_eo 是结束下标，可以获取 substr

用 `regcomp` 编译正则表达式，注意 C 语言的 `\\` 才能表达正则表达式中的 `\`
用 `regexec` 执行正则匹配，匹配结果通过 `pmatch[]` 数组返回

