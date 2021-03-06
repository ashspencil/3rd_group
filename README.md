# Just GPU Driver and CUDA install step

## 導覽
以下操作皆在Ubuntu 16.04環境下進行,並已接上GPU
在此系統都已抓取到GPU,且無其他錯誤回報
按照以下目錄操作

目錄待補
### 軟體環境安裝

由於需要安裝Nvidia自家的驅動,因此並不需要此預設的開源驅動,要先關閉他
```
$ sudo echo "blacklist nouveau" >> /etc/modprobe.d/blacklist-nouveau.conf
$ sudo echo "options nouveau modeset=0" >> /etc/modprobe.d/blacklist-nouveau.conf  
```

重新build kernel
```
$ sudo update-initramfs -u
```

抓取Nvidia Cuda 在這裡直接用wget,方便省事
```
$ wget https://developer.nvidia.com/compute/cuda/9.0/Prod/local_installers/cuda_9.0.176_384.81_linux-run
```

安裝,參數建議依自身需求去下
```
$ chmod +x cuda_9.0.176_384.81_linux-run
$ sudo ./cuda_9.0.176_384.81_linux-run --silent --driver --toolkit
```

安裝完後應該要有成功回報,然後把LIBRARY抓出來,把以下寫進.bashrc
```
export PATH=/usr/local/cuda-9.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64:$LD_LIBRARY_PATH
```

接著看你要重登或是
```
$ exec bash
```
然後檢查一下
```
＄ cat /proc/driver/nvidia/version
NVRM version: NVIDIA UNIX x86_64 Kernel Module  384.81  Sat Sep  2 02:43:11 PDT 2017
GCC version:  gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.9) 

＄ nvidia-smi （-a）
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 384.81                 Driver Version: 384.81                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Quadro K620         Off  | 00000000:01:00.0 Off |                  N/A |
| 34%   45C    P0     2W /  30W |      0MiB /  1998MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Quadro K620         Off  | 00000000:03:00.0 Off |                  N/A |
|  0%   36C    P0     1W /  30W |      0MiB /  2000MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

```
最重要的NVCC
```
＄ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2017 NVIDIA Corporation
Built on Fri_Sep__1_21:08:03_CDT_2017
Cuda compilation tools, release 9.0, V9.0.176

```
都有與硬體資訊相同的話就大功告成


### 安裝Nvidia-docker 

參考[官方](https://github.com/NVIDIA/nvidia-docker)

### 測試軟硬體

NV_GPU 依自己的需求去下,但官方似乎已不建議使用此參數,如果不下的話,會全部GPU都吃,新的下法後面會提到
```
$ (NV_GPU=?) nvidia-docker run --rm -ti ryanolson/device-query
```
若有跑出正確資訊,則軟硬體環境皆沒問題,且Nvidia-docker環境也正常,感謝黃老大!

若有跑出
```
CUDA driver version is insufficient for CUDA runtime version
```
檢查一下你的驅動可能過舊,不可低於runtime_driver

### 試跑Tensorflow

簡單撈了一份IMG來試
```
$ nvidia-docker run -it (-p 8888:8888) gcr.io/tensorflow/tensorflow:latest-gpu-py3 /bin/bash
```
8888port是jupyter要用的,暫且不管他,也可不下

進來後可以先指定GPU
```
$ export CUDA_VISIBLE_DEVICES='?'
```

找了一份測試專案 來源為:[tobegit3hub](https://github.com/tobegit3hub/tensorflow_template_application)
```
$ curl https://codeload.github.com/tobegit3hub/deep_recommend_system/zip/master -o test.zip
$ unzip test.zip
$ cd tensorflow_template_application-master/
```
接著按照此作者提供的流程即可

### Contribution
以上操作流程皆從網路上整理且自己修改指令而來,感謝其他作者,也歡迎自由編輯,若有錯誤或不清楚之處也歡迎發issue告知
