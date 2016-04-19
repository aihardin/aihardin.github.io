---
author: aihardin
comments: true
date: 2013-08-27 23:11:48+00:00
layout: post
slug: making-an-organism-out-of-a-genome-with-python
title: Making an organism out of a genome with python
wordpress_id: 88
tags:
- drosophila
- python
---

Genomes are long strands of nucleic acids consisting of the instruction guide for the assembly of an organism. Once sequencing projects are completed we conceptualize genome as strings of text for convenience instead of huge chemical diagrams. There are really powerful techniques that researchers can use to manipulate and study these long strings of 'ATGC' but sometimes I feel like these abstractions lose a bit of the linkage of the genome to the organism that it came from.  If I was given a file named GRCh37.p13.fasta and nothing else, it would take a bit of work just to figure out that this is from a human.

How can we identify an organism just by looking at the sequence?  I don't know of a method but one you know what it is, how can you use it to represent the organism? One cool way from Peter Cock at [blastedbio](http://blastedbio.blogspot.com/2013/08/pixelated-potato-posters-in-python.html) is to turn a bit of the genome into a pixelated picture of the organism.  The conceptual steps are to take the colors from a region of a picture and write a letter from the genome in that color. He used it for making pictures of the potato genome but anyone can take their favorite genome sequenced organism and do the same thing!  Peter wrote this so that the output is a series of pdfs of various sizes for your poster making needs. For convenience, I've made his script a bit more general and posted it here:[ https://github.com/aihardin/picobio/blob/master/acgt_dither/dither_rgb.py ](https://github.com/aihardin/picobio/blob/master/acgt_dither/dither_rgb.py) To use this, first make sure you have BioPython, numpy, Python Image Library and reportlab installed.  Then get a [picture](http://en.wikipedia.org/wiki/File:Drosophila_melanogaster_-_side_(aka).jpg) from wikipedia and a [genome](ftp://ftp.flybase.net/genomes/Drosophila_melanogaster/current/fasta/dmel-all-chromosome-r5.52.fasta.gz) from flybase and run it like so:
python [dither_rbg.py](https://github.com/aihardin/picobio/blob/master/acgt_dither/dither_rgb.py) [dmel_wiki.jpg](http://en.wikipedia.org/wiki/File:Drosophila_melanogaster_-_side_(aka).jpg) [dm3.fasta](ftp://ftp.flybase.net/genomes/Drosophila_melanogaster/current/fasta/dmel-all-chromosome-r5.52.fasta.gz)

![dmel_wiki_bg_A0]({{ site.url }}/assets/dmel_wiki_bg_A0.png)

I've resized this image so it is a reasonable size for my hosting limits. A side effect is that the individual characters are no longer visible and it looks more like the original image.  While this is a cool compression artifact, it loses the intent of the program. Here is a sample of the full size image: ![close_up_atcg]({{ site.url }}/assets/close_up_atcg.png)

While one fly is awesome, I enjoy the evolutionary perspective and using these fantastic [illustrations](http://flybase.org/static_pages/anatomy/Drosophilidae/Drosophilidae.html) by Mrs. Sarah L. Martin provided by FlyBase and this [list](http://bergmanlab.smith.man.ac.uk/?p=1612) of sequenced Drosophilids curated by Casey Bergman, I created pictures of all available fly genomes below.  Warning, these are rather large PDFs so don't open in your browser.

[Drosophila_ananassae_A0]({{ site.url }}/assets/Drosophila_ananassae_A0.pdf)

[Drosophila_bipunctata_A0]({{ site.url }}/assets/Drosophila_bipunctata_A0.pdf)

[Drosophila_mojavensis_A0]({{ site.url }}/assets/Drosophila_mojavensis_A0.pdf)

[Drosophila_melanogaster_A0]({{ site.url }}/assets/Drosophila_melanogaster_A0.pdf)

[Drosophila_pseudoobscura_A0]({{ site.url }}/assets/Drosophila_pseudoobscura_A0.pdf)

[Drosophila_virilis_A0]({{ site.url }}/assets/Drosophila_virilis_A0.pdf)

[Drosophila_willistoni_A0]({{ site.url }}/assets/Drosophila_willistoni_A0.pdf)
