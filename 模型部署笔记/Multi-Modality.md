# CLIP

CLIP牵扯到了非常多的其它工作：有监督，自监督，无监督，弱监督。

有监督

无监督/自监督：对比学习，GPT，BERT

自监督的训练方式是与下游任务无关的，只是为了得到一个泛化特征。





采用自然语言作为监督型号：

1. 不需要标注，很容易获得很大规模的文本数据，并且文本的形式非常自由
2. 文本+图片多模态特征，容易做zero-shot



但是目前的困难是并没有一个高质量，大规模的text-image pair的数据集。



prompt template