# Python相关问题



# 1. distro-info问题

问题描述

```

```



# 2. PIP和PIP3

pip和pip3是为了区别python和python3

如果电脑只安装了python3，pip和pip3是一样的。

但是如果电脑安装了python2，那么pip安装会把包安装在python2.7/site-packages下而pip3会安装在python3.7/site-packages下。



**为什么sudo -H pip install找不到但是sudo -H pip3 install找得到命令**

# 3. Python3.8-disutils



# 4. Python中的Wheel



# 5. [The following packages have unmet dependencies!](https://askubuntu.com/questions/563178/the-following-packages-have-unmet-dependencies)



# 6. Python虚拟环境

python虚拟环境管理有多种工具：

1. **pyenv** 用于管理不同的python版本
2. **virtualenv** 用于管理不同的python环境
3. **conda** 用于管理不同的python环境
4. **venv** 用于管理不同的python环境



# 7. apt下载的包为什么能用pip3运行



# 8. python与pip之间的依赖问题





# 9. OSError: /home/zhangkj/anaconda3/envs/nerf-torch/lib/python3.8/site-packages/nvidia/cublas/lib/libcublas.so.11: undefined symbol: cublasLtGetStatusString, version libcublasLt.so.11

**原因**：pytorch1.13会自动安装nvidia_cublas_cu11，nvidia_cuda_nvrtc_cu11，nvidia_cuda_runtime_cu11和nvidia_cudnn_cu11，但是如果系统预先装好了CUDA这几项是都有的。



**解决办法**：`pip uninstall nvidia_cublas_cu11`





# 10. AttributeError: module 'collections' has no attribute 'Iterable'



# 11. egg_info
