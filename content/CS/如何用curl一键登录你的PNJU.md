If you are a student from Nanjing University, you may be mad about enter [PNJU](p.nju.edu.cn) each time you want to surf the internet inside campus. Although ITSC provided "non-sensible"  service, it support only up to 4 devices, and your devices default using *dynamic MAC* such as iPad and Mac cannot login in by that way.

But, a shell script solve them all.
```bash
curl https://p.nju.edu.cn/api/portal/v1/login -X POST -d '{"username": "<your student id as 2312xxx", "password"<your password>""}'
```


