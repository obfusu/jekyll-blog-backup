---
layout: post
title: DVD Subtitles... ugh!
author: 77ganesh
tags:
---

I stumbled upon an anime that had bitmap based subtitles. These unlike SubRip Text or any other text based subtitles, contain the actual image information for the subtitle text. Though I could care less about the format in which subtitles were stored, the rendered text quality was poor. Additionally I didn't like the its positioning over the video. That's when I thought, why can't we use OCR (Optical Character Recognition) tools like tessaract to convert these to a text based subtitle.

### Step 1: Identify format

Though rare these days, there are a couple of bitmap subtitle formats. We have to find out the exact format. 

```
ffmpeg -i video.mkv
```

A simple media info reveals that it is of "DVD Subtitles" format.

### Step 2: Extract only the subtitles out of the mkv

mkv like mp4 is a container format for storing various combination of streams of audio, video, subtitles, and a lot of other crazy stuff. I tried for a while to extract this using `ffmpeg`, but as I couldn't get the syntax right I went ahead with `mkvextract`

```bash
## Find out subtitle track number using mkvinfo
mkvinfo video.mkv

## Extract it out
mkvextract video.mkv tracks 4:subtitle

## Outputs two files - subtitle.sub & subtitle.idx
```

### R & D

When it comes to media codecs, the licenses are messed up. Most of them are patented and royalty based, meaning you have to pay a heavy fee to use them. This also means that the specification as well blogs are not as many as you would hope to find for popular open source software.

DVD Subtitles as its name suggests is part of the DVD Video specification (I'll call it just DVD from now on).

DVD is a subset of MPEG.

MPEG again has a lot of standards, and for our particular interest, we are interested in MPEG-2 standard which was first released in 1996.

There are two files associated with a "DVD Subtitle" - the idx file & the sub file.

> The idx file is a plain text file containing meta information about the subtitles. It contains details like palette, language, etc.

> The sub file is a MPEG-2 Program Stream (PS) containing Sub Picture Unit (SPU) data as "Private Stream 1". PS is made up of Packetized Elementry Stream (PES) packets.

Private Stream 1 is a type of Elementary Stream (ES), and in our case the ES in sub file (which is a PS) is SPU.

SPU is again a series of SPU packets which with the idx meta info can be used to construct our subtitle's bitmap image.


### Goal #1: Parse and construct this bitmap image out

First we need to extract the ES from the sub file.

```
  ---------------------------------------------------/
 |             |             |             |        /
 | Pack Header | PES header  |  PES data   |       /  .... continues .....
 |             |             |             |      /
  -----------------------------------------------/
```