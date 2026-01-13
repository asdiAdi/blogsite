+++
title = "I installed Arch linux btw"
date = "2025-12-06"
draft = false
cover = "/images/cover/arch_linux.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++

<!--more-->

{{< details summary="TLDR" >}}

- Successfully installed arch linux with windows dual boot.
{{< /details >}}

## Background

One day I was in the middle of an important ranked match on TFT, a League of Legends spin-off game.
I was 1 match away from achieving my seasonal goal, which is to reach Masters rank.
Suddenly the dreaded blue screen of death flashed before my eyes.
I audibly gasped as I know for a fact that losing this match will drive me crazy.
I might go on a loss streak and spend days or even weeks grinding for my goal.
I quickly boot up my pc again and then BOOM! WINDOWS UPDATES.
That was the time I've had enough. This isn't an isolated incident.
Throughout my years of using Windows, there have always been those moments when it just decides to let you down.
So I decided, I will switch to Linux someday and forget about this garbage OS.

Here I am now trying to install Arch.
You may be asking why Arch?
Although I've used other distros like Ubuntu and Fedora before, I saw a lot of posts on reddit about this hard to install linux distro which all the cool kids should be using.
I figured why not? How hard can this be?

## Installation

Boy was I wrong. I followed the official documentation, but I still messed up the installation.
It took me the whole day debugging, retrying and installing.
I'm listing what I learned in case I forgot:

- When partitioning, you can indicate size with +8g for 8gb
- Before running command of setting hardware time, set local timezone to Asia/Manila first
- You don't need to partition efi boot if you already have it on other drives (Windows)
- Windows efi drives is too small (100mb). You may need to partition efi anyway. (This part is where I got stuck)
- Install using the grub-install command and make the grub-mkconfig output file.
- On the terminal, the format is user@hostname
- Set hostname by nano /etc/hostname. I set it to 'arch-btw'
- You also need to install gpu packages.
- Enable packages you installed that needs to run on boot e.g. NetworkManager.

## Problems I encountered

- I installed arch but it's not showing up on the bootloader
- I installed in the wrong order.
- I should've mounted the efi partition first before installing the linux kernel.
- WINDOWS AND THEIR EFI.
  - Apparently, 100mb is not enough for linux to install in the efi partition.
  - Although it is only recommended to use only 1 EFI partition, it's not strictly forbidden to add another one.
  - It all depends on the motherboard's specification.
  - As a general rule EFI partitions on different disks behave more stable.

## Packages

```
base base-devel efibootmgr 
git grub linux linux-firmware 
man nano networkmanager sudo

hyprland #via [end_4's dotfiles](https://github.com/end-4/dots-hyprland)
```

## Result

Overall I'm happy that I was successful on installing Arch even though I ran into some problems.
It's not much but it's honest work.

{{< figure src="/images/posts/arch_first.jpg" alt="first install" position="center" caption="First Successful Installation" captionPosition="center" >}}

{{< figure src="/images/cover/arch_linux.png" alt="hyperland" position="center" caption="Hyprland with end_4's config" captionPosition="center" >}}

## Thoughts

The system feels fast. Every animation is smooth. I've encountered 0 lag, 0 issues, 0 crashes.
It was so good that I didn't even realize my display was set to 60Hz not until several hours later.
I thought Arch-linux is buggy because it's a rolling release distro.
I'm glad that I was wrong. I guess this is it, I will continue to learn this on my free time.
In the future, I plan to configure my own settings and set up a script that I can use on all my devices.

## TODO

- Try out other tiling managers:
  - [niri](https://github.com/YaLTeR/niri)
  - [mangowc](https://github.com/DreamMaoMao/mangowc)
- Rice my own environment (editing config files to make it look good)
- Relearn [Neovim](https://neovim.io/)
