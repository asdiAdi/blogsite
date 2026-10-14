+++
title = "Solar Tracker"
date = "2026-09-29"
draft = false
cover = "/images/cover/solar-tracker.png"
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++

[Website](https://solar.carladi.com)
[Github](https://github.com/asdiAdi/solar-tracker)

Simple PWA solar tracking app to track and estimate expenses.
<!--more-->

{{< details summary="TLDR" >}}

- Technologies used: [React](https://react.dev/), [Vite](https://vite.dev/), [TypeScript](https://www.typescriptlang.org/), [Tailwind](https://tailwindcss.com/), [AWS CDK](https://aws.amazon.com/cdk/), [S3 + CloudFront](https://aws.amazon.com/)
- PWA with live watts + battery %, daily/monthly/yearly views, kWh totals, PHP cost/savings calc. Live at `https://solar.carladi.com`
- Backend is CDK + Lambda + API Gateway + DynamoDB + SSM talking to the SolarMAN API, static frontend on S3 + CloudFront
  {{< /details >}}

We added a solar power system for our home last April, but we were wondering why the bills were still high. I found that the inverter we use has a public API with a logger. By building this, I found the cause of our problem and now we can monitor our system much easier. My parents also only cared about the savings data so other apps are way too complicated for them.

What it does:

- Live cards: solar / home / grid / battery watts + battery % with auto-refresh every 5 min, plus manual refresh
- Daily / Monthly / Yearly views with period tabs, date selector, and calendar modal. View persists in `localStorage`
- Totals (kWh): generated, consumed, grid import, system loss, bypass
- Cost (PHP): consumed, bypass, system loss, solar, net based on configurable elec rate
- Monthly update page (`/monthly-update`): password-gated form to submit `{year, month, rate, bypass_kwh}`
- PWA: installable, standalone, theme switcher, big-number UI with no graphs

The interesting bits are the polling + `?refresh=1` live endpoint, the billing-month labels for the monthly view, and keeping the UI simple enough for non-technical users while still exposing loss/bypass breakdowns for debugging high bills.

```sh
npm ci
npm run dev
npm run build:vite
npm run preview
```
