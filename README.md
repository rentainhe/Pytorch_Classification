# Pytorch Classification
Classification on `CIFAR10` `CIFAR100` and `ImageNet` using Pytorch

## Features
* Two training ways
  * epoch training: training network using __fixed epoches__ _( warmup_epoch/epoch= 1/200 )_
  * step training: training network using __fixed steps__ _( warmup_steps/total_steps= 1000/80000 )_
* Multi-GPU support
* Easy and Useful Training log file
* Support Different Training Schedule ( `multi-step` / `cosine` / `linear` )

## Requirements
* python3.6
* pytorch1.6.0 + cuda10.1
* tensorboard 2.3.0
* ptflops

## Installation
* clone
  ```
  git clone https://github.com/x-multimodal/x-classification.git
  ```

## Usage

### 1. enter directory
```bash
$ cd x-classification
```

### 2. dataset
* Only support cifar10 and cifar100 now (Will support Imagenet Later)
* Using cifar10 and cifar100 dataset from torchvision since it's more convinient
* for `cifar10` and `cifar100` we introduce you to follow this transform method:
```python
from torchvision import transforms
transform_train = transforms.Compose([
        transforms.RandomCrop(32, padding=4),
        transforms.ToTensor(),
        transforms.Normalize(mean=...,std=...)
    ])
```

### 3. run tensorboard
Install tensorboard
```bash
$ pip install tensorboard
Run tensorboard
$ tensorboard --logdir runs --port 6006 --host localhost
```

### 4. training
Here is an example which trains a `resnet18` on the `cifar100` dataset using `epoch training loop` 
```bash
$ python3 epoch_run.py --dataset cifar100 --model resnet18 --run train
```

Here is an example of mixed-precision training (only support `epoch training loop` now):
```bash
$ python3 epoch_run.py --dataset cifar100 --model resnet18 --run train --mix
```

Here is an example which trains a `resnet18` on the `cifar100` dataset using `step training loop`
```bash
$ python3 step_run.py --dataset cifar100 --model resnet18 --run train
```

- ```--run={'train','test','visual'}``` to set the mode to be executed

- ```--model=str```, e.g., to set the model for training

- ```--dataset={'cifar10','cifar100','imagenet'}``` to set the dataset for training

Addition:

- ```--version=str```, e.g, ```--version='test'``` to set a name for this training

- ```--gpu=str```, e.g, ```--gpu='2'``` to train the model on specified GPU device, set ```--gpu='1,2,3'``` to use Multi-GPU Training 

- ```--seed=int```, e.g, ```--seed=1020``` to use a fixed seed to initialize the model. Unset it results in random seeds.

- ```--label_smoothing```, e.g, ```--label_smoothing``` to use label smoothing for training

- ```--gradient_accumulation_steps=int```, e.g, ```--gradient_accumulation_steps=32``` to set the gradient steps to reduce the memory use

- ```--no_bias_decay```, e.g, ```--no_bias_decay``` to use `no bias decay` for training, only specified models work

Specified Addition For `Epoch` Training:

- ```--warmup_epoch=int```, e.g, ```--warmup_epoch=1``` to set the warmup epoches

- ```--epoch=int```, e.g, ```--epoch=200``` to set the total training epoch nums

- ```--mix=False```, e.g, ```--mix``` to use mixed-precision training for accelerating, make sure your Pytorch version >= 1.6

Specified Addition For `Step` Training:

- ```--warmup_steps=int```, e.g, ```--warmup_steps=1000``` to set the warmup steps

- ```--num_steps=int```, e.g, ```--num_steps=80000``` to set the total steps

- ```--decay_type=str```, e.g, ```--decay_type='cosine'``` to use the `cosine` learning rate decay schedule

The supported net args are:
```
resnet18
resnet34
resnet50
mobilenet
mobilenetv2
mobilenetv3
shufflenet
seresnet18
seresnet34
seresnet50
seresnet101
seresnet152
resnext50
resnext101
resnext152
densenet121
densenet169
densenet201
densenet161
```

### 5. testing
In this repo, we offer two kinds of testing method: `Epoch Testing` and `Step Testing` which corresponds to `epoch training` and `step training` 

Here is an example which tests trained `resnet18` model on `cifar100` using `Epoch Testing`
```bash
$ python3 epoch_run.py --dataset cifar100 --model resnet18 --run test --ckpt_e={epoch} --ckpt_v={version} --ckpt_type best 
```

Details of `Epoch Testing` method:

- ```--run='test'```, run mode must be `test`

- ```--ckpt_e=int```, the ckpt_e must be specified which corresponds to the epoch of the loaded moel

- ```--ckpt_v=str```, the ckpt_v must be specified which corresponds to the version of the loaded model

- ```--ckpt_type={'best', 'regular'}```, the ckpt_type should correspond to the type of the loaded model

### 6. Network information
There are two ways for you to know the information of your network:
```bash
$ python counter.py --model resnet18 --gpu 1
```

- `ptflops` need GPUs, you can use `--gpu=1` to set the test on specific GPU

```bash
$ python net_stat.py --model resnet18 --gpu 1
```
make sure you have already installed `ptflops`, `torchstat`

## Implementated NetWork

- resnet [Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385v1)
- mobilenet [MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications](https://arxiv.org/abs/1704.04861)
- mobilenetv3 [Searching for MobileNetV3](https://arxiv.org/pdf/1905.02244.pdf)
- shufflenet [ShuffleNet: An Extremely Efficient Convolutional Neural Network for Mobile Devices](https://arxiv.org/abs/1707.01083v2)
- senet [Squeeze-and-Excitation Networks](https://arxiv.org/abs/1709.01507)
- densenet [Densely Connected Convolutional Networks](https://arxiv.org/pdf/1608.06993.pdf)

## Training Details
- `epoch=200`
- `lr = 0.1` divide by `5` at `60th`, `120th`, `160th` epochs
- `batchsize 128`
- `weight decay=5e-4`, 
- `Nesterov momentum=0.9` 

more common use:

- initial `lr = 0.1`, lr divied by `10` at `150th` and `225th` epochs, and training for `300 epochs` with `batchsize 128`

You could decrese the batchsize to 64 or whatever suits you or change the training details in `./conf/global_configs.py`

## Training Related Work
- [Improved Regularization of Convolutional Neural Networks with Cutout](https://arxiv.org/abs/1708.04552v2)
- [Regularizing Neural Networks by Penalizing Confident Output Distributions](https://arxiv.org/abs/1701.06548v1)
- [Random Erasing Data Augmentation](https://arxiv.org/abs/1708.04896v2)

## Results
### Epoch Training Results
#### Resnet
|dataset|network|params|warmup-epochs|total-epochs|label-smoothing|Top-1 accuracy
|:---:|:---:|:---:|:---:|:---:|:---:|:---:
|cifar100|resnet18|11.2M|1|200|0.1|76.22%
|cifar100|resnet34|21.3M|1|200|0.1|
|cifar100|resnet50|23.7M|1|200|0.1|
|cifar100|resnext50|14.7M|1|200|0.1|
|cifar100|seresnet18|11.3M|1|200|0.1|
|cifar100|shufflenet|1.01M|1|200|0.1|
|cifar100|mobilenet|3.3M|1|200|0.1|


### Step Training Results
|dataset|network|params|warmup-steps|total-steps|label-smoothing|accuracy
|:---:|:---:|:---:|:---:|:---:|:---:|:---:
