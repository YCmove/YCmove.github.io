---
layout: posts
title:  "3D U-Net in Video Diffusion Models(VDMs)"
date:   2024-08-20 17:51:33 +0200
categories: diffusion
---

3D U-Nets is one of the popular denoising networks in generative models. What's the hack in it?

Here, we use the VideoCrafter2 as the example in this post.

## 2D U-Nets
![]({% link /assets/imgs/unet2d.png %} "2D U-Net")

It's got a U-shaped structure that resembles the Encoder/Decoder design architecture. On the left, we got the **contracting path**, which contracts and compresses features, downsampling the feature map, and upsampling the channels.

On the right side, we meet the **expansive path** to recover the feature map while downsampling the channels.

And here's the special mention: At the start of each expansive stage, we grab the corresponding feature map from the contraction and concatenate it onto itself (along the channel axis). This is called the **Skip/Residual connection**, one of the most essential hacks in deep neural networks. You can check this [reddit](https://www.reddit.com/r/learnmachinelearning/comments/x0q39m/comment/im9czaj/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) post for an interesting interpretation.


<!-- |              |  f |     c     |    h   |    w   |                  |  f |      c     |    h   |    w   |              |
|:------------:|:--:|:---------:|:------:|:------:|:----------------:|:--:|:----------:|:------:|:------:|:------------:|
|   Encoder-0  | 16 |   4->320   |   40   |   64   | --Concate on c-> | 16 |  640->320  |   40   |   64   |  Decoder-11  |
|   Encoder-1  | 16 |    320    |   40   |   64   | --Concate on c-> | 16 |  640->320  |   40   |   64   |  Decoder-10  |
|   Encoder-2  | 16 |    320    |   40   |   64   | --Concate on c-> | 16 |  960->320  |   40   |   64   |   Decoder-9  |
|   Encoder-3  | 16 |    320    | 40->20 | 64->32 | --Concate on c-> | 16 |  960->640  | 20->40 | 32->64 |   Decoder-8  |
|   Encoder-4  | 16 |  320->640 |   20   |   32   | --Concate on c-> | 16 |  1280->640 |   20   |   32   |   Decoder-7  |
|   Encoder-5  | 16 |    640    |   20   |   32   | --Concate on c-> | 16 |  1920->640 |   20   |   32   |   Decoder-6  |
|   Encoder-6  | 16 |    640    | 20->10 | 32->16 | --Concate on c-> | 16 | 2560->1280 | 10->20 | 16->32 |   Decoder-5  |
|   Encoder-7  | 16 | 640->1280 |   10   |   16   | --Concate on c-> | 16 | 2560->1280 |   10   |   16   |   Decoder-4  |
|   Encoder-8  | 16 |    1280   |   10   |   16   | --Concate on c-> | 16 | 2560->1280 |   10   |   16   |   Decoder-3  |
|   Encoder-9  | 16 |    1280   |  10->5 |  16->8 | --Concate on c-> | 16 | 2560->1280 |  5->10 |  8->16 |   Decoder-2  |
|  Encoder-10  | 16 |    1280   |    5   |    8   | --Concate on c-> | 16 | 2560->1280 |    5   |    8   |   Decoder-1  |
|  Encoder-11  | 16 |    1280   |    5   |    8   | --Concate on c-> | 16 | 2560->1280 |    5   |    8   |   Decoder-0  |
| Middle Block | 16 |    1280   |    5   |    8   | No Concatenation | 16 |    1280    |    5   |    8   | Middle Block |-->


## From 2D to 3D-UNets

When it comes to video, you're dealing with a stack of images. So, while a 2D U-Net is processing images with input dimensions like:
```
# 2D U-net
input_x = (batch, channel, height, width)
```
We just added a whole new dimension to the 3D-UNets:
```
# 3D U-net
input_x = (batch, frames, channel, height, width)
```
The key factor is being able to **upsample and downsample with visual and temporal awareness**. In the VideoCrafter2 model, they achieve this by typical convolution with data fusion, and the spatio-temporal attention mechanism.


## 3D-Unets in VideoCrafter2
VideoCrafter2 is the text-to-video(T2V) and image-to-video(I2V) generative model, this architecture is also used in StyleCrafter [1](https://arxiv.org/abs/2312.00330), DynamiCrafter [3](https://arxiv.org/abs/2310.12190), etc. 

It consists 12 layers in contracting path, and also 12 layers of the expansive path. You can click [here](https://drive.google.com/file/d/1g39KaudeqBXqn_Zqv-agyo41RoQ-6aE0/view?usp=drive_link) to view the original image, the 3D-Unets starting from the bottom to top:
![]({% link /assets/imgs/unet3d_details.jpg %} "3D-Unets in VideoCrafter2")


The changes of the input latents is shown in the following table, where `f` is frames, `c` is channels, `h` and `w` are the height and width of the feature map.
![]({% link /assets/imgs/3dunet_table.png %} "Dimension changes in 3D-Unets")
We can see that during contracting path, the feature map is compressed 4 times, and the channel is expanded up 4 times. Vice versa in expansive path. The dimension of 16 frames is intact but used in the spatio-temporal attention module.

<!-- [](https://github.com/AILab-CVC/VideoCrafter/blob/11bcd76fc62fb98b9715b994afe45b4fa081120c/lvdm/modules/networks/openaimodel3d.py#L542) -->

<!-- [3D-Unet](https://arxiv.org/abs/2204.03458)
![]({% link /assets/imgs/unet3d.png %} "3D U-Net in Video Diffusion Model(VDM)") -->

```
emb = self.time_embed(t_emb)
```

We will look into the building block in [this post]().




### Reference
- [U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/abs/1505.04597)
- [(3D-Unet) Video Diffusion Models](https://arxiv.org/abs/2204.03458)

