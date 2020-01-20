---
title: "Overlaying Image in QR"
layout: post
date: 2018-09-15
tag: 
    - QR Code
    - Error Correction
    - ZXing
    - Image Overlay
    - Image Embed
description: "This blog describes how you can overlay an image in your QR Code."
image: ../assets/qr-code.png
headerImage: ../assets/qr-code.png
category: blog
author: pranjal
---

## Who should read this?
If you are into generating QR Codes and want to generate one which includes an image, like your company logo etc, then you are in the right place. This blog is *NOT* focused on explaining what QR Codes are and how they are used and the amount of data they can store. If you need information with that, refere Wikipedia, it has more than sufficient information for you.

If you are still with me, then get ready, open your IDE. We will be using Java and ZXing here.

## Overlay Image in QR Code
These are pretty common types of QR Codes. This technique is specifically targetted at marketting/branding. Looking at these QR Codes, a user can instantly know who this QR belongs to and what he/she is supposed to do with it.

{% for image in site.data.qrimages %}
  <img src="{{image.code}}">
  <figcaption class="caption">Examples of branded QR Codes</figcaption>
{% endfor %}

### Steps to be performed
There are a number of steps to be performed while overlaying an image on a QR code. Like you might have noticed, all the QR Codes shows in the above image are layered i.e. the bottom layer is the actual QR and the top layer is the logo. We have to do the same with ZXing. Let us first see the steps :
1. Create configuration that specifies the error correction
2. Create a QR code as a <code>BitMatrix</code>, using ZXing, with your content
3. Load QR Image into <code>BufferedImage</code>
4. Resize the logo image that you want to overlay and get it in another <code>BufferedImage</code> object
5. Calculate the difference between diamentions of QR Image and the Overlay Image
6. Write the final Image

The following piece of code explains how I wrote the function. There may be many different ways to do this but this is the only one I am aware of. If you know of a different way of doing this, speak up in the comments!

{% gist jalispran/b3a6ab653e4d52981ffe412dd70b4b18 %}

The variables <code>WIDTH</code>, <code>HEIGHT</code>, <code>BLACK</code>, <code>WHITE</code> mark diamentions and  color combination of the QR Code. If you are interested in the values that I used, here you go :
* WIDTH: 150
* HEIGHT: 150
* BLACK: 0xFF000000
* WHITE: 0xFFFFFFFF

And this is how I wrote the <code>getOverlay</code> function.
{% gist jalispran/21cd9ee99440aa6d1f3b2ba5cb15acda %}

In the line 8 and 9, I found out that If I reduced the overlay image by 6 times, it gave me the best results. However, in your case, you may want to play around with this value to see what suits you the best. 

You can just copy the code and replace <code>LOGO</code> and <code>content</code> and it should straightaway work for you. If not, feel free to post queries in the comments.

### Links to resources
1. [Wikipedia](https://en.wikipedia.org/wiki/QR_code)
2. [Google QR Codes](https://developers.google.com/chart/infographics/docs/qr_codes)