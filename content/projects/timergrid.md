+++
title = "TimerGrid"
date = "2026-09-24"
draft = false
cover = "/images/cover/timergrid.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++

[Website](https://timergrid.carladi.com)
[Github](https://github.com/asdiAdi/timer-grid)

Browser multi-timer with a command-bar input. Timers persist in `localStorage` and keep running while the site is closed.
<!--more-->

{{< details summary="TLDR" >}}

- Technologies used: [Vite](https://vite.dev/), [React](https://react.dev/), [TypeScript](https://www.typescriptlang.org/), [Tailwind](https://tailwindcss.com/), [AWS CDK](https://aws.amazon.com/cdk/), [S3 + CloudFront](https://aws.amazon.com/)
- Static site, no backend or login. Live at `https://timergrid.carladi.com`
- Add one or many timers: `5m eggs`, `1h30m, 90s, 1:30`. Pause, resume, stop, delete, toggle, repeat by label or `all`
  {{< /details >}}

I use different OS environments on my computer and I like to time myself when studying or doing tasks.
I often need multiple timers at once, and I wanted something with fast response and controlled with keyboard instead of clicking around with a mouse.
I looked around and never found one I liked, so I built one myself.

I also wanted to try coding something entirely with AI, so the whole frontend is vibe coded.
The only part I did properly myself is the backend hosting structure.

The interesting bits are the command parser (`1h30m`, `90s`, `1:30`, bare `90` means seconds, comma for multi-add), the reducer + 100ms tick loop with wall-clock `endAt` so timers keep running while the site is closed, `localStorage` persistence with cross-tab sync, and Web Audio beeps plus custom mp3 uploads (those don't survive reload).
The display uses editable time segments with scrub support. I removed the analog clock along the way.

```sh
npm ci
npm run dev
npm run build
npm run preview
```
