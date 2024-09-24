---
layout: single
title:  "[Mess-around Series] Removing DDPM Random Noise"
date:   2024-09-03 20:38:16 +0200
author_profile: true
mathjax: true
classes: wide
customcss: true
author: yuc
permalink: /posts/2024-09-03-mess-ddim-noise
category:
    - ai
---

In Denoising Diffusion Probabilistic Models(DDPMs), each denoising step moves the latent closer to a cleaner versio. This step can be expressed as: (Eq. 12 from  [DDIM paper](https://arxiv.org/abs/2010.02502){:target="blank"} eq. 12)):

![]({% link /assets/imgs/3dunet_diffusion/ddim_eq12.png %} "DDIM paper Eq. 12")

And the corresponding [code](https://github.com/YCmove/FIFO-Diffusion_public/blob/4ef71d4d89b578e5d2f6c9c8c108ab45b5738baf/lvdm/models/samplers/ddim.py#L327){:target="blank"}:
```python
def ddim_step(self, sample, noise_pred, indices):
    ...
    # current prediction for x_0
    pred_x0 = (x - sqrt_one_minus_at * e_t) / a_t.sqrt()
    # direction pointing to x_t
    dir_xt = (1. - a_prev - sigma_t**2).sqrt() * e_t

    noise = sigma_t * noise_like(x.shape, device)

    x_prev = a_prev.sqrt() * pred_x0 + dir_xt + noise
```

What would happen if I remove the random noise part?
```python
x_prev = a_prev.sqrt() * pred_x0 + dir_xt
```

If we force the random noise to be zero, it's like turning off the randomness in the model. The model becomes a special case --- DDIM(Denoising Diffusion Implicit Models). To be more percise, the video is made using a DDIM sampling, but the base model (the one that does the heavy lifting) was trained using DDPM.

Now, that's reveil the results:

<table class="center">
<thead>
    <tr>
        <th colspan="1">FIFO-Diffusion</th>
        <th colspan="1">Remove Random Noise</th>
    </tr>
</thead>
<tbody>
<tr>
    <td colspan="2">"a person swimming in ocean, high quality, 4K resolution."</td>
</tr>
<tr>
    <td><img src="/assets/imgs/a_person_swimming_in_ocean/fifo_origin.gif"/></td>
    <td><img src="/assets/imgs/noDDIMnoise/ocean.gif"/></td>
</tr>
</tbody>
</table>
<br>

<table class="center">
<thead>
    <tr>
        <th colspan="1">FIFO-Diffusion</th>
        <th colspan="1">Remove Random Noise</th>
    </tr>
</thead>
<tbody>
<tr>
    <td colspan="2">"a person walking in the snowstorm, high quality, 4K resolution."</td>
</tr>
<tr>
    <td><img src="/assets/imgs/noDDIMnoise/snow_fifo.gif"/></td>
    <td><img src="/assets/imgs/noDDIMnoise/snow.gif"/></td>
</tr>
</tbody>
</table>

<br>

<table class="center">
<thead>
    <tr>
        <th colspan="1">FIFO-Diffusion</th>
        <th colspan="1">Remove Random Noise</th>
    </tr>
</thead>
<tbody>
<tr>
    <td colspan="2">"a person washing the dishes, high quality, 4K resolution."</td>
</tr>
<tr>
    <td><img src="/assets/imgs/noDDIMnoise/washdish_fifo.gif"/></td>
    <td><img src="/assets/imgs/noDDIMnoise/washdish.gif"/></td>
</tr>
</tbody>
</table>

<br>

<table class="center">
<thead>
    <tr>
        <th colspan="1">FIFO-Diffusion</th>
        <th colspan="1">Remove Random Noise</th>
    </tr>
</thead>
<tbody>
<tr>
    <td colspan="2">"alley"</td>
</tr>
<tr>
    <td><img src="/assets/imgs/noDDIMnoise/alley_fifo.gif"/></td>
    <td><img src="/assets/imgs/noDDIMnoise/alley.gif"/></td>
</tr>
</tbody>
</table>

In this case, even with the same 64 denoising steps, the implicit model struggles to generate **fine details** like waves, snowflakes, water droplets, and wall tiles.

It's pretty fascinating how much impact that random noise term has on these elements!
