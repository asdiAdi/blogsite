+++
title = "Ecommerce"
date = "2025-02-24"
draft = false
cover = "/images/cover/ecommerce.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++
I made a full-stack e-commerce app hosted on AWS EC2 using Next.js and Nest.js.
<!--more-->

{{< details summary="TLDR" >}}
- Technologies used: [Next.js](https://nextjs.org/), [Nest.js](https://nestjs.com/), [Postgresql](https://www.postgresql.org/), [AWS](https://aws.amazon.com/)
- Unfinished. Made to relearn backend development.
{{< /details >}}

After making the [connect4](/projects/connect4-app) app. I was confident and wanted to make an ecommerce app.
Why? Because ecommerce is one of the most widely used applications on the internet.
As such there are a lot of technologies made for it.
The number of ecommerce websites in the internet is insane.
Learning it inside out might give me some job security.

I decided to use the most popular frameworks for it thinking it would be an easier approach.
I used Next.js for SSR, Nestjs for backend, Postgresql for database and AWS for deployment.
The design is not mine though, I copied it from a paid template on Material UI hahaha.
I just went to their demo website and replicate it as much as I can using DaisyUI and Tailwind.

Halfway through the project I got bored and left it now as it is.
There are a lot of features that I still want to implement but I think the purpose is already achieved.
That is to remind my brain on how to code backend again.

Features I finished at the top of my head:
- Authentication using JWT tokens
- CRUD for adding/removing to cart, wishlist, address page, card, etc.
- 80% of the Frontend UI
- Search products
- Product Search Category
- Theming
- Pagination
- Private/Public Pages


TODO (I might not complete this because I want to do more interesting projects):
- Checkout and Order page. (I already completed the backend stuff but got lazy continuing on the frontend)
- Cursor Pagination
- Stripe Integration
- Fix Bugs
