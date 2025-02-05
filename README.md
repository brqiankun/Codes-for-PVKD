Point-to-Voxel Knowledge Distillation for LiDAR Semantic Segmentation (CVPR 2022)

Our model achieves state-of-the-art performance on three challenges, i.e., ranks **1st** in [Waymo 3D Semantic Segmentation Challenge](https://waymo.com/open/challenges/2022/3d-semantic-segmentation/) (the "Cylinder3D" and "Offboard_SemSeg" entries, May 2022), ranks **1st** in [SemanticKITTI LiDAR Semantic Segmentation Challenge](https://competitions.codalab.org/competitions/20331#results) (single-scan, the "Point-Voxel-KD" entry, Jun 2022), ranks **2nd** in [SemanticKITTI LiDAR Semantic Segmentation Challenge](https://competitions.codalab.org/competitions/20331#results) (multi-scan, the "PVKD" entry, Dec 2021). Do not hesitate to use our trained models!

## News

- **2022-11** [NEW:fire:] Some useful training tips have been provided.

- **2022-11** The distillation codes and some training tips will be released after CVPR DDL.

- **2022-7** We provide a trained model of [CENet](https://github.com/huixiancheng/CENet), a range-image-based LiDAR segmentation method. The reproduced performance is much higher than the reported value! 

- **2022-6** Our method ranks **1st** in [SemanticKITTI LiDAR Semantic Segmentation Challenge](https://competitions.codalab.org/competitions/20331#results) (single-scan, the "Point-Voxel-KD" entity)
<p align="center">
   <img src="./img/semantickitti_single_scan.PNG" width="30%"> 
</p>

- **2022-5** Our method ranks **1st** in [Waymo 3D Semantic Segmentation Challenge](https://waymo.com/open/challenges/2022/3d-semantic-segmentation/) (the "Cylinder3D" and "Offboard_SemSeg" entities)
<p align="center">
   <img src="./img/waymo.PNG" width="30%"> 
</p>

## Installation

### Requirements
- PyTorch >= 1.2 
- yaml
- tqdm
- numba
- Cython
- [torch-scatter](https://github.com/rusty1s/pytorch_scatter)
- [nuScenes-devkit](https://github.com/nutonomy/nuscenes-devkit) (optional for nuScenes)
- [spconv](https://github.com/traveller59/spconv) (tested with spconv==1.2.1 and cuda==10.2)

## Data Preparation

### SemanticKITTI
```
./
├── 
├── ...
└── path_to_data_shown_in_config/
    ├──sequences
        ├── 00/           
        │   ├── velodyne/	
        |   |	├── 000000.bin
        |   |	├── 000001.bin
        |   |	└── ...
        │   └── labels/ 
        |       ├── 000000.label
        |       ├── 000001.label
        |       └── ...
        ├── 08/ # for validation
        ├── 11/ # 11-21 for testing
        └── 21/
	    └── ...
```

### nuScenes
```
./
├── 
├── ...
└── path_to_data_shown_in_config/
		├──v1.0-trainval
		├──v1.0-test
		├──samples
		├──sweeps
		├──maps

```

### Waymo
```
./
├── 
├── ...
└── path_to_data_shown_in_config/
		├──first_return
		├──second_return

```

## Test
We take evaluation on the SemanticKITTI test set (single-scan) as example.

1. Download the [pre-trained models](https://drive.google.com/drive/folders/1LyWhVCqMzSVDe44c8ARDp8b94w1ct-tR?usp=sharing) and put them in `./model_load_dir`.

2. Generate predictions on the SemanticKITTI test set.

```
CUDA_VISIBLE_DEVICES=0 python -u test_cyl_sem_tta.py
```

We perform test-time augmentation to boost the performance. The model predictions will be saved in `./out_cyl/test` by default.


3. Convert label number back to the original dataset format before submitting:
```
python remap_semantic_labels.py -p out_cyl/test -s test --inverse
cd out_cyl/test
zip -r out_cyl.zip sequences/
```

4. Upload out_cyl.zip to the [SemanticKITTI online server](https://competitions.codalab.org/competitions/20331#participate).

## Train

```
CUDA_VISIBLE_DEVICES=0 python -u train_cyl_sem.py
```

Remember to change the `imageset` of `val_data_loader` to `val`, `return_test` of `dataset_params` to `False` in `semantickitti.yaml`. Currently, we only support vanilla training.


## Useful Training Tips
1. Finetuning.

You can finetune the model using both train and val sets as well as a smaller learning rate (1/3 or 1/4 of the original learning rate).

2. Model ensemble. 

You can use models of different epochs as an ensemble. Different models can also be taken as an ensemble, e.g., SPVCNN and Cylinder3D.

3. Semi-supervised learning. 

You can follow [GuidedContrast](https://arxiv.org/abs/2110.08188) to use pseudo labels of the test set to complement the original training set. (**DO NOT** use it in the supervised training. It can only be used in the semi-supervised setting to prove the value of the proposed semi-supervised algorithm.)

4. More data augmentations. 

You can use [LaserMix](https://arxiv.org/abs/2207.00026), [Instance Augmentation](https://github.com/edwardzhou130/Panoptic-PolarNet/blob/main/dataloader/instance_augmentation.py) and [PolarMix](https://arxiv.org/abs/2208.00223) to increase the diversity of training samples.

5. Knowledge distillation (KD).

You can refer to [CRD](https://github.com/HobbitLong/RepDistiller) to apply KD to boost the performance of LiDAR segmentation models. We will release a more efficient and effective version of the PVKD algorithm soon.

6. Using more inputs.

In addition to the (x, y, z), you can also use the intensity, range, azimuth, inclination and elongation as additional inputs. Remember to normalize these input signals if necessary. Tanh function is a good normalizer in some cases.

7. Increasing the model size.

You can either increase the width (more channels) or the depth (more layers) of the model to boost the performance.

8. Test time augmentation (TTA).

You can use more augmentations (flipping, rotation, scaling, translation) in TTA to boost the performance. A proper combination of them is vital to the final performance.

## Performance

Abbreviation:

cyl: Cylinder3D, sem: SemanticKITTI, nusc: nuScenes, ms: multi-scan task, tta: test-time augmentation,

1.5x: channel expansion ratio, 72_4: performance (mIoU), 64x512: resolution of the range image

1. SemanticKITTI test set (single-scan):

|Model|Reported|Reproduced|Gain|Weight|
|:---:|:---:|:---:|:---:|:---:|
|SPVNAS|66.4%|71.4%|**5.0%**|--|
|Cylinder3D_1.5x|--|**72.4%**|--|[cyl_sem_1.5x_72_4.pt](https://drive.google.com/drive/folders/1LyWhVCqMzSVDe44c8ARDp8b94w1ct-tR?usp=sharing)|
|Cylinder3D|68.9%|71.8%|**2.9%**|[cyl_sem_1.0x_71_8.pt](https://drive.google.com/drive/folders/1LyWhVCqMzSVDe44c8ARDp8b94w1ct-tR?usp=sharing)|
|Cylinder3D_0.5x|71.2%|71.4%|0.2%|[cyl_sem_0.5x_71_4.pt](https://drive.google.com/drive/folders/1LyWhVCqMzSVDe44c8ARDp8b94w1ct-tR?usp=sharing)|
|CENet_1.0x|64.7%|67.6%|2.9%|[CENet_64x512_67_6](https://drive.google.com/drive/folders/1LyWhVCqMzSVDe44c8ARDp8b94w1ct-tR?usp=sharing)|

2. SemanticKITTI test set (multi-scan):

|Model|Reported|Reproduced|Gain|Weight|
|:---:|:---:|:---:|:---:|:---:|
|Cylinder3D|52.5%|--|--|--|
|Cylinder3D_0.5x|58.2%|58.4%|0.2%|[cyl_sem_ms_0.5x_58_4.pt](https://drive.google.com/drive/folders/1LyWhVCqMzSVDe44c8ARDp8b94w1ct-tR?usp=sharing)|

3. Waymo test set:

|Model|Reported|Reproduced|Gain|Weight|
|:---:|:---:|:---:|:---:|:---:|
|Cylinder3D|71.18%|71.18%|--|--|
|Cylinder3D_0.5x|--|--|--|--|

4. nuScenes val set:

|Model|Reported|Reproduced|Gain|Weight|
|:---:|:---:|:---:|:---:|:---:|
|Cylinder3D|76.1%|--|--|--|
|Cylinder3D_0.5x|76.0%|76.15%|0.15%|[cyl_nusc_0.5x_76_15.pt](https://drive.google.com/drive/folders/1LyWhVCqMzSVDe44c8ARDp8b94w1ct-tR?usp=sharing)|

## Citation
If you use the codes, please consider citing the following publications:
```
@inproceedings{pvkd,
    title     = {Point-to-Voxel Knowledge Distillation for LiDAR Semantic Segmentation},
    author    = {Hou, Yuenan and Zhu, Xinge and Ma, Yuexin and Loy, Chen Change and Li, Yikang},
    booktitle = {IEEE Conference on Computer Vision and Pattern Recognition},
    pages     = {8479-8488}
    year      = {2022},
}

@inproceedings{cylinder3d,
    title={Cylindrical and Asymmetrical 3D Convolution Networks for LiDAR Segmentation},
    author={Zhu, Xinge and Zhou, Hui and Wang, Tai and Hong, Fangzhou and Ma, Yuexin and Li, Wei and Li, Hongsheng and Lin, Dahua},
    booktitle={IEEE Conference on Computer Vision and Pattern Recognition},
    pages={9939--9948},
    year={2021}
}

@article{cylinder3d-tpami,
    title={Cylindrical and Asymmetrical 3D Convolution Networks for LiDAR-based Perception},
    author={Zhu, Xinge and Zhou, Hui and Wang, Tai and Hong, Fangzhou and Li, Wei and Ma, Yuexin and Li, Hongsheng and Yang, Ruigang and Lin, Dahua},
    journal={IEEE Transactions on Pattern Analysis and Machine Intelligence},
    year={2021},
    publisher={IEEE}
}
```

## Acknowledgements
This repo is built upon the awesome [Cylinder3D](https://github.com/xinge008/Cylinder3D).

## 蒸馏(distillation)
从大模型**蒸馏**会产生较差效果，由于点云自身的稀疏性sparsity，随机性randomness和可变密度varying density.
### Point-to-Voxel-Knowledge Distillation 点云到体素知识蒸馏
从点云级别和体素级别转移权重
1. 从点级和体素级两方面输出蒸馏来补充稀疏监督信号。
2. 将点云划分为超体素，对不频繁的类和远处的物体的超体素增加采样。
3. 点间和体素间的**亲和力**蒸馏

实现2倍加速Cylinder3D模型， 75% MACs(Multiply-Accumulate-Operations, 乘加累积操作数)减少

- FLOPS(Floating Point Operations Per Second) 每秒浮点运算次数
- FLOPs(Floating Point Operations) 浮点运算次数，衡量模型的计算复杂度
- MACs(Multiply-Accumulate Operations)乘加累积操作数，包含一个乘法一个加法，大约为2FLOPs

Point-to-Voxel Knowledge Distillation(PVD)
1. pointwise output 包含 fine-grained perceptual information
2. voxelwise prediction embraces coarse but richer clues about the surrounding environment

### 点级和体素级的亲和力知识(通过点特征和体素特征的成对语义相似性获得)
**亲和力矩阵**

划分固定数量的超体素(supervoxel), 只对K个超体素进行采样和蒸馏
不同的类别，不同的感知距离，提出了difficulty-aware sampling strategy来对少数类和远距离物体增加采样。

对Cylinder3D进行蒸馏，模型压缩，性能得到提升。

#### Knowledge distillation(KD)

传统蒸馏方案在2D分割上表现良好
PVD是首个在Lidar分割上进行蒸馏的方案，可以用于各种模型。

给定点云X:[N, 3]  当前方案使用CNN进行端到端预测
现有传统的针对2D图像分割的蒸馏方案比较成熟。

### Framework overview of Cylinder3D
1. 对输入的点云使用堆叠的MLP生成点云特征[N, Cf]   Cf是点云特征的维度
2. 对点云特征[N, Cf]按照Cylinder进行重新划分
3. 属于同一体素的特征通过maxpooling操作被聚合在一起，获得体素特征[M, Cf] M是非空的体素数量
4. 将体素特征送入asymmetrical 3D convolution networks(非对称3D卷积网络)，生成体素级输出[R, A, H, C]
5. 逐点细化模块进一步生成点级的预测[N, C] <br>
N, C, R, A, H分别是点云中点的数量，点的分类类别，半径，角度，高度
6. 之后使用argmax，得到每个点的分类结果


