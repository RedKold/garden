
- 建立 ssh 端口映射的途径。
```
ssh -N \
  -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 \
  -L 127.0.0.1:6185:127.0.0.1:6185 \
  -L 127.0.0.1:6099:127.0.0.1:6099 \
  kasumi
```