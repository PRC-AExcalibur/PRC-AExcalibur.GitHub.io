---
title:  【深度学习】PyTorch 快速入门
categories:
- DeepLearning
tags:
- ComputerScience 
- DeepLearning 
---
PyTorch 快速入门教程。


---
# PyTorch 快速入门
PyTorch 是一个强大的开源机器学习库，特别适用于深度学习应用。它提供了灵活的神经网络构建能力以及GPU加速功能，被广泛应用于研究和生产环境。

本教程介绍一下 PyTorch 的基本使用流程，方便读者快速入门。默认读者已经装好了 PyTorch 环境。

针对MNIST数据集我们会给出一个示例。

### 1. 数据准备
在PyTorch中，数据的准备通常涉及几个关键步骤：定义数据转换、创建数据集、以及构建数据加载器。

#### 数据预处理
`torchvision.transforms`模块提供了许多用于图像数据预处理的工具。

在这个例子中，我们使用两个转换：
1. `ToTensor()`: 将PIL图像或numpy数组转换为张量。
2. `Normalize(mean, std)`: 使用给定的均值和标准差对张量图像进行标准化。对于MNIST数据集，由于其灰度图像的范围在0到1之间，我们使用(0.5,)作为均值和(0.5,)作为标准差，使得数据的分布中心位于0。

```python
from torchvision import datasets, transforms

# 定义数据转换
transform = transforms.Compose([
    transforms.ToTensor(),  # 转换为张量
    transforms.Normalize((0.5,), (0.5,))  # 归一化
])
```

#### 创建数据集
`torchvision.datasets`提供了许多常见的数据集，包括MNIST。
通过`datasets.MNIST()`函数，我们可以轻松地加载MNIST数据集。
我们设定`root`参数为我们想要保存数据集的路径（这里是当前目录下的`dataset`文件夹），`download=True`意味着如果数据集不存在，它将被下载并存储在这个位置，`train=True`表示我们正在加载训练集。

```python
# 创建数据集实例
trainset = datasets.MNIST(root='./dataset',  # 指定数据集保存路径
                          download=True,  # 如果数据集不存在，则下载
                          train=True,  # 加载训练集
                          transform=transform)  # 应用数据转换
```

#### 构建数据加载器
数据加载器`DataLoader`从数据集中抽取样本，并将它们打包成批。
它还支持数据打乱，这对于训练神经网络是很有帮助的。
我们设置`batch_size=64`，这意味着每次迭代将返回64个样本。
`shuffle=True`确保了每个epoch开始时，数据会被随机重新排序，避免了学习顺序上的依赖性。

```python
# 创建数据加载器
trainloader = torch.utils.data.DataLoader(trainset,
                                          batch_size=64,  # 批量大小
                                          shuffle=True)  # 每个epoch开始时随机重排数据
```

现在，`trainloader`就可以用来迭代训练数据集的批次，这些批次可以直接送入模型进行训练。


### 2. 模型定义
在深度学习项目中，模型定义是核心部分之一，它决定了神经网络的结构和行为。
在PyTorch中，模型通常通过继承`torch.nn.Module`类并定义`forward`方法来创建。

下面详细介绍这段代码，以构建一个多层感知器（MLP）为例：

``` python
import torch.nn as nn
import torch.nn.functional as F

class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(784, 128)
        self.fc2 = nn.Linear(128, 64)
        self.fc3 = nn.Linear(64, 10)

    def forward(self, x):
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return F.log_softmax(x, dim=1)
```
#### 创建模型类
我们定义了一个名为`Net`的类，它继承自`torch.nn.Module`。这意味着`Net`将具有所有`Module`类的功能，包括参数追踪和反向传播机制。

在`Net`类中，我们首先定义了初始化方法`__init__`，并在其中创建了神经网络的层。`super(Net, self).__init__()`这一行是必须的，它调用了父类`nn.Module`的初始化方法，确保了正确初始化模型的所有属性。

接下来，我们定义了三个全连接（`Linear`）层，它们构成了我们的多层感知器：

- `self.fc1 = nn.Linear(784, 128)`: 第一层接收784维输入（对应于MNIST图像的28x28像素），并将其转换为128维的向量。
- `self.fc2 = nn.Linear(128, 64)`: 第二层接收前一层的128维输出，并将其转换为64维的向量。
- `self.fc3 = nn.Linear(64, 10)`: 最后一层接收64维输入，并产生10维输出，对应MNIST数据集中的10个类别。

#### 前向传播
`forward`方法定义了数据流过网络的过程。
在这个方法中，我们首先将输入传递给第一层`fc1`，然后通过ReLU激活函数增加非线性。
接着，数据继续传递给`fc2`层，再次经过ReLU激活，最后通过`fc3`层。
输出经过log_softmax函数，将其转换为对数概率分布，这对于交叉熵损失函数的计算非常有用。


### 3. 训练模型
#### 定义损失函数
选择一个适合你任务的损失函数，如交叉熵损失`nn.CrossEntropyLoss`。

#### 优化器
选择一个优化算法，如随机梯度下降`torch.optim.SGD`或Adam`torch.optim.Adam`。

#### 训练循环
迭代数据集，前向传播，计算损失，反向传播，并更新权重。

示例代码如下：
```python
model = Net()
criterion = nn.NLLLoss()
optimizer = torch.optim.SGD(model.parameters(), lr=0.003)

epochs = 5
for e in range(epochs):
    running_loss = 0
    for images, labels in trainloader:
        images = images.view(images.shape[0], -1)
        optimizer.zero_grad()
        
        output = model(images)
        loss = criterion(output, labels)
        loss.backward()
        optimizer.step()
        
        running_loss += loss.item()
    else:
        print(f"Training loss: {running_loss/len(trainloader)}")
```

### 4. 测试模型

在训练完模型之后，对其进行测试是非常重要的一步，以评估模型在未见过的数据上的泛化能力。

以下是在PyTorch中测试模型的一般步骤：

#### 加载测试数据集
首先，你需要加载测试数据集，这类似于加载训练数据集，但要确保`train=False`，以便加载测试集。
同时，应用相同的预处理步骤，如归一化和转为张量。

```python
test_transforms = transforms.Compose([transforms.ToTensor(),
                                      transforms.Normalize((0.5,), (0.5,))])
testset = datasets.MNIST('./dataset/MNIST_data/', download=True, train=False, transform=test_transforms)
testloader = torch.utils.data.DataLoader(testset, batch_size=64, shuffle=True)
```

#### 设置模型为评估模式
在测试之前，你需要将模型设置为评估模式。这是因为像Batch Normalization和Dropout这样的层在训练和评估阶段的行为不同。
评估模式下，Batch Normalization使用整个训练集的统计信息，而Dropout则关闭。

```python
model.eval()
```

#### 测试循环
遍历测试数据集，对每个批次的数据进行前向传播，并计算准确率或其它性能指标。
注意，在测试阶段不需要计算梯度，因此可以使用`torch.no_grad()`上下文管理器来提高效率和减少内存消耗。

```python
correct = 0
total = 0

with torch.no_grad():
    for images, labels in testloader:
        images = images.view(images.shape[0], -1)
        outputs = model(images)
        _, predicted = torch.max(outputs.data, 1)
        total += labels.size(0)
        correct += (predicted == labels).sum().item()

print('Accuracy of the network on the 10000 test images: %d %%' % (100 * correct / total))
```

### 5. 模型保存与加载
在PyTorch中，模型的保存和加载是模型持久化和复用的关键步骤。
这允许你保存训练好的模型以供将来使用，或者在不同的设备上加载模型进行推理或进一步的训练。

#### 模型保存
模型的保存可以通过`torch.save`函数完成。有几种不同的方式可以保存模型：

##### 保存整个模型（推荐）
保存整个模型包括模型的结构、权重、优化器的状态以及其他可能的状态字典。
这是最常见和最方便的方式，因为你可以在加载模型时直接使用，无需重新定义模型结构。

```python
torch.save({
            'model_state_dict': model.state_dict(),
            'optimizer_state_dict': optimizer.state_dict(),
            'epoch': epoch,
            'loss': loss
           }, PATH)
```

在这个例子中，除了模型的权重，我们还保存了优化器的状态、训练的epoch数以及损失。这对于断点续训非常有用。

##### 仅保存模型的权重
如果你只需要保存模型的权重而不关心模型结构或优化器状态，可以只保存模型的状态字典。

```python
torch.save(model.state_dict(), PATH)
```

#### 模型加载
加载模型同样使用`torch.load`函数。加载的方式应与保存的方式相对应。

##### 加载整个模型
当模型是以整个状态保存的，加载时需要重新实例化模型和优化器，并加载保存的状态。

```python
checkpoint = torch.load(PATH)
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
epoch = checkpoint['epoch']
loss = checkpoint['loss']
```

##### 加载模型权重
如果只保存了模型的权重，加载时只需关注模型结构的定义和权重的加载。

```python
model = TheModelClass(*args, **kwargs)
model.load_state_dict(torch.load(PATH))
model.eval()
```

在加载模型后，记得调用`model.eval()`切换模型到评估模式，这对于诸如BatchNorm和Dropout层是很重要的，因为在评估模式下它们的行为与训练模式不同。

##### 注意事项
- 当在不同设备上加载模型时，确保使用正确的`map_location`参数，例如：
  ```python
  device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
  model.load_state_dict(torch.load(PATH, map_location=device))
  ```
- 如果在CPU上加载GPU训练的模型，或者反之，`map_location`参数是必须的。

通过这些步骤，你可以有效地保存和恢复PyTorch模型，无论是为了部署模型还是为了训练的连续性。

### 总结
上述步骤覆盖了深度学习项目的核心流程，从数据集的预处理到模型的定义、训练、测试，直至模型的保存与加载。

到这里我们的 pytorch 入门就结束啦，之后可能会继续更新（不过可能会先写别的）。