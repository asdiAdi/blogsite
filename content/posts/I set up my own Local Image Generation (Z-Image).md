+++
title = "I set up my own Local Image Generation (Z-Image)"
date = "2025-12-05"
draft = false
cover = "images/cover/z-image.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++

<!--more-->

{{< details summary="TLDR" >}}
- Successfully ran local AI text to image generation using ComfyUI.
{{< /details >}}


I always thought AI and machine learning is way too complicated for me to replicate on my pc.
I know that you need a ton of training data, a powerful gpu, lots of electricity bills and a lot of things to learn before I could do these.
But then [Z-Image](https://github.com/Tongyi-MAI/Z-Image) was released by alibaba (the chinese giant) and it was trending on the AI community.
I got interested because of its supposed features and performance over the western counterparts.
And most important of all, it is open source!

## Installation

After reading some articles and tutorials. It's actually pretty easy. 
There were already open-source software that dumbs down the whole process, even a guy without any programming background can do it.

1. First you need ComfyUI. Download it at https://www.comfy.org/.
It doesn't matter what method you installed it.
2. Copy the official workflow to ComfyUI. https://github.com/Comfy-Org/workflow_templates/blob/main/templates/image_z_image_turbo.json
3. Next, download the Z-Image by following the instructions from the workflow.
4. (Optional). If your vram is too low you can still run it by downloading the appropriate GGUF. Copy this workflow and download the appropriate size for your needs. https://huggingface.co/jayn7/Z-Image-Turbo-GGUF
6. Run the thing.

## Notes
- I have a RTX3080, but I still get 6-10 seconds image generation on a 1024 x 1024 size image.
- You can increase the size, but it will also exponentially take more time.
- There is a repository for different algorithms other than Z-Image. Some are even trained on NSFW stuff o_O.

## Images
![z-image-2](/images/posts/z-image/z-image-2.png)
![z-image-3](/images/posts/z-image/z-image-3.png)
![z-image-4](/images/posts/z-image/z-image-4.png)
![z-image-5](/images/posts/z-image/z-image-5.png)
![z-image-6](/images/posts/z-image/z-image-6.png)


## Conclusion
Although the results are amazing. I still don't know its purpose.
What do I do with these? T_T