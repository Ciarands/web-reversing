# Defeating web-application obfuscation
This guide is aimed at individuals with intermediate web literacy, you are expected to understand certain fundemental concepts, which will not be explained within the guide.
All techniques shown will be framed from the perspective of web-scraping, if you are not familiar with this field I would suggest Blatzar's [scraping-tutorial](https://github.com/Blatzar/scraping-tutorial/tree/master), this is a perfect starting point.

While not completely necessary a good understanding of Javascript/Typescript will also help.

## Pre-requisites 
- Node.js
- Basic Regex
- Understanding of some cryptography concepts

## What is obfuscation?
`noun` obfuscation 
> "The act or process of obfuscating, or obscuring the perception of something; the concept of concealing the meaning of a communication by making it more confusing and harder to interpret. "

In terms of programming; obfuscation is used to refer to the act of intentionally making code very difficult for humans to interpret and mentally parse, whilst still being interpretable by machines.

## Obfuscation is everywhere
Obfuscation, can be seen everywhere, from desktop applications using Themeda to web applications using obfuscator.io.
These techniques are used to intentionally make anaylsis incredibly difficult, this is highly problematic for security research, privacy analysis and debugging, and for control over your own device in general.

## Support
Join the [discord](https://discord.gg/z2r8e8neQ7)