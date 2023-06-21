# runc-nvidia-test

Steps to reproduce:

- Pull this repo.
- Install `recvtty` (ensure `$GOPATH/bin` is part of your regular `$PATH`):
```sh
go install github.com/opencontainers/runc/contrib/cmd/recvtty@latest
```
- Create `rootfs` directory:
```sh
mkdir rootfs
```
- Pull Ubuntu base image and extract it to `rootfs` (also indicated in `nvidia-container-runtime` [here](https://github.com/NVIDIA/nvidia-container-runtime/blob/6f328ae638853b5b295deabfa08becc74f7273ef/README.md?plain=1#LL17C109-L17C144)):

```sh
curl -sS https://cdimage.ubuntu.com/ubuntu-base/releases/20.04/release/ubuntu-base-20.04.1-base-amd64.tar.gz | tar --exclude 'dev/*' -C rootfs -xz
```
- In one terminal, create the TTY socket and wait for the connection:

```sh
recvtty tty.sock
```
- In another terminal, create the container. The hook is called at this point and all Nvidia dependencies become available on the `rootfs`:
```
sudo runc create --console-socket $(pwd)/tty.sock container
```
- `rootfs` before `runc create` -no Nvidia deps-:
```sh
paperspace@psal6i8au:~/go/src/github.com/matiasinsaurralde/runc-nvidia-test$ mkdir rootfs
paperspace@psal6i8au:~/go/src/github.com/matiasinsaurralde/runc-nvidia-test$ curl -sS https://cdimage.ubuntu.com/ubuntu-base/releases/20.04/release/ubuntu-base-20.04.1-base-amd64.tar.gz | tar --exclude 'dev/*' -C rootfs -xz
paperspace@psal6i8au:~/go/src/github.com/matiasinsaurralde/runc-nvidia-test$ find rootfs | grep nvidia
paperspace@psal6i8au:~/go/src/github.com/matiasinsaurralde/runc-nvidia-test$ 
```
- `rootfs` after `runc create` -the hook works-:
```sh
paperspace@psal6i8au:~/go/src/github.com/matiasinsaurralde/runc-nvidia-test$ find rootfs/ | grep nvidia
rootfs/usr/bin/nvidia-cuda-mps-control
rootfs/usr/bin/nvidia-smi
rootfs/usr/bin/nvidia-persistenced
rootfs/usr/bin/nvidia-cuda-mps-server
rootfs/usr/bin/nvidia-debugdump
rootfs/usr/lib/firmware/nvidia
rootfs/usr/lib/firmware/nvidia/515.105.01
rootfs/usr/lib/firmware/nvidia/515.105.01/gsp.bin
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-cfg.so.1
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-opencl.so.515.105.01
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-allocator.so.515.105.01
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-allocator.so.1
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-compiler.so.515.105.01
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-opencl.so.1
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-ptxjitcompiler.so.1
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-cfg.so.515.105.01
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-ptxjitcompiler.so.515.105.01
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1
rootfs/usr/lib/x86_64-linux-gnu/libnvidia-ml.so.515.105.01
rootfs/run/nvidia-persistenced
rootfs/run/nvidia-persistenced/socket
```
- Start the container:
```
sudo runc start container
```
- `recvtty` opens up the container terminal, we should be able to run `nvidia-smi`:

```sh
# nvidia-smi
nvidia-smi
Wed Jun 21 11:23:57 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 515.105.01   Driver Version: 515.105.01   CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro M4000        Off  | 00000000:00:05.0  On |                  N/A |
| 46%   32C    P8    16W / 120W |    190MiB /  8192MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+

```