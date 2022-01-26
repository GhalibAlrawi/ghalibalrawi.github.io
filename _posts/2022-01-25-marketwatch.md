---
layout: post
title: MarketWatch
---

### Background/Purpose

I started repairing phones around the start of Grade nine. I would look over local markets like Facebook Marketplace, Kijiji, or Craigslist, and see if I could find a phone I could buy, repair, and sell for a profit. This eventually expanded to electronics repair in general and flipping whatever I could find, but the majority of money I made still came from phone repair.

[//]: <> Finding a good deal before everyone else was a huge advantage as they would usually be gone quite fast. I realized that this can be automated using my programming knowledge. I could use an IPO (Input, Process, Output) approach to search for available listings on the market using Kijiji’s API, filter for new listings that have appeared since last search, and if they were listed for a good price, I’d make my program notify me through a sound and console output. I used several software engineering techniques such as Dependency Injection (DI), Object Oriented Programming (OOP), and Serialization/Deserialization. I’ve also expanded the project to add automatic listing reposting and a graphical user interface (still in development).

### Development tools, Frameworks, etc
This project was made in C#/.NET Core, with Visual Studio 2019 as my IDE.

## Code snippets
### API Implementations
There was a publicly available GitHub repository that allowed me to fetch ads from Kijiji. The repo is written in Javascript, so I couldn’t import the code directly into my application and run it. I chose to create a Javascript script for which I could run through a terminal with any additional arguments it would need (i.e., query string) also supplied through the terminal. After completion, the script would produce a JSON string outputted through a file either indicating a successful request and the retrieved ads or a failed request and error information:

Successful request:
```javascript
{
  "success": true,
  "ads":[...]
}
```

Failed request:
```javascript
{
  "success": false,
  "exception": "403 error encountered"
}
```

{C# code running the script}

In hindsight, a more elegant approach could have been implemented using interprocess communication via pipe. This would allow for more direct dataflow between the two processes (the script, and my C# program).

