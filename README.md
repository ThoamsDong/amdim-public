# Learning Representations by Maximizing Mutual Information Across Views

## Introduction
**AMDIM** (Augmented Multiscale Deep InfoMax) is an approach to self-supervised representation learning based on maximizing mutual information between features extracted from multiple *views* of a shared *context*. 

Our paper describing AMDIM is available at: https://arxiv.org/abs/1906.00910.

## Note
We are in the process of cleaning-up and porting over code from our working repo, which is still in flux, so performance of models from this repo will be variable for a little while. We will add checkpoints for some strong models trained on ImageNet via self-supervision. Also, contact me (Phil) for help with adding the 3rd-party data augmentation -- it's pretty quick to get working.

### Main Results 
Results of AMDIM compared to other methods when evaluating accuracy using a linear classifier trained on top of representations provided by the self-supervised encoder:

Method                  | ImageNet        | Places205
------------------------| :-------------: | :----------------:
Rotation [1]            | 55.4            | 48.0
Exemplar [1]            | 46.0            | 42.7
Patch Offset [1]        | 51.4            | 45.3 
Jigsaw [1]              | 44.6            | 42.2
CPC - big [1]           | 48.7            | n/a
CPC - huge [2]          | 61.0            | n/a
**AMDIM**               | **68.1**        | **55.0**

> [1]: Results from [Kolesnikov et al. [2019]](https://arxiv.org/abs/1906.00910).<br/>
> [2]: Results from [Henaff et al. [2019]](https://arxiv.org/abs/1905.09272).<br/>

## Self-Supervised Training


You should be able to get some good results on ImageNet if you have access to 4 Tesla V100 GPUs with: 
```
CUDA_VISIBLE_DEVICES=0,1,2,3 python train.py \
  --ndf 192 \
  --n_rkhs 1536 \
  --batch_size 480 \
  --tclip 20.0 \
  --res_depth 8 \
  --dataset IN128 \
  --amp
```

For GPUs older than Volta, you will need to tweak the model size to fit in the available memory. The command above will take about 15GB of memory on device 0, and slightly less on devices 1-3, when running in mixed precision (`--amp`). When running in FP32, memory usage will be significantly higher. 

Results with the data augmentation implemented in this repo will be less than our strongest results
with equivalent architecture by 1-2%. Our strongest results use augmentation based on the ImageNet policy from the [Fast AutoAugment](https://arxiv.org/abs/1905.00397) paper by Lim et al., implemented in the repo available at: [https://github.com/kakaobrain/fast-autoaugment](https://github.com/kakaobrain/fast-autoaugment).

Using the stronger augmentation and an appropriate learning schedule, the command above should produce a bit over 63% accuracy on ImageNet using the online evaluation classifiers. With the standard torchvision augmentations the result will drop to a bit over 62%, which is still decent (significantly state-of-the-art, makes good coffee, etc.).

## Evaluation Classifiers

Example of retraining final evaluation classifiers on Places205, using an encoder checkpointed after training via self-supervised learning on ImageNet:

```
CUDA_VISIBLE_DEVICES=0,1,2,3 python train.py \
  --finetune # Add this flag to start the fine-tuning process
  --checkpoint_path ./path/to/imagenet/checkpoint.pth \
  --ndf 192 \
  --n_rkhs 1536 \
  --batch_size 480 \
  --tclip 20.0 \
  --res_depth 8 \
  --dataset Places205 \
  --input_dir /path/to/places205 \
  --amp
```

> When restoring from an encoder checkpoint, the evaluator will be re-initialized before training again.

## Enabling Mixed Precision Training (`--amp`)
If your GPU supports half precision, you can take advantage of it when training by passing the `--amp` (automatic mixed precision) flag.    
We use [NVIDIA/apex](https://github.com/NVIDIA/apex) to enable mixed precision, so you will need to have Apex installed, see: [Quick Start](https://github.com/NVIDIA/apex#quick-start).

## Citation

```
@article{bachman2019amdim,
  Author={Bachman, Philip and Hjelm, R Devon and Buchwalter, William},
  Journal={arXiv preprint arXiv:1906.00910},
  Title={Learning Representations by Maximizing Mutual Information Across Views},
  Year={2019}
}
```

## Contact

For questions please contact Philip Bachman at `phil.bachman at gmail.com`.

