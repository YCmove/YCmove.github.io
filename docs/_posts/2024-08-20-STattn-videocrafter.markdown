---
layout: posts
title:  "Spatio-Temporal Attention(ST-Attention) explained"
date:   2024-08-20 17:51:33 +0200
categories: diffusion
---

Spatio-Temporal Attention Mechanisms(ST-attention) is the building block of the 3D-Unets in Video Diffusion Models(VDMs), including the Crafter family [1](https://arxiv.org/abs/2310.19512), [2](https://arxiv.org/abs/2401.09047), [3](https://arxiv.org/abs/2310.12190), AnimateAnything [4](https://arxiv.org/abs/2311.12886), and Tune-A-Video [5](https://arxiv.org/abs/2212.11565). The core spirit is to facilitate the tempocal coherence for video generation. In this blog, we use the VideoCrafter2 as the example in the artical to showcase the architecture of the ST-attention.

## Essential of the 2D U-Nets
- U-shaped structure that resembles the Encoder/decoder design. Encoder is a contracting path on the left side and the decoder is an expansive path on the right. During encoding/contraction, we downsampling the feature map, while upsampling the channel. On the other hand, in the decoder/expansion part, we upsampling the feature map whereas downsampling the channel. The special mention is that in the beginning of each decoder stage, we concaenate the correspondingly feature map(along the channel axis) from the encoder. This is so-called the skip or residual connection.

- Skip/Residual connection
[2D-Unet](https://arxiv.org/abs/1505.04597)
![]({% link /assets/imgs/unet2d.png %} "2D U-Net")
[3D-Unet](https://arxiv.org/abs/2204.03458)
![]({% link /assets/imgs/unet3d.png %} "3D U-Net")



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

## TL;DR
Video is basically the slack of the images, thus
```
# 2D U-net
input_x = (batch, channel, height, width)

# 3D U-net
input_x = (batch, frames, channel, height, width)
```
That's it! All the conceptial operations(squeezing the feature map while upsampling the channel in decoder and vice versa) is the same as the 2D U-net.

Detailed 3D-Unets in VideoCrafter2(from the bottom to top):
[![](/assets/imgs/unet3d_details.jpg)](https://drive.google.com/file/d/1g39KaudeqBXqn_Zqv-agyo41RoQ-6aE0/view?usp=drive_link)

The input latents has feature map with sizes`(40, 64)`, and 
![]({% link /assets/imgs/3dunet_table.png %} "U-shape Networks")


## Factorizing the Space and Time Dimensions






