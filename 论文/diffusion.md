# ControlNet

原文[Adding Conditional Control to Text-to-Image Diffusion Models](https://arxiv.org/pdf/2302.05543.pdf)

ControlNet的作用是

```
a neural network architecture to add spatial conditioning controls to large, pretrained text-to-image diffusion models
```



**zero convolution**：即zero-initialized convolution layers，用0来初始化网络权重，然后从0开始逐渐地变化，从而避免harmful noise影响finetuning。

现存的text-to-image存在的问题：单单以文本形式作为prompt提供的信息是有限的，需要对于prompt进行多次的重复编辑，因此希望引入图片作为prompt，提供更加精细的控制信息。



现存的几种微调网络的方法：

- **HyperNetwork**
- **Adapter** Adapter方法是通过添加Adapter Layer，不同的下游任务有不同的Adapter layer。
- **Additive Learning** 
- **Low-Rank Adaptation (LoRA)**
- **Zero-Initialized Layers**