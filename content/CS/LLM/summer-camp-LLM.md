## assignment 0

firstly, install docker on wsl2 (for it will use your gpu).

```bash
docker run --gpus all -it --rm nvcr.io/nvidia/pytorch:26.06-py3
```

this is the newest docker image from NVIDIA torch.

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260717162243.png)


This ends the installation.

挂载到 docker 中运行：
```bash
cd ~/summer-camp/assignments/a0-RedKold
docker run --gpus all -it --rm -v $(pwd):/workspace nvcr.io/nvidia/pytorch:26.06-py3
```


- [[Pytorch]]