---
layout: posts
title:  "[FIFO-Diffusion+] Improving Visual Consistency for Long Video Generation"
date:   2024-08-20 17:51:33 +0200
author_profile: true
classes: wide
customcss: true
author: yuc
---

This series began with a perplexing body-flipped video…

<table class="center">
<thead>
    <tr>
        <th>VideoCrafter2</th>
        <th>Extended by FIFO-Diffusion</th>
    </tr>
</thead>
<tbody>
<tr><td style="text-align:center;" colspan="2">"a person swimming in ocean, high quality, 4K resolution."</td></tr>
<tr>
    <td><img src="/assets/imgs/a_person_swimming_in_ocean/origin.gif"/></td>
    <td><img src="/assets/imgs/a_person_swimming_in_ocean/fifo_origin.gif"/></td>
</tr>
</tbody>
<thead>
    <tr>
        <th colspan="2">Inconsistancy in FIFO-Diffusion (frame 85-95)</th>
    </tr>
</thead>
<tbody>
<tr>
    <td colspan="2"><img src="/assets/imgs/a_person_swimming_in_ocean/85-95_fifo/body_flipping.gif"/></td>
</tr>
</tbody>
</table>

How can I fix visual inconsistency in long video generation in FIFO-Diffusion? I rolled up my sleeves and started digging around..

After some exploring, I uncovered a few tricks that can help (no cats were harmed because I don't have one.):

1. **[Seeding the initial latent frame](/posts/2024-08-22-trick-seeding-initial-frame)**
2. **[Weighted Q-caches](/posts/2024-08-24-trick-weighted-q-caches)**
3. **[Extending the Latent Uniformly](/posts/2024-08-27-trick-uniform-latent)**

Implemented code -> [my GitHub](https://github.com/YCmove/FIFO-Diffusion_public)