# Person-reid-GAN-pytorch
A Pytorch Implementation of ["Unlabeled Samples Generated by GAN Improve the Person Re-identification Baseline in vitro"(ICCV17)](http://openaccess.thecvf.com/content_ICCV_2017/papers/Zheng_Unlabeled_Samples_Generated_ICCV_2017_paper.pdf), the official code is available [here](https://github.com/layumi/Person-reID_GAN)(in matlab).


We arrived **Rank@1=93.55%, mAP=90.67%** only with a very easy model.


**Random Erasing** is added to help train as a data augmentation method. the details of Random Erasing is available [here](https://github.com/zhunzhong07/Random-Erasing)

**re-rank strategy** is used to deal with the initial result, the details of re-rank method is available [here](https://github.com/zhunzhong07/person-re-ranking)


## Model Structure（we simply alter the model from ResNet and DenseNet）
You may learn more from `model.py`. 
We add one linear layer(bottleneck), one batchnorm layer and relu.

## Prerequisites

- Python 2.7
- GPU 
- Numpy
- Pytorch
- Torchvision

## Getting started
### Installation
- Install Pytorch(the version is 0.2.0_3) from http://pytorch.org/
- Install Torchvision from the source
```
git clone https://github.com/pytorch/vision
cd vision
python setup.py install
```
## Dataset & Preparation
Download [Market1501 Dataset](http://www.liangzheng.org/Project/project_reid.html)

Preparation: Put the images with the same id in one folder. You may use 
```bash
python prepare.py
```
Remember to change the dataset path to your own path.

Preparation: change the name of the folder to 0 to n-1(where n is the number of class, i.e. the number of person),  the folder name is the label of each person (all the pictures under the same folder are the same person) 
```bash
python changeIndex.py
```
the usage of the DCGAN model is under the DCGAN folder,  first use market1501 to train the dcgan model,  then generated some pictures using the trained DCGAN model,  then you can use different numbers of generated images to help train the model using LSRO. 

**the generated images are in the gen_0000 folder, you can copy this folder under your training set, for more details, you can refer to DCGAN-TENSORFLOW folder**

Our baseline code is only finetuned from resNet or DenseNet,  
we use pretained DenseNet as baseline to train our model, the archieved result are as follows:

|Rank@1 | mAP | Note|
| ----- | ---- | ---- |
|0.921  |   0.793| ----|
|0.934   |  0.907|re-rank|

using LSRO loss and added some image generated by DCGAN model, the achieved result are as follow:

 |**Batchsize|Multi/Single GPU training |Rank@1 | mAP | Note**|
 | ----- | ---- | ----- | ---- | ---- |
 | 32 | Single | 0.9162 | 0.7887 | add 0 generated image|
 | 32 | Single | 0.9355 | 0.9067 | after re-rank|
 | 32 | Multi | 0.8367 | 0.6442 | add 0 generated image|
 | 32 | Multi | 0.8655 | 0.8143 | after re-rank|
 | 64 | Multi | 0.843 | 0.646 | add 0 generated image|
 | 64 | Multi | 0.872 | 0.815 | after re-rank|
 | 32 | Single | 0.919 | 0.798 | add 6000 generated image|
 | 32 | Single | 0.932 | 0.9012 | after re-rank|
 | 64 | Multi | 0.909 | 0.779 | add 12000 generated image|
 | 64 | Multi | 0.931 | 0.896 | after re-rank|
 | 32 | Single | 0.925 | 0.801 | add 12000 generated image|
 | 32 | Single | 0.939 | 0.904 | after re-rank|
 | 64 | Multi | 0.915 | 0.790 | add 18000 generated image|
 | 64 | Multi | 0.933 | 0.899 | after re-rank|
 | 64 | Multi | 0.909 | 0.773 | add 24000 generated image|
 | 64 | Multi | 0.924 | 0.887 | after re-rank|
 | 32 | Single | 0.918 | 0.790 | add 24000 generated image|
 | 32 | Single | 0.932 | 0.899 | after re-rank|

To save trained model, we make a dir.
```bash
mkdir model 
```


## Train
Train the baseline by
```bash
python train_baseline.py --use_dense
```

`--name` the name of model.(ResNet or DesNet)

`--data_dir` the path of the training data.

`--batchsize` batch size.

`--erasing_p` random erasing probability.


## Test
Use trained model to extract feature by
```bash
python test.py   --which_epoch 99  --use_dense
```
`--gpu_ids` which gpu to run.

`--name` the dir name of trained model.

`--which_epoch` select the i-th model.

`--data_dir` the path of the testing data.

`--batchsize` batch size.


## Evaluation
```bash
python evaluate.py
```
It will output Rank@1, Rank@5, Rank@10 and mAP results.

For mAP calculation, you also can refer to the [C++ code for Oxford Building](http://www.robots.ox.ac.uk/~vgg/data/oxbuildings/compute_ap.cpp). We use the triangle mAP calculation (consistent with the Market1501 original code).

### re-ranking
```bash
python evaluate_rerank.py
```
**It may take more than 10G Memory to run.** So run it on a powerful machine if possible. 

It will output Rank@1, Rank@5, Rank@10 and mAP results.

### Conclusion

**when the baseline result is not so high,  the generated images can help model training(see multi-gpu training, add GAN images VS not add) , thus can improve the performance(more robust) ,  while when the baseline result is high(rank-1, 0.934), its difficult to improve the result.  when batchsize is set to 32, the result is the best,  and single-gpu training achieves better result than multi-gpu training.**

## Thanks

* Many, many thanks to [layumi](https://github.com/layumi/Person-reID_GAN) for his Great work!
  
### ~~p.s. If you have any questions, you can open an issue!~~ This Repo will no longer be supported for personal reasons!
