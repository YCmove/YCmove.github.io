---
layout: posts
title:  "[FIFO-Diffusion+] Weighted Q-caches"
date:   2024-08-24 09:23:10 +0200
author_profile: true
mathjax: true
classes: wide
customcss: true
author: yuc
permalink: /posts/2024-08-24-trick-weighted-q-caches
---


While I was searching for methods to improve visual consistency, two papers jumped out at me:

- [ConsiStory: Training-Free Consistent Text-to-Image Generation](https://arxiv.org/abs/2402.03286){:target="_blank"}
- [Cross-Image Attention for Zero-Shot Appearance Transfer](https://arxiv.org/abs/2311.03335){:target="_blank"}

Research indicates that in denoising 3D U-Nets with attention mechanisms, the queries carry **structural information**.

![]({% link /assets/imgs/3dunet_diffusion/crossattn_qkv.png %} "Cross-Image Attention for Zero-Shot Appearance Transfer")
![]({% link /assets/imgs/3dunet_diffusion/crossattn_overview.png %} "Cross-Image Attention for Zero-Shot Appearance Transfer")

In ConsiStory, they use vanilla query features to improve the diversity of the generated image while maintaining the structure. They introduce blending of encoder and decoder queries in their Subject Driven Self-Attention modules.
![]({% link /assets/imgs/3dunet_diffusion/consistory_qcache.png %} "ConsiStory - Using Vanilla Query Features")

Inspired by these insights, I went on to identify actionable codes in VideoCrafter's 3D U-Net. The first challenge I noticed is that "not all layers are transformers." As shown below, the blocks marked in olive green are transformers (ST-attention). This makes it impossible to map queries directly from each encoder layer to the decoders. 

[Google Drive](https://drive.google.com/file/d/1cby3S2QylL4r9XakL8y7ZPpwErPuM3VU/view?usp=drive_link){:target="_blank"}
![]({% link /assets/imgs/3dunet_diffusion/Unet_skipcon.jpg %} "3D U-Net Diffusion Model")

Since direct mapping wasn't possible, I took a manual approach. I selected corresponding queries from the encoder to map to the decoder, ensuring they have the same dimensions for weighted addition. Nothing too complex --- just making sure the pieces fit together perfectly.
$$ 
Q^* = (1-v_t)Q_{\text{decoder}} + v_t \underbrace{Q_{\text{cache}}}_{=Q_{\text{encoder}}}
$$

I tried experimenting with different values for $$v_t$$ to see if I could improve the results and started following the paper with linearly decreased $$v_t=[0.9, 0.8]$$, but they didn't make a big difference. So, I stuck with a fixed value of 0.9. This means that 90% of the $$Q_{\text{decoder}}$$ are replaced by the one from the encoder.

<!-- We test with linearly decreased $$v_t=[0.9, 0.8]$$, but didn't find it more effective. so i stick to simplie fixed scalar $$v_t=0.9$$, which means that the $$Q_{\text{cache}}$$ from encoder replace 90% of the $$Q_{\text{decoder}}$$. -->

[Google Drive](https://drive.google.com/file/d/1DUwz0NqpvYYC1DO5IQSItORpL2nW-G5C/view?usp=drive_link){:target="_blank"}
![]({% link /assets/imgs/3dunet_diffusion/Unet_qcaches.jpg %} "3D U-Net Diffusion Model with weighted q-caches")

Since I aimed to enhance temporal visual consistency, I targeted the self-attention modules in the temporal transformer for applying Q-caches. Here's the breakdown:
![]({% link /assets/imgs/3dunet_diffusion/STattn_qdimension.jpg %} "Spatio-Temporal Attention Overview")
- Shape of Q = (batch, frames, vector length) = (12800, 16, 64)
- Shape of Self-attention map $$QK^T$$: (12800, 16, 16)

In simpler terms, each "pixel" on the feature map (40x64) gets processed by 5 heads, which results in 12800 batches. The self-attention mechanism focuses solely on frame-to-frame relationships, with each frame represented by a vector of length 64.

By applying Q-caches to the temporal transformer, I hope to consistently improve the "**frame-to-frame structure**" as the video progresses. While "structure" is a bit abstract, consider it the underlying pattern that might hold the footage together.

### Results


<table class="center">
<thead>
    <tr>
        <th>FIFO-Diffusion</th>
        <th>+ Q-Caches</th>
    </tr>
</thead>
<tbody>
<tr><td style="text-align:center;" colspan="2">"a bicycle accelerating to gain speed, high quality, 4K resolution."</td></tr>
<tr>
    <td><img src="/assets/imgs/a_bicycle_accelerating_to_gain_speed/fifo_origin.gif"/></td>
    <td><img src="/assets/imgs/a_bicycle_accelerating_to_gain_speed/TTqcache_attn1_weighted90.gif"/></td>
</tr>
</tbody>
<!-- <thead>
    <tr>
        <th colspan="2">Improved Segment (Frame 85-95)</th>
    </tr>
</thead>
<tbody>
<tr>
    <td><img src="/assets/imgs/a_bicycle_accelerating_to_gain_speed/85-95_fifo/body_flipping.gif"/></td>
    <td><img src="/assets/imgs/a_bicycle_accelerating_to_gain_speed/85-95_TTqcache_attn1_weighted90/body_flipping.gif"/></td>
</tr>
</tbody> -->
</table>
<br>
<table class="center">
<thead>
    <tr>
        <th>FIFO-Diffusion</th>
        <th>+ Q-Caches</th>
    </tr>
</thead>
<tbody>
<tr><td style="text-align:center;" colspan="2">"a person swimming in ocean, high quality, 4K resolution."</td></tr>
<tr>
    <td><img src="/assets/imgs/a_person_swimming_in_ocean/fifo_origin.gif"/></td>
    <td><img src="/assets/imgs/a_person_swimming_in_ocean/TTqcache_attn1_weighted90_2.gif"/></td>
</tr>
</tbody>

<thead>
    <tr>
        <th colspan="2">Improved Segment (Frame 85-95)</th>
    </tr>
</thead>
<tbody>
<tr>
    <td><img src="/assets/imgs/a_person_swimming_in_ocean/85-95_fifo/body_flipping.gif"/></td>
    <td><img src="/assets/imgs/a_person_swimming_in_ocean/85-95_TTqcache_attn1_weighted90/body_flipping.gif"/></td>
</tr>
</tbody>
</table>
<br>
<table class="center">
<thead>
    <tr>
        <th>FIFO-Diffusion</th>
        <th>+ Q-Caches</th>
    </tr>
</thead>
<tbody>
<tr><td style="text-align:center;" colspan="2">"a car slowing down to stop, high quality, 4K resolution."</td></tr>
<tr>
    <td><img src="/assets/imgs/a_car_slowing_down_to_stop/fifo.gif"/></td>
    <td><img src="/assets/imgs/a_car_slowing_down_to_stop/TTqcache_attn1_weighted90.gif"/></td>
</tr>
</tbody>

<thead>
    <tr>
        <th colspan="2">Improved Segment (Frame 73-80)</th>
    </tr>
</thead>
<tbody>
<tr>
    <td><img src="/assets/imgs/a_car_slowing_down_to_stop/fifo_73-80/animation.gif"/></td>
    <td><img src="/assets/imgs/a_car_slowing_down_to_stop/TTqcache_attn1_weighted90_73-80/animation.gif"/></td>
</tr>
</tbody>
</table>

<!-- a person giving a presentation to a room full of colleagues, high quality, 4K resolution. -->
<br>
<table class="center">
<thead>
    <tr>
        <th>FIFO-Diffusion</th>
        <th>+ Q-Caches</th>
    </tr>
</thead>
<tbody>
<tr><td style="text-align:center;" colspan="2">"a person giving a presentation to a room full of colleagues, high quality, 4K resolution."</td></tr>
<tr>
    <td><img src="/assets/imgs/a_person_giving_a_presentation_to_a_room_full_of_colleagues/fifo.gif"/></td>
    <td><img src="/assets/imgs/a_person_giving_a_presentation_to_a_room_full_of_colleagues/TTqcache_attn1_weighted90.gif"/></td>
</tr>
</tbody>

<thead>
    <tr>
        <th colspan="2">Improved Segment (Frame 100-110)</th>
    </tr>
</thead>
<tbody>
<tr>
    <td><img src="/assets/imgs/a_person_giving_a_presentation_to_a_room_full_of_colleagues/fifo_segment/animation.gif"/></td>
    <td><img src="/assets/imgs/a_person_giving_a_presentation_to_a_room_full_of_colleagues/TTqcache_attn1_weighted90_segment/animation.gif"/></td>
</tr>
</tbody>
</table>
Take the "a person swimming in the ocean" video as an example --- there's still some inconsistency between frames 119 and 134. In this part of the video, the person's back flips unnaturally when the camera dives below the ocean surface.

<table class="center">
<thead>
    <tr>
        <th>Inconsistancy around frame 119-134 (FIFO+Q-Caches)</th>
    </tr>
</thead>
<tbody>
<tr>
    <td><img src="/assets/imgs/a_person_swimming_in_ocean/119-136_TTqcache_attn1_weighted90/animation.gif"/></td>
</tr>
</tbody>
</table>

I looked closely at the denoising process for some problematic frames (119, 120, 121). In the figure below, the top-left item represents noise generated from a normal distribution $$z \sim \mathcal{N}(0, 1)$$ ([code](https://github.com/jjihwan/FIFO-Diffusion_public/blob/4ef71d4d89b578e5d2f6c9c8c108ab45b5738baf/scripts/evaluation/funcs.py#L42)).

<table class="center">
<thead>
    <tr>
        <th colspan="2">Denoising step by step of the Frame 119</th>
    </tr>
</thead>
<tbody>
<tr>
    <td colspan="2"><img src="/assets/imgs/a_person_swimming_in_ocean/denoise_119_8.jpg"/></td>
</tr>
</tbody>
<thead>
    <tr>
        <th colspan="2">Denoising step by step of the Frame 120</th>
    </tr>
</thead>
<tbody>
<tr>
    <td colspan="2"><img src="/assets/imgs/a_person_swimming_in_ocean/denoise_120_8.jpg"/></td>
</tr>
</tbody>
<thead>
    <tr>
        <th colspan="2">Denoising step by step of the Frame 121</th>
    </tr>
</thead>
<tbody>
<tr>
    <td colspan="2"><img src="/assets/imgs/a_person_swimming_in_ocean/denoise_121_8.jpg"/></td>
</tr>
</tbody>
</table>

Since I'm using FIFO-Diffusion to make long videos, we're working with a technique called "diagonal denoising." It's a bit complicated, so let's dive deeper into it in my next post --- [Uniform Latents](/posts/2024-08-27-trick-uniform-latent).

Improving Visual Consistency Series:

1. **[Seeding the initial latent frame](/posts/2024-08-22-trick-seeding-initial-frame)**
2. **[Weighted Q-caches](/posts/2024-08-24-trick-weighted-q-caches)**
3. **[Uniform Latents](/posts/2024-08-27-trick-uniform-latent)**