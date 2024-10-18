# HuggingFace

HuggingFace就是机器学习界的Github，这样类比就够了。非常好用。



# 1. HuggingFace下载模型方法

参考：

1. [如何快速下载huggingface模型——全方法总结 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/663712983)



在huggingface上下载模型和数据集有多种方法，各有优劣：

1. 基于URL，基于URL就是利用下载**直链**直接在浏览器或者下载器中下载，利用下载器可以使用IDM，Aria2等多线程下载器下载。除此之外，还有一个开源的huggingface专用下载器hfd。
2. CLI工具，一些cli工具，比如git也能够实现下载。利用git下载模型/数据集，由于存在大文件，但是git对大文件的支持并不是特别友好，所以需要采用`git lfs`特性。
3. 专用CLI工具，huggingface官方给出了下载工具`huggingface-cli`与`hf_transfer`。
4. python，最后就是利用python的API，`snapshot_download`，`from_pretrained`和`hf_hub_download`等下载。



访问huggingface需要魔法，因此一般通过镜像站下载。通过设置**HF_ENDPOINT**环境变量来设置镜像站 

```python
import os
os.environ['HF_ENDPOINT'] = 'https://hf-mirror.com'
```

注意`os.environ`得在import huggingface库相关语句之前执行。



huggingface



# 2. HuggingFace accelerate



# 3. Trainer



# 4. transformers库

`ModelOutput`是所有模型输出的基类

`@dataclass`装饰器





`PretrainedModel`继承自pytorch的`nn.Module`，是所有预训练模型的基类，同时混合了几个不同的 mixin 类，如 `ModuleUtilsMixin`、`GenerationMixin`、`PushToHubMixin` 和 `PeftAdapterMixin`。



以Llama为例，

