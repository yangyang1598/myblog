----
title: MLP 모델 설명 및 MLP방식 데이터 시각화
----

## MLP 모델이란?
- 퍼셉트론이 지니고 있는 한계(비선형 분류 문제 해결)를 극복하기 위해 여러 Layer를 쌓아올린 MLP(Multi Layer Perceptron)가 등장함. 
- 간단히 비교하면, 
  + 퍼셉트론은 Input Layer와 Output Layer만 존재하는 형태입니다. 
  + MLP는 Input과 Output 사이에 Hidden Layer를 추가합니다. 즉, MLP는 이러한 Hidden Layer를 여러겹으로 쌓게 되는데, 이를 MLP 모델이라고 부릅니다. 

## MLP 모델을 활용한 MNIST 모델 개발 
- MLP 모델 순서대로 코드를 구현합니다. 

### Step 1. 모듈 불러오기


```python
import numpy as np
import matplotlib.pyplot as plt
import torch
import torch.nn as nn
import torch.nn.functional as F
from torchvision import transforms, datasets
```

### Step 2. 딥러닝 모델 설계 시 필요한 장비 세팅 


```python
if torch.cuda.is_available():
  DEVICE = torch.device('cuda')
else:
  DEVICE = torch.device('cpu')

print("PyTorch Version:", torch.__version__, ' Device:', DEVICE)
```

    PyTorch Version: 1.9.0+cu102  Device: cuda
    


```python
BATCH_SIZE = 32 # 데이터가 32개로 구성되어 있음. 
EPOCHS = 10 # 전체 데이터 셋을 10번 반복해 학습함.
```

### Step 3. 데이터 다운로드
- torchvision 내 datasets 함수 이용하여 데이터셋 다운로드 합니다.
- ToTensor() 활용하여 데이터셋을 tensor 형태로 변환
- 한 픽셀은 0\~255 범위의 스칼라 값으로 구성, 이를 0\~1 범위에서 정규화 과정 진행
- DataLoader는 일종의 Batch Size 만큼 묶음으로 묶어준다는 의미
  + Batch_size는 Mini-batch 1개 단위를 구성하는 데이터의 개수
  


```python
train_dataset = datasets.MNIST(root = "../data/MNIST", 
                               train = True, 
                               download = True, 
                               transform = transforms.ToTensor())

test_dataset = datasets.MNIST(root = "../data/MNIST", 
                              train = False, 
                              transform = transforms.ToTensor())

train_loader = torch.utils.data.DataLoader(dataset = train_dataset, 
                                           batch_size = BATCH_SIZE, 
                                           shuffle=True)

test_loader = torch.utils.data.DataLoader(dataset = test_dataset, 
                                           batch_size = BATCH_SIZE, 
                                           shuffle=False)
```

    Downloading http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz
    Downloading http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz to ../data/MNIST/MNIST/raw/train-images-idx3-ubyte.gz
    


    HBox(children=(FloatProgress(value=0.0, max=9912422.0), HTML(value='')))


    
    Extracting ../data/MNIST/MNIST/raw/train-images-idx3-ubyte.gz to ../data/MNIST/MNIST/raw
    
    Downloading http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz
    Failed to download (trying next):
    HTTP Error 503: Service Unavailable
    
    Downloading https://ossci-datasets.s3.amazonaws.com/mnist/train-labels-idx1-ubyte.gz
    Downloading https://ossci-datasets.s3.amazonaws.com/mnist/train-labels-idx1-ubyte.gz to ../data/MNIST/MNIST/raw/train-labels-idx1-ubyte.gz
    


    HBox(children=(FloatProgress(value=0.0, max=28881.0), HTML(value='')))


    
    Extracting ../data/MNIST/MNIST/raw/train-labels-idx1-ubyte.gz to ../data/MNIST/MNIST/raw
    
    Downloading http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz
    Failed to download (trying next):
    HTTP Error 503: Service Unavailable
    
    Downloading https://ossci-datasets.s3.amazonaws.com/mnist/t10k-images-idx3-ubyte.gz
    Downloading https://ossci-datasets.s3.amazonaws.com/mnist/t10k-images-idx3-ubyte.gz to ../data/MNIST/MNIST/raw/t10k-images-idx3-ubyte.gz
    


    HBox(children=(FloatProgress(value=0.0, max=1648877.0), HTML(value='')))


    
    Extracting ../data/MNIST/MNIST/raw/t10k-images-idx3-ubyte.gz to ../data/MNIST/MNIST/raw
    
    Downloading http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz
    Failed to download (trying next):
    HTTP Error 503: Service Unavailable
    
    Downloading https://ossci-datasets.s3.amazonaws.com/mnist/t10k-labels-idx1-ubyte.gz
    Downloading https://ossci-datasets.s3.amazonaws.com/mnist/t10k-labels-idx1-ubyte.gz to ../data/MNIST/MNIST/raw/t10k-labels-idx1-ubyte.gz
    


    HBox(children=(FloatProgress(value=0.0, max=4542.0), HTML(value='')))


    
    Extracting ../data/MNIST/MNIST/raw/t10k-labels-idx1-ubyte.gz to ../data/MNIST/MNIST/raw
    
    

    /usr/local/lib/python3.7/dist-packages/torchvision/datasets/mnist.py:498: UserWarning: The given NumPy array is not writeable, and PyTorch does not support non-writeable tensors. This means you can write to the underlying (supposedly non-writeable) NumPy array using the tensor. You may want to copy the array to protect its data or make it writeable before converting it to a tensor. This type of warning will be suppressed for the rest of this program. (Triggered internally at  /pytorch/torch/csrc/utils/tensor_numpy.cpp:180.)
      return torch.from_numpy(parsed.astype(m[2], copy=False)).view(*s)
    

### step 4. 데이터 확인 및 시각화
- 데이터를 확인하고 시각화를 진행합니다. 
- 32개의 이미지 데이터에 label 값이 각 1개씩 존재하기 때문에 32개의 값을 갖고 있음


```python
for (X_train, y_train) in train_loader:
  print('X_train:', X_train.size(), 'type:', X_train.type())
  print('y_train:', y_train.size(), 'type:', y_train.type())
  break
```

    X_train: torch.Size([32, 1, 28, 28]) type: torch.FloatTensor
    y_train: torch.Size([32]) type: torch.LongTensor
    


```python
pltsize = 1
plt.figure(figsize=(10 * pltsize, pltsize))
for i in range(10):
  plt.subplot(1, 10, i + 1)
  plt.axis('off')
  plt.imshow(X_train[i, :, :, :].numpy().reshape(28, 28), cmap="gray_r")
  plt.title('Class: ' + str(y_train[i].item()))
```


![](output_11_0.png)

    


### step 5. MLP 모델 설계
- torch 모듈을 이용해 MLP를 설계합니다. 


```python
class Net(nn.Module):
  '''
  Forward Propagation 정의
  '''
  def __init__(self):
    super(Net, self).__init__()
    self.fc1 = nn.Linear(28 * 28 * 1, 512) # (가로 픽셀 * 세로 픽셀 * 채널 수) 크기의 노드 수 설정 Fully Connected Layer 노드 수 512개 설정
    self.fc2 = nn.Linear(512, 256) # Input으로 사용할 노드 수는 512으로, Output 노드수는 256개로 지정
    self.fc3 = nn.Linear(256, 10) # Input 노드수는 256, Output 노드수는 10개로 지정

  def forward(self, x):
    x = x.view(-1, 28 * 28) # 1차원으로 펼친 이미지 데이터 통과
    x = self.fc1(x)
    x = F.sigmoid(x)
    x = self.fc2(x)
    x = F.sigmoid(x)
    x = self.fc3(x)
    x = F.log_softmax(x, dim = 1)
    return x
```

### step 6. 옵티마이저 목적 함수 설정
- Back Propagation 설정 위한 목적 함수 설정 


```python
model = Net().to(DEVICE)
optimizer = torch.optim.SGD(model.parameters(), lr = 0.01, momentum=0.5)
criterion = nn.CrossEntropyLoss() # output 값과 원-핫 인코딩 값과의 Loss 
print(model)
```

    Net(
      (fc1): Linear(in_features=784, out_features=512, bias=True)
      (fc2): Linear(in_features=512, out_features=256, bias=True)
      (fc3): Linear(in_features=256, out_features=10, bias=True)
    )
    

### step 7. MLP 모델 학습
- MLP 모델을 학습 상태로 지정하는 코드를 구현 



```python
def train(model, train_loader, optimizer, log_interval):
  model.train()
  for batch_idx, (image, label) in enumerate(train_loader): # 모형 학습
    image = image.to(DEVICE)
    label = label.to(DEVICE)
    optimizer.zero_grad() # Optimizer의 Gradient 초기화
    output = model(image)
    loss = criterion(output, label)
    loss.backward() # back propagation 계산
    optimizer.step()

    if batch_idx % log_interval == 0:
      print("Train Epoch: {} [{}/{}({:.0f}%)]\tTrain Loass: {:.6f}".format(Epoch, batch_idx * len(image), 
                                                                           len(train_loader.dataset), 100. * batch_idx / len(train_loader), loss.item()))
```

### step 8. 검증 데이터 확인 함수



```python
def evaluate(model, test_loader):
  model.eval()
  test_loss = 0
  correct = 0

  with torch.no_grad():
    for image, label in test_loader:
      image = image.to(DEVICE)
      label = label.to(DEVICE)
      output = model(image)
      test_loss += criterion(output, label).item()
      prediction = output.max(1, keepdim = True)[1]
      correct += prediction.eq(label.view_as(prediction)).sum().item()

  test_loss /= len(test_loader.dataset)
  test_accuracy = 100. * correct / len(test_loader.dataset)
  return test_loss, test_accuracy
```

- 모델 평가 시, Gradient를 통해 파라미터 값이 업데이트되는 현상 방지 위해 torch.no_grad() Gradient의 흐름 제어

### step 9. MLP 학습 실행


```python
for Epoch in range(1, EPOCHS + 1):
  train(model, train_loader, optimizer, log_interval=200)
  test_loss, test_accuracy = evaluate(model, test_loader)
  print("\n[EPOCH: {}], \tTest Loss: {:.4f}, \tTest Accuracy: {:.2f} %\n".format(Epoch, test_loss, test_accuracy))
```

    /usr/local/lib/python3.7/dist-packages/torch/nn/functional.py:1805: UserWarning: nn.functional.sigmoid is deprecated. Use torch.sigmoid instead.
      warnings.warn("nn.functional.sigmoid is deprecated. Use torch.sigmoid instead.")
    

    Train Epoch: 1 [0/60000(0%)]	Train Loass: 2.343919
    Train Epoch: 1 [6400/60000(11%)]	Train Loass: 2.304868
    Train Epoch: 1 [12800/60000(21%)]	Train Loass: 2.276658
    Train Epoch: 1 [19200/60000(32%)]	Train Loass: 2.320047
    Train Epoch: 1 [25600/60000(43%)]	Train Loass: 2.309569
    Train Epoch: 1 [32000/60000(53%)]	Train Loass: 2.293312
    Train Epoch: 1 [38400/60000(64%)]	Train Loass: 2.264061
    Train Epoch: 1 [44800/60000(75%)]	Train Loass: 2.311200
    Train Epoch: 1 [51200/60000(85%)]	Train Loass: 2.261489
    Train Epoch: 1 [57600/60000(96%)]	Train Loass: 2.260837
    
    [EPOCH: 1], 	Test Loss: 0.0698, 	Test Accuracy: 36.43 %
    
    Train Epoch: 2 [0/60000(0%)]	Train Loass: 2.233048
    Train Epoch: 2 [6400/60000(11%)]	Train Loass: 2.196863
    Train Epoch: 2 [12800/60000(21%)]	Train Loass: 2.173305
    Train Epoch: 2 [19200/60000(32%)]	Train Loass: 2.120425
    Train Epoch: 2 [25600/60000(43%)]	Train Loass: 2.023674
    Train Epoch: 2 [32000/60000(53%)]	Train Loass: 1.936951
    Train Epoch: 2 [38400/60000(64%)]	Train Loass: 1.738194
    Train Epoch: 2 [44800/60000(75%)]	Train Loass: 1.493235
    Train Epoch: 2 [51200/60000(85%)]	Train Loass: 1.547187
    Train Epoch: 2 [57600/60000(96%)]	Train Loass: 1.400939
    
    [EPOCH: 2], 	Test Loss: 0.0402, 	Test Accuracy: 60.40 %
    
    Train Epoch: 3 [0/60000(0%)]	Train Loass: 1.429194
    Train Epoch: 3 [6400/60000(11%)]	Train Loass: 1.178954
    Train Epoch: 3 [12800/60000(21%)]	Train Loass: 0.970049
    Train Epoch: 3 [19200/60000(32%)]	Train Loass: 1.105888
    Train Epoch: 3 [25600/60000(43%)]	Train Loass: 0.926736
    Train Epoch: 3 [32000/60000(53%)]	Train Loass: 0.794726
    Train Epoch: 3 [38400/60000(64%)]	Train Loass: 0.828843
    Train Epoch: 3 [44800/60000(75%)]	Train Loass: 0.872613
    Train Epoch: 3 [51200/60000(85%)]	Train Loass: 0.713790
    Train Epoch: 3 [57600/60000(96%)]	Train Loass: 0.464628
    
    [EPOCH: 3], 	Test Loss: 0.0243, 	Test Accuracy: 76.67 %
    
    Train Epoch: 4 [0/60000(0%)]	Train Loass: 0.507696
    Train Epoch: 4 [6400/60000(11%)]	Train Loass: 0.635314
    Train Epoch: 4 [12800/60000(21%)]	Train Loass: 0.636553
    Train Epoch: 4 [19200/60000(32%)]	Train Loass: 0.875113
    Train Epoch: 4 [25600/60000(43%)]	Train Loass: 0.558601
    Train Epoch: 4 [32000/60000(53%)]	Train Loass: 0.462972
    Train Epoch: 4 [38400/60000(64%)]	Train Loass: 0.630431
    Train Epoch: 4 [44800/60000(75%)]	Train Loass: 0.428842
    Train Epoch: 4 [51200/60000(85%)]	Train Loass: 0.826000
    Train Epoch: 4 [57600/60000(96%)]	Train Loass: 0.508183
    
    [EPOCH: 4], 	Test Loss: 0.0182, 	Test Accuracy: 83.32 %
    
    Train Epoch: 5 [0/60000(0%)]	Train Loass: 0.576632
    Train Epoch: 5 [6400/60000(11%)]	Train Loass: 0.437238
    Train Epoch: 5 [12800/60000(21%)]	Train Loass: 0.942570
    Train Epoch: 5 [19200/60000(32%)]	Train Loass: 0.412996
    Train Epoch: 5 [25600/60000(43%)]	Train Loass: 0.362784
    Train Epoch: 5 [32000/60000(53%)]	Train Loass: 0.677201
    Train Epoch: 5 [38400/60000(64%)]	Train Loass: 0.706629
    Train Epoch: 5 [44800/60000(75%)]	Train Loass: 0.587188
    Train Epoch: 5 [51200/60000(85%)]	Train Loass: 0.584579
    Train Epoch: 5 [57600/60000(96%)]	Train Loass: 0.602702
    
    [EPOCH: 5], 	Test Loss: 0.0147, 	Test Accuracy: 86.63 %
    
    Train Epoch: 6 [0/60000(0%)]	Train Loass: 0.388631
    Train Epoch: 6 [6400/60000(11%)]	Train Loass: 0.571896
    Train Epoch: 6 [12800/60000(21%)]	Train Loass: 0.287511
    Train Epoch: 6 [19200/60000(32%)]	Train Loass: 0.394190
    Train Epoch: 6 [25600/60000(43%)]	Train Loass: 0.264939
    Train Epoch: 6 [32000/60000(53%)]	Train Loass: 0.495090
    Train Epoch: 6 [38400/60000(64%)]	Train Loass: 0.274051
    Train Epoch: 6 [44800/60000(75%)]	Train Loass: 0.299830
    Train Epoch: 6 [51200/60000(85%)]	Train Loass: 0.450571
    Train Epoch: 6 [57600/60000(96%)]	Train Loass: 0.257513
    
    [EPOCH: 6], 	Test Loss: 0.0129, 	Test Accuracy: 87.99 %
    
    Train Epoch: 7 [0/60000(0%)]	Train Loass: 0.377794
    Train Epoch: 7 [6400/60000(11%)]	Train Loass: 0.630070
    Train Epoch: 7 [12800/60000(21%)]	Train Loass: 0.320578
    Train Epoch: 7 [19200/60000(32%)]	Train Loass: 0.658281
    Train Epoch: 7 [25600/60000(43%)]	Train Loass: 0.643368
    Train Epoch: 7 [32000/60000(53%)]	Train Loass: 0.420115
    Train Epoch: 7 [38400/60000(64%)]	Train Loass: 0.687097
    Train Epoch: 7 [44800/60000(75%)]	Train Loass: 0.430246
    Train Epoch: 7 [51200/60000(85%)]	Train Loass: 0.343727
    Train Epoch: 7 [57600/60000(96%)]	Train Loass: 0.577327
    
    [EPOCH: 7], 	Test Loss: 0.0120, 	Test Accuracy: 88.78 %
    
    Train Epoch: 8 [0/60000(0%)]	Train Loass: 0.530810
    Train Epoch: 8 [6400/60000(11%)]	Train Loass: 0.278406
    Train Epoch: 8 [12800/60000(21%)]	Train Loass: 0.140812
    Train Epoch: 8 [19200/60000(32%)]	Train Loass: 0.471123
    Train Epoch: 8 [25600/60000(43%)]	Train Loass: 0.415495
    Train Epoch: 8 [32000/60000(53%)]	Train Loass: 0.534980
    Train Epoch: 8 [38400/60000(64%)]	Train Loass: 0.318894
    Train Epoch: 8 [44800/60000(75%)]	Train Loass: 0.444783
    Train Epoch: 8 [51200/60000(85%)]	Train Loass: 0.211995
    Train Epoch: 8 [57600/60000(96%)]	Train Loass: 0.358874
    
    [EPOCH: 8], 	Test Loss: 0.0113, 	Test Accuracy: 89.73 %
    
    Train Epoch: 9 [0/60000(0%)]	Train Loass: 0.423838
    Train Epoch: 9 [6400/60000(11%)]	Train Loass: 0.225967
    Train Epoch: 9 [12800/60000(21%)]	Train Loass: 0.348597
    Train Epoch: 9 [19200/60000(32%)]	Train Loass: 0.397753
    Train Epoch: 9 [25600/60000(43%)]	Train Loass: 0.199611
    Train Epoch: 9 [32000/60000(53%)]	Train Loass: 0.269096
    Train Epoch: 9 [38400/60000(64%)]	Train Loass: 0.311267
    Train Epoch: 9 [44800/60000(75%)]	Train Loass: 0.575633
    Train Epoch: 9 [51200/60000(85%)]	Train Loass: 0.246814
    Train Epoch: 9 [57600/60000(96%)]	Train Loass: 0.272062
    
    [EPOCH: 9], 	Test Loss: 0.0108, 	Test Accuracy: 89.94 %
    
    Train Epoch: 10 [0/60000(0%)]	Train Loass: 0.399807
    Train Epoch: 10 [6400/60000(11%)]	Train Loass: 0.282056
    Train Epoch: 10 [12800/60000(21%)]	Train Loass: 0.270553
    Train Epoch: 10 [19200/60000(32%)]	Train Loass: 0.605585
    Train Epoch: 10 [25600/60000(43%)]	Train Loass: 0.333442
    Train Epoch: 10 [32000/60000(53%)]	Train Loass: 0.395121
    Train Epoch: 10 [38400/60000(64%)]	Train Loass: 0.676208
    Train Epoch: 10 [44800/60000(75%)]	Train Loass: 0.170694
    Train Epoch: 10 [51200/60000(85%)]	Train Loass: 0.160667
    Train Epoch: 10 [57600/60000(96%)]	Train Loass: 0.381771
    
    [EPOCH: 10], 	Test Loss: 0.0105, 	Test Accuracy: 90.28 %
    
    

- train 함수 실행하면, model은 기존에 정의한 MLP 모델, train_loader는 학습 데이터, optimizer는 SGD, log_interval은 학습이 진행되면서 mini-batch index를 이용해 과정을 모니터링할 수 있도록 출력함.
- 학습 완료 시, Test Accuracy는 90% 수준의 정확도를 나타냄. 

- 
