---
layout:     post
title:      "DiSCO使用指南"
subtitle:   " \"DiSCO Tutorial\""
date:       2022-03-20 15:00:00
author:     "xxc"
header-img: "img/DiSCO/Disco-bg.png"
catalog: true
tags:
    - SLAM
    - 工作分享

---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

### Basic Knowledge:

[DiSCO：Differentiable Scan Context with Orientation](https://ieeexplore.ieee.org/abstract/document/9359460)是我去年在RAL上发表的一篇关于地点重识别的文章，它可以做到又快又好地识别室外场景中的回环，同时还能估计回环关键帧之间的相对旋转。我已经把DiSCO集成到一个大系统中，目前运行下来，表现非常令人满意。具体的介绍我之前的一篇blog有写，详见[DiSCO 介绍](https://maverickpeter.github.io/2021/09/19/DiSCO/)。


### Motivation for this blog

最近有位同学在github上的repo给我提issue，我才发现上传到github的版本还有许多小bug没有及时修复。因此我以这篇教程文章为契机，顺便对开源的代码进行查漏补缺。开源代码在这里：[DiSCO code](https://github.com/MaverickPeter/DiSCO-pytorch)

### Dependencies

- Ubuntu: 16.04-20.04
- ROS: kinetic-noetic (optional，如果你需要接入ROS)
- PyTorch： 1.4.0-1.6.0 (fft module is modified after 1.6.0)

这里PyTorch中的fft模块在1.6.0版本之后修改了接口，其实要改也很好改，就在models/DiSCO.py里面把torch.rfft()这个函数用新的替代掉就好了。

### A Quick Use

#### w/o ROS
```
# 0. Clone the code from 
git clone https://github.com/MaverickPeter/DiSCO-pytorch
cd DiSCO-pytorch && mkdir log

# 1. Prepare python environment (e.g.:torch 1.4.0; python 3.7)
conda create -n disco_py37 python==3.7
conda activate disco_py37
pip install -r requirements.txt

# 2. Build point cloud processor
cd multi-layer-polar-cython/cython
python setup.py build_ext --inplace
mv gputransform.cpython-37m-x86_64-linux-gnu.so ../../

# 3. Download the pretrained model from https://drive.google.com/file/d/1yGFtNUavJT0kxS_mtYAE8U3FzOTpt3c4/view?usp=sharing and move the model.ckpt to the log/

# 4. Try inference.py
python inference.py

```

#### w/ ROS

```
# 0. Prepare workspace and clone the code from 
mkdir -p disco_ws/src && cd disco_ws/src
git clone https://github.com/MaverickPeter/DiSCO-pytorch
cd DiSCO-pytorch && mkdir log
mv dislam_msgs ../

# 1. Prepare python environment (e.g.:torch 1.4.0; python 3.7)
conda create -n disco_py37 python==3.7
conda activate disco_py37
pip install -r requirements.txt

# 2. Build point cloud processor
cd multi-layer-polar-cython/cython
python setup.py build_ext --inplace
mv gputransform.cpython-37m-x86_64-linux-gnu.so ../../

# 3. Download the pretrained model from https://drive.google.com/file/d/1yGFtNUavJT0kxS_mtYAE8U3FzOTpt3c4/view?usp=sharing and move the model.ckpt to the log/

# 4. catkin_make to generate DiSCO msg
cd disco_ws/
catkin_make

# 5. Try infer_ros.py
source devel/setup.bash
rosrun disco_ros infer_ros.py 
```

### Train DiSCO from Scratch
#### 1. Prepare data
```
# 1. Move the gputransform.cpython-37m-x86_64-linux-gnu.so to generating_queries/nclt/
# 2. Modify config.py
# 3. Prepare folder structure. For NCLT dataset, if you want to use our code for convenience you have to form the data into this file structure. **(occ_xm is an empty folder)**
├── 2012-02-04
│   ├── ground_truth
│   ├── occ_0.5m
│   └── velodyne_data
│       └── velodyne_sync
├── 2012-03-17
│   ├── ground_truth
│   ├── occ_3m
│   └── velodyne_data
│       └── velodyne_sync
...(you may need to download at least 4 rounds)

# 4. Generate pickle files
python generate_training_tuples_baseline_with_pose.py
python generate_test_sets.py

# 5. Train
cd ../../
python train_DiSCO.py
```

### DiSCO w/o Network

```
class DiSCO(nn.Module):
    def __init__(self, output_dim):
        super(DiSCO, self).__init__()

        self.out_dim = output_dim
        self.col = int(np.sqrt(self.out_dim)/2)
        self.unet = UNet(cfg.num_height)
        self.device = torch.device("cuda:0")

    def forward_fft(self, input):
        median_output = torch.rfft(input, 2, normalized=True, onesided=False)
        median_output_r = median_output[:, :, :, :, 0]
        median_output_i = median_output[:, :, :, :, 1]
        output = torch.sqrt(median_output_r ** 2 + median_output_i ** 2 + 1e-15)
        output = fftshift2d(output)
        return output, median_output

    def forward(self, x):

        batchsize = x.size()[0]

        unet_out = self.unet(x) 
        # 如果你不想用network，直接把高度维加起来过fft也可以得到很好的效果。
        # unet_out = torch.sum(x, dim=1, keepdim=True)

        out, fft_result = self.forward_fft(unet_out)
        
        x = out.squeeze(1)
        x = x[:, (cfg.num_ring//2 - self.col):(cfg.num_ring//2 + self.col), (cfg.num_sector//2 - self.col):(cfg.num_sector//2 + self.col)]
        x = x.reshape(batchsize, -1)

        return x, out, fft_result, unet_out
```
