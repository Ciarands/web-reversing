# Defeating web-application obfuscation

___You will need to have a fluent understanding of web concepts in order to follow along with the subjects explored in this guide.___

This guide is aimed at individuals with intermediate to expert literacy when it comes to the web, you are expected to understand certain fundemental concepts, these concepts will not be explained within the guide.
All techniques shown will be framed from the perspective of web-scraping, if you are not familiar with this field of work I would suggest Blatzar's [scraping-tutorial](https://github.com/Blatzar/scraping-tutorial/tree/master), this is a perfect starting point.

While not completely necessary a good understanding of Javascript/Typescript will also help.

## Pre-requisites 

- Node.js
- A basic understanding of cryptography concepts
- Some brain cells (I dont have that many dw)

## What is obfuscation?
`noun` obfuscation 
> "The act or process of obfuscating, or obscuring the perception of something; the concept of concealing the meaning of a communication by making it more confusing and harder to interpret. "

In terms of programming; obfuscation is used to refer to the act of intentionally making code very difficult for humans to interpret and mentally parse, whilst still being interpretable by machines.
Obfuscation is a security through obscurity (STO) practice which is widely criticised by security researchers, however it is the foundation of most protections used in the digital sphere, to some degree of success.

## Obfuscation is everywhere
Obfuscation, otherwise commonly refered to as "packing" can be seen everywhere, from desktop applications using Themeda to web applications using Dean Edward's packer.
These techniques are used to intentionally make anaylsis incredibly difficult, this is highly problematic for security research, privacy analysis and debugging, and for control over your own device in general.
Your computer should not be a black box in which data goes in and out, you should be able to see how your device interfaces with with a website, protections like obfuscation make this very difficult.
