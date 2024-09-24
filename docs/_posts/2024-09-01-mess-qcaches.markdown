---
layout: single
title:  "[Mess-around Series] Anywhere Q-caches"
date:   2024-09-01 13:00:21 +0200
author_profile: true
mathjax: true
classes: wide
customcss: true
author: yuc
permalink: /posts/2024-09-01-mess-qcaches
category:
    - ai
---

Remember when we tried [Q-caches in Temporal Transformer](/posts/2024-08-24-trick-weighted-q-caches){:target="blank"} and it was a success? 

Well, there were some pretty magical results during experiments when I applied Q-caches to the Spatial Transformer's second attention module.

![]({% link /assets/imgs/3dunet_diffusion/STattn_qdimension_STattn2.jpg %} "Spatio-Temporal Attention Overview")


### Q-caches at Spatio Transformer's 2nd attention module

<table class="center">
<tbody>
<tr>
    <td><img src="/assets/imgs/STqcache_attn2_weighted90/1.gif"/></td>
    <td><img src="/assets/imgs/STqcache_attn2_weighted90/2.gif"/></td>

</tr>
<tr>
    <td><img src="/assets/imgs/STqcache_attn2_weighted90/3.gif"/></td>
    <td><img src="/assets/imgs/STqcache_attn2_weighted90/4.gif"/></td>
</tr>
<tr>
    <td><img src="/assets/imgs/STqcache_attn2_weighted90/5.gif"/></td>
    <td><img src="/assets/imgs/STqcache_attn2_weighted90/6.gif"/></td>
</tr>
</tbody>
</table>
