+++
title = "Quotes Api"
date = "2025-12-03"
draft = false
cover = "/images/cover/quotes-api.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++
[Website](https://quote.carladi.com)
[Github](https://github.com/asdiAdi/quote)

I tried making a Rest API using AWS Gateway inside AWS Amplify based on [quotable](https://github.com/lukePeavey/quotable).
<!--more-->

{{< details summary="TLDR" >}}
- This project was made because the original repo has no working api left.
- Technologies used: [Amplify](https://aws.amazon.com/amplify/), [Gateway](https://docs.aws.amazon.com/apigateway/), [Lambda](https://docs.aws.amazon.com/lambda/),  [AWS CDK](https://docs.aws.amazon.com/cdk/), [Supabase](https://supabase.com/),
{{< /details >}}

I decided to make a basic rest api complete with a frontend design to jog my memory.
Luckily I am working on a project that requires an api that will dish out quotes.
After browsing for 10mins, I realized almost all apis are either abandoned or no longer free.
So I decided why not make it myself. How hard could it be?

Well it took me a week. T_T
Mainly because AWS amplify is dogshit. I shouldn't have used it.
I had a lot of errors where the solution was to delete the entire app and upload it again.
This ate up my time instead of just focusing on the logic of the app.
I won't use this service anymore.

Okay so the main feature I wanted was to fetch 1 quote of the day.
I originally wanted to have a lambda that will trigger once a day to get a random quote.
It will then update it to the database and that value is what I will fetch.
It was too complicated that I got lazy and basically just did this:

``` ts
  // total count of quotes
  const count = 2127;

  const minute = 1000 * 60;
  const hour = minute * 60;
  const day = hour * 24;

  // number of days since jan 1 1970
  const numDays = Math.round(Date.now() / day);

  const index = numDays - count * Math.floor(numDays / count);

  const response = await db
    .select({
      id: quotes.id,
      content: quotes.content,
      author: authors.name,
      length: sql<number>`LENGTH(${quotes.content})`,
      tags: sql<string[]>`ARRAY_AGG(${tags.name} ORDER BY ${tags.name} ASC)`,
    })
    .from(quotes)
    .leftJoin(authors, eq(quotes.author, authors.id))
    .leftJoin(quotesTags, eq(quotes.id, quotesTags.quoteId))
    .leftJoin(tags, eq(tags.id, quotesTags.tagId))
    .groupBy(quotes.id, authors.id)
    .orderBy(quotes.id)
    .limit(1)
    .offset(index);

  const quoteOfTheDay = response[0];
```

It will calculate how many days since Jan 1 1970.
Based from that, I can calculate an index, fetch the api based on it, get my quote of the day.
This is the simplest way I can implement it.
Main drawback is if I change the date of my computer, the quote will also change. But who cares? 

## Structure
As per the technicals.
- Hosted on AWS Amplify
- Code backend with Amplify, then it will automatically provision resources (like cdk).
- REST Api to expose to public.
- Added a public api key to throttle it to 50 requests/day to all combined users. AWS can get crazy if I'm not careful so I make sure to limit it as I can.
- REST to Lambda Integration to process the main logic.
- Lambda will connect to a Supabase Database where all quotes copied from quotable are stored.
- Route53 for Subdomain of the api and the website. (Although I also set it with Amplify).

## This is the end result btw.
![Quotes API cover](/images/projects/quote.png)



