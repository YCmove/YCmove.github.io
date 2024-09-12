---
layout: single
title:  "[FIFO-Diffusion Series] Seeding the Initial generated frame as the image embedding"
date:   2024-08-22 19:20:07 +0200
author_profile: true
classes: wide
customcss: true
author: yuc
permalink: /posts/2024-08-22-trick-seeding-initial-frame

---

The original [FIFO-Diffusion](https://github.com/jjihwan/FIFO-Diffusion_public){:target="_blank"} supports only text-to-video generation, but out of curiosity, I decided to extend it to handle image-to-video as well, adapting from the [VideoCrafter](https://github.com/AILab-CVC/VideoCrafter){:target="_blank"}. Below are some results using input images and texts from [VBench](https://github.com/Vchitect/VBench?tab=readme-ov-file){:target="_blank"}:


<table class="center">
<thead>
    <tr>
        <th>Input Image</th>
        <th>VideoCrafter</th>
        <th>Extended by FIFO-Diffusion</th>
    </tr>
</thead>
<tbody>
<tr><td style="text-align:center;" colspan="3">"a city street with cars driving in the rain, high quality, 4K resolution."</td></tr>
<tr>
    <td width="33%"><img src="/assets/imgs/a_city_street_with_cars_driving_in_the_rain/512x.jpg"/></td>
    <td width="33%"><img src="/assets/imgs/a_city_street_with_cars_driving_in_the_rain/origin.gif"/></td>
    <td width="33%"><img src="/assets/imgs/a_city_street_with_cars_driving_in_the_rain/fifo.gif"/></td>
</tr>
<tr><td style="text-align:center;" colspan="3">"an older woman sitting in front of an old building, high quality, 4K resolution."</td></tr>
<tr>
    <td width="33%"><img src="/assets/imgs/an_older_woman_sitting_in_front_of_an_old_building/512x.jpg"/></td>
    <td width="33%"><img src="/assets/imgs/an_older_woman_sitting_in_front_of_an_old_building/origin.gif"/></td>
    <td width="33%"><img src="/assets/imgs/an_older_woman_sitting_in_front_of_an_old_building/fifo.gif"/></td>
</tr>
</tbody>
</table>

The main difference between text-to-video (T2V) and image-to-video (I2V) generation lies in the Spatio-Temporal Attention Mechanisms. If you're curious, check out my [previous posts]((/posts/2024-08-19-3d-u-net-in-video-diffusion-modelse#spatio-temporal-attention-mechanisms){:target="_blank"}) for more details. In short, it's all about the red-marked area in cross-attention where the image embedding interacts with latents and text at every layer of the denoising process. 

![]({% link /assets/imgs/3dunet_diffusion/STblock_dualtextimg_red.jpg %} "Image embedding in dual cross-attention")

The conditional image embedding strongly guides the diffusion model in creating frames that stay true to the input image. This got me thinking --- **If we have the image embedding in text-to-video pipeline, could it make the generated video more visually consistent?** 

Luckily, FIFO-Diffusion is built on an existing fixed-length video generator. That means we already have an initial image!

Pseudo code:
```python
def concat_text_img_emb(model, frame1, text_embbeding):
    # frame1 is the first frame in latent space (from the pre-trained T2V model)
    # Shape of frame1 = (4, 40, 64)
    # channel=4 (latent space channels), and (40, 64) is the sizes of the feature map
    
    # Decode the latent frame to obtain the image tensor
    frame_tensor = model.decode_first_stage_2DAE(frame1)

    # frame_tensor is the decoded image, shape = (3, 320, 512)
    # Where 3 is the number of color channels (RGB), and (320, 512) is the pixel resolution
    
    # Convert the image to image embedding.
    first_ref_img_emb = model.get_image_embeds(frame_tensor)

    # Concatenate the text embedding with the image embedding along the embedding dimension
    context_embbeding = [torch.cat([text_embbeding, first_ref_img_emb])]

    return context_embbeding
```

Experiment results:
<table class="center">
<thead>
    <tr>
        <th>FIFO-Diffusion</th>
        <th>Seeding the initial latent frame</th>
    </tr>
</thead>
<tbody>
<!-- <tr><td style="text-align:center;" colspan="2">"an older woman sitting in front of an old building, high quality, 4K resolution."</td></tr>
<tr>
    <td width="33%"><img src="/assets/imgs/an_older_woman_sitting_in_front_of_an_old_building/fifo.gif"/></td>
    <td width="33%"><img src="/assets/imgs/an_older_woman_sitting_in_front_of_an_old_building/t2v_cohe.gif"/></td>
</tr> -->
<tr><td style="text-align:center;" colspan="2">"a person swimming in ocean, high quality, 4K resolution."</td></tr>
<tr>
    <td width="33%"><img src="/assets/imgs/a_person_swimming_in_ocean/fifo_origin.gif"/></td>
    <td width="33%"><img src="/assets/imgs/a_person_swimming_in_ocean/t2v_cohe.gif"/></td>
</tr>
<tr><td style="text-align:center;" colspan="2">"a train crossing over a tall bridge, high quality, 4K resolution."</td></tr>
<tr>
    <td width="33%"><img src="/assets/imgs/a_train_crossing_over_a_tall_bridge/fifo.gif"/></td>
    <td width="33%"><img src="/assets/imgs/a_train_crossing_over_a_tall_bridge/t2v_cohe.gif"/></td>
</tr>
<tr><td style="text-align:center;" colspan="2">"a car slowing down to stop, high quality, 4K resolution."</td></tr>
<tr>
    <td width="33%"><img src="/assets/imgs/a_car_slowing_down_to_stop/fifo.gif"/></td>
    <td width="33%"><img src="/assets/imgs/a_car_slowing_down_to_stop/t2v_cohe.gif"/></td>
</tr>
</tbody>
</table>


<!-- ---

Improving Visual Consistency Series:

1. **[Seeding the initial latent frame](/posts/2024-08-22-trick-seeding-initial-frame)**
2. **[Weighted Q-caches](/posts/2024-08-24-trick-weighted-q-caches)**
3. **[Extending the Latent Uniformly](/posts/2024-08-27-trick-uniform-latent)** -->