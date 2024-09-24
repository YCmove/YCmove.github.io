---
layout: single
title:  "3D U-Net in Video Diffusion Models(VDMs)"
date:   2024-08-19 22:08:01 +0200
author_profile: true
mathjax: true
classes: wide
author: yuc
permalink: /posts/2024-08-19-3d-u-net-in-video-diffusion-models
last_modified_at: 2024-08-19 22:08:01 +0200
category:
    - FIFO-Diffusion Series
# tags:
#    - blog
---

> Latent Diffusion Models = Autoencoder + Forward process + Denoising process

The Denoising part is the core because it's the **generative process** that reconstructs the image in latent space.

Here's a simple way to think about it: In the forward process, we start with a clear picture and gradually add noise to it. In the reverse process, we start with a noisy picture and try to clean it up.

U-Net is one of the popular neural networks for the denoising process. It is trained to iteratively estimate the noise at each step, refining the noisy latent toward the clean target.

What's the hack in it?


## 2D U-Net
![]({% link /assets/imgs/3dunet_diffusion/unet2d.png %} "2D U-Net")

It's got a U-shaped structure that resembles the Encoder/Decoder design architecture. On the left, we got the **contracting path**, which contracts and compresses features, downsampling the feature map, and upsampling the channels.

On the right side, the **expansive path** is to recover the feature map while downsampling the channels.

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


## From 2D to 3D U-Net

When it comes to video, you're dealing with a stack of images. So, while a 2D U-Net is processing images with input dimensions like:
```
# 2D U-net
input_x = (batch, channel, height, width)
```
We just added a whole new dimension to the 3D U-Net:
```
# 3D U-net
input_x = (batch, frames, channel, height, width)
```
The key factor for 3D U-Net is being able to **upsample and downsample with visual and temporal awareness**. In the VideoCrafter model, they achieve this by (1) ResNet fused with timestep and fps information and (2) the spatio-temporal attention mechanism.


## 3D U-Net in VideoCrafter
VideoCrafter2 is the text-to-video(T2V) and image-to-video(I2V) generative model; this architecture is also used in StyleCrafter [1](https://arxiv.org/abs/2312.00330){:target="_blank"}, DynamiCrafter [3](https://arxiv.org/abs/2310.12190){:target="_blank"}, etc. The authors(Chen et al.) show a simplified figure of its architecture:

![]({% link /assets/imgs/3dunet_diffusion/videocrafter_overview1.png %} "3D U-Net in VideoCrafter")

As always, there is more than meets the eye for all deep learning papers! I looked into VideoCrafter's [codebase](https://github.com/AILab-CVC/VideoCrafter){:target="_blank"} and drew a more sophisticated figures as follows:

Google Drive: [3D U-Net in VideoCrafter](https://drive.google.com/file/d/1DUwz0NqpvYYC1DO5IQSItORpL2nW-G5C/view?usp=drive_link){:target="_blank"}
![]({% link /assets/imgs/3dunet_diffusion/Unet_skipcon.jpg %} "3D U-Net in VideoCrafter")

Google Drive: [Detailed 3D U-Net in VideoCrafter](https://drive.google.com/file/d/1d9w8vV4qXRoMuf3Ypu-WZZTwx0RNnmzx/view?usp=drive_link){:target="_blank"}, the network starts from the bottom to top.
![]({% link /assets/imgs/3dunet_diffusion/unet3d_details.jpg %} "Detailed 3D U-Net in VideoCrafter")

The size changes are shown in the following table, where `f` frames, `c` channels, and `h` and `w` are the feature map's height and width.
![]({% link /assets/imgs/3dunet_diffusion/3dunet_table.png %} "Dimension changes in 3D U-Net")

- The denoising 3D U-Net part consists of 12 encoder layers and also 12 layers of the decoder. 
- In the contracting path, the feature map is compressed down to 1/4.
- In the contracting path, the channel is expanded 4 times. 
- The temporal dimension (16 frames) is neither compressed nor expanded, but is used in the networks.


## Building Blocks in 3D U-Net
![]({% link /assets/imgs/3dunet_diffusion/3dunet_buildingblock.png %} "RestNet and ST-attention")

The core building block of the 3D U-Net in video diffusion models utilizes a combination of residual networks (ResNet) and spatio-temporal attention mechanisms. Additional information includes: texts embedding(prompts) for T2V, image embedding for I2V, and the temporal data such as denoising timesteps and the FPS as the motion speed control.

### Temporal data embedding
Diffusion steps(timesteps) and motion speed control(FPS). They are fused as follows ([code](https://github.com/AILab-CVC/VideoCrafter/blob/11bcd76fc62fb98b9715b994afe45b4fa081120c/lvdm/models/utils_diffusion.py#L8){:target="_blank"}):
![]({% link /assets/imgs/3dunet_diffusion/Timesteps_fps_emb.jpg %} "Timesteps and FPS fusion.")

### ResNet
ResNet modules are in charge of both upsampling and downsampling. These modules contain convolutional layers responsible for the upsampling and downsampling ([code](https://github.com/AILab-CVC/VideoCrafter/blob/11bcd76fc62fb98b9715b994afe45b4fa081120c/lvdm/basics.py#L36){:target="_blank"}). The temporal embedding from the previous step is integrated into the ResNet module by adding it to the latent along the channel dimension. Additionally, ResNet modules contain temporal convolutions that further refine the temporal representation within the latent space ([code](https://github.com/AILab-CVC/VideoCrafter/blob/11bcd76fc62fb98b9715b994afe45b4fa081120c/lvdm/modules/networks/openaimodel3d.py#L230){:target="_blank"}).


### Text and Image Embedding
In the denoising process, text and image data serve as conditional information. Both are encoded using the [CLIP model](https://huggingface.co/docs/transformers/en/model_doc/clip){:target="_blank"}. For text-to-video (T2V) diffusion, the CLIP-encoded text becomes the context embedding, which is later fed into cross-attention to generate keys and values. In VideoCrafter's image-to-video (I2V) model, a separate projection network is trained to align the image embedding with the CLIP-encoded text embedding ([code](https://github.com/AILab-CVC/VideoCrafter/blob/11bcd76fc62fb98b9715b994afe45b4fa081120c/lvdm/models/ddpm3d.py#L692){:target="_blank"}). These are then concatenated together to form the context embedding ([code](https://github.com/AILab-CVC/VideoCrafter/blob/11bcd76fc62fb98b9715b994afe45b4fa081120c/scripts/evaluation/funcs.py#L34){:target="_blank"}).


### Spatio-Temporal Attention Mechanisms
The Spatio-Temporal Attention mechanism (ST-attention) is a sequential block of a spatial transformer followed by a temporal transformer. **Each transformer --- both spatial and temporal --- includes two attention modules: self-attention and cross-attention.**
![]({% link /assets/imgs/3dunet_diffusion/STattn_overview.jpg %} "Spatio-Temporal Attention Overview")
The modules within ST-Attention operate as follows:
1. Spatial self-attention: Each "pixel" in the feature map attends to every other pixel, creating a pixel-to-pixel attention map for a single frame in latent space.
2. Spatial cross-attention: The output from spatial self-attention is projected as queries, while the context embedding is projected into keys and values.
3. Temporal self-attention: Each frame attends to all other frames, building a frame-to-frame attention map across a batch of latent frames.
4. Temporal cross-attention: The output from temporal self-attention is projected as queries, and the context embedding is again used for keys and values.

**ST-Attention for Text-to-Video (T2V)**
![]({% link /assets/imgs/3dunet_diffusion/STblock_text.jpg %} "Spatio-Temporal Attention")

**ST-Attention for Image-to-Video (I2V)**
The I2V model in VideoCrafter adds one more cross-attention block.
![]({% link /assets/imgs/3dunet_diffusion/videocrafter_dualcrossattn.png %} "From VideoCrafter1 paper")
The modules within ST-Attention operate as follows:
1. Spatial self-attention
2. Spatial cross-attention: After spatial self-attention, the output is projected as queries, while the context embedding is split into text and image embeddings. These are then separately projected into keys and values, resulting in two attention outputs ($$\text{Out}_{\text{text}}$$ and $$\text{Out}_{\text{img}}$$). The final output is obtained by adding these two outputs together, referred to as "dual cross-attention" in the paper.
3. Temporal self-attention
4. Temporal cross-attention: Similar dual cross-attention is applied here as well. 
![]({% link /assets/imgs/3dunet_diffusion/STblock_dualtextimg.jpg %} "Spatio-Temporal Attention")


### Reference
- [U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/abs/1505.04597){:target="_blank"}
- [(3D-Unet) Video Diffusion Models](https://arxiv.org/abs/2204.03458){:target="_blank"}


