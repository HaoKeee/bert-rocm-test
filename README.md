# 使用说明
sentiment.py用于训练，predict.py用于预测。

## 部分代码说明
### 构造数据集
使用utils.build_dataset()，可以在该函数的基础上针对当前数据的格式以及需要得到的输出进行修改。

本例中使用到的训练数据是xlsx格式保存的，如下图所示，按区间进行分类，大于70一类，大于50小于70一类，大于20小于50一类，以此类推，类别存放在“./sentiment/data/class.txt”中
![img](img_1.png)
最后构造的数据一共包含4个部分，裁剪后的转为数字列表的样本，标签，该样本的长度，以及根据长度构造的mask。  
具体的构成可以进行调试查看。

### 对Config的部分补充说明
使用“./models/sentiment_bert.py”中的Config类，大部分训练用到的参数都可以直接通过配置config进行修改。  
 - class_list：通过“./sentiment/data/class.txt”直接生成的一个列表
 - require_improvement：训练时，用于判断提前终止训练的轮次，这里设置为1000表示训练1000个batch后若效果还没有提升就提前终止
 - pad_size：将每条样本都处理成固定的长度，方便训练

### 对build_iterator()的补充说明
通过该函数构造一个数据生成器，输入给模型进行预测或训练，predict参数用于指定是否需要构造标签。  
每次输出的数量是一个batch的量，如果想要每次只要一个样本，则在config中修改batch_size为1即可。  
另外也可以直接构造，不适用该方法，只要能和目前的格式保持一致即可。

### 对训练的补充说明
对于pad_size目前还没有调试出一个较优的值，可以根据具体的数据情况多次测试调优。

训练结果保存在“./sentiment/saved_dict/bert.ckpt”中，加载模型的方法如下
```commandline
    config = Config('sentiment')
    model = Model(config)
    model.load_state_dict(torch.load(config.save_path))           # 加载模型
```

目前没有做连续训练的功能，该功能需要考虑的问题较复杂，后续的数据量的问题，数据的分布问题，训练效果的评估都会比较麻烦。

由于现阶段数据集还是一个较大的问题，数据分布不均匀、数据只包含标题都会导致训练效果较差，并且由于极端数据较少，最后的测试函数会出现bug，这是由于测试数据中部分类别为空，且验证集一定程度上能够反映训练效果，我就没有再对测试函数进行调试，如想要使用该函数，可以花时间再调试一下。

### 对预测的补充说明
如果遇到数据量过大的情况，可以在以下两行代码之间添加保存的代码，在调用build_dataset_predict()时，按一定量对数据进行拆分，然后保存。  
需要使用时先读取保存的小批量数据再调用build_iterator()即可。
```commandline
 17   datas = build_dataset_predict(config)                        # 构造数据
 18   datas_iter = build_iterator(datas, config, predict=True)
```

### 对预测结果的补充说明
输出的结果为int型，包括“./sentiment/data/class.txt”中提到的那些数字。  
结果全部保存在predicts这个列表中。#   b e r t - r o c m - t e s t  
 