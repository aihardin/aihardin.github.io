---
author: aihardin
comments: true
date: 2013-06-20 18:52:59+00:00
layout: post
slug: the-melanogaster-iceberg
title: The melanogaster iceberg
wordpress_id: 40
categories:
- research blogging
tags:
- 1KG
- allele frequency
- d3
- DGRP
- DPGP2
- javascript
---

One thing that I like about studying fruit flies is that as a species, they share a similar history with humans.  Both _D. melanogaster_ and _H. sapiens_ diverged from their common ancestors while in Africa and have since undergone a population explosion and spread across the world within tens of thousands of years ago.  I don't know how many other species share this pattern but I would guess that anything that lives off of human byproducts is similar: mice, rats, lice.

In the past, fruit flies have taught us many things about genetic mechanisms starting with the first link of chromosomes to genetics in the pioneering work of Thomas Hunt Morgan in the early 1900s.  Once DNA was understood and scientists were able to read the bases, fruit flies were the first organisms to have a gene sequenced at a global scale (eleven samples but a huge amount of work at the time).  This trend of a model organism informing us on the natural processes that we expect to be occurring in humans has been immensely useful. Viruses, bacteria, yeast, fruit flies and a plant all had their genomes sequenced before a human sequence was finished. However, once DNA sequencing and other technologies became cheap enough, scientists spent a huge amount of time and money to characterize the amount of human genetic diversity. First by interrogating diversity at specific nucleotides with microarrays with the HapMap Project and then supplementing with whole genome sequencing in the 1000 Genomes Project we quickly amassed more data on humans than any other species.  Now the trend has flipped and we can apply information we have gathered on _H. sapiens_ to _D. melanogaster_.

One example of a modern human study that supasses anything done in flies is the joint [work](http://www.ncbi.nlm.nih.gov/pubmed/22604722) of the [Novembre](http://jnpopgen.org/) lab, then at UCLA now at U of Chicago, and the pharmaceutical company GlaxoSmithKline in sequencing 202 genes from 14,002 people.  I'm not sure what the maximum gene sequencing depth we have on _D. melanogaster_ but it is nothing close to 14,000.  Maybe a thousand? In any case, the reason that this data set is interesting is that we do not know how much genetic diversity is in the human or fly population.  We do know that both populations are growing at enormous speed historically speaking and based on that fact we can predict there will be a large amount of new and rare variants in the population.  This follows from realizing that each new individual inherits variants from their parents and also through various mechanisms, creates new mutations in their genomes.  In an expanding population, most people will not have had time for their new alleles to spread and will therefore be rare.  These rare alleles are potentially very relevant to human health since purifying selection has not had time to purge them from the population.  The idea that these new deleterious mutations increases the _average_ genetic burden has been [challenged](http://haldanessieve.org/2013/05/10/the-deleterious-mutation-load-is-insensitive-to-recent-population-history/) recently by Jonathan Pritchard and Guy Sella's labs but the total number of deleterious alleles in the population still goes up.

Using exon capture methods followed by high throughput sequencing, the Novembre lab were able to find alleles that were present at frequencies as low as 0.007%.  I attended a presentation Novembre gave about this work and he described Fig 1B as an "iceberg plot" meaning that the previous studies only saw the tip of human genetic variation.

![](http://newsroom.ucla.edu/portal/ucla/artwork/3/4/0/7/4/234074/John_Novembre_2012_iceberg_image.jpg) The human genetic variation iceberg. Variants that could be found in samples with up to 200 individuals shown on top and everything else below.

The rest of their analysis goes on the show that the rarest of variants are more likely to be damaging and that the iceberg goes down even farther than this, particularly in the African populations.

I'm quite curious if this same pattern exists in the global _D. melanogaster_ population.  I don't have 14 thousand flies to sequence but I do have the genomes from the [Drosophila Genetic Reference Panel](http://dgrp.gnets.ncsu.edu/), the [Drosophila Population Genomics Project](http://www.dpgp.org/dpgp2/DPGP2.html) and some [prerelease](http://bergmanlab.smith.man.ac.uk/?p=1685) genomes from Casey Bergman and Penny Haddrill for some 325 fully sequenced genomes.  Baring line mixups and all kinds of mistakes I could have made, #overlyhonestmethods, I should be able to see some excess of rare variants since the DPGP genomes are mainly African and have higher diversity. I'm not going to make as fancy of a figure as Novembre did since this is also an exercise in learning D3.

Method: align all raw sequence data to the reference genome with BWA/Stampy.  Realign mapped reads and call variants with GATK.  Recalibrate the variant quality with snps called from four DGRP genomes resequenced to 50x coverage on a HighSeq by me (well, really resequenced by the lab's awesome and talented tech who wishes to remain behind the scenes for now). Filter out false positives and non-biallelic snps. The second filter is going to decrease the number of rare alleles found but this is quick and dirty and it makes the partitioning easy. Then use GATK and bedtools to find how many segregating sites are in each exon:

    
    java -jar ~/bin/GenomeAnalysisTK-2-latest/GenomeAnalysisTK.jar -R ~/data/genome/dm3.fasta -T SelectVariants --variant global_snps.recalibrated.sep.pass.vcf.gz --restrictAllelesTo BIALLELIC -select "AF >= 0.005" --logging_level ERROR|bedtools intersect -sorted -a ~/data/annotation/dmel_gene.gff -b stdin -c > all_highfreq_gene_counts.bed


A bit of python to turn the bedtools outputs into a proper GFF file:

    
    low_fh = file('all_lowfreq_gene_counts.bed')
    high_fh = file('all_highfreq_gene_counts.bed')
    out_fh = file('all_gene_mut_counts.bed','w')
    out_fh.write('##gff-version 3\n')
    for line in high_fh:
        low_counts = low_fh.readline().strip().split('\t')[-1]
        exon_data = line.strip().split('\t')[:-1]
        high_counts = line.strip().split('\t')[-1]
        exon_data[-1] = exon_data[-1]+';high_count=%s;low_count=%s'%(high_counts,low_counts)
        out_fh.write('\t'.join(exon_data)+'\n')
    out_fh.close()


And some more to process the GFF and get counts:

    
    input_bed = file('all_gene_mut_counts.bed')
    exons = {}
    for line in input_bed:
        if line.startswith('##'): continue
        try:
            annotation = line.rstrip().split('\t')[-1].split(';')
            annotation = dict([tuple(a.split('=')) for a in annotation if '=' in a]) #skips the flag values but don't need them
        except ValueError:
            print line, annotation
    #print annotation
    exons[annotation['ID']] = (int(annotation['high_count']),int(annotation['low_count']))
    mut_counts = zeros((3,len(exons)))
    
    for i,exon in enumerate(exons):
        mut_counts[0][i] = exons[exon][0]
        mut_counts[1][i] = exons[exon][1]
    mut_counts[2]=mut_counts[0]+mut_counts[1]
    mut_counts.sort(axis=1)
    out_fh = file('data.js','w')
    out_fh.write('var high_freq = ['+','.join(map(str,mut_counts[0][-200:]))+']\n')
    out_fh.write('var low_freq = ['+','.join(map(str,mut_counts[1][-200:]))+']\n')
    out_fh.close()


Now I have the top 200 genes with the most segregating sites in them, sorted from least to most.  To replicate the broad strokes of the iceberg plot, I wanted to use D3 and generate SVG files. I haven't used javascript ever and it has been 8 years since I coded HTML so it took a bit but I managed to make some bar graphs:
<iframe style="border: 0px;" src="{{ "/plots/iceberg.html" | prepend: site.baseurl }}" height="400" width="800" scrolling="no"></iframe>
We see with 325 genomes the hints of an iceberg! Variants seen at a frequency of less than 0.05% in this dataset means the singletons, the variants seen in only one of the genomes. I'm sure that with more genome sequences the _D. melanogaster_ plot will look more like the _H. sapiens_. Other projects are in the works to describe genetic variation in other model organisms such as the plant _[Arabidopsis thaliana](http://en.wikipedia.org/wiki/Arabidopsis_thaliana)_ and the worm _[Caenorhabditis elegans](http://en.wikipedia.org/wiki/Caenorhabditis_elegans)_.

In working through coding and presenting the D3, I leaned heavily on the D3 tutorial by [Scott Murray](http://alignedleft.com/tutorials/d3/) and the tips of [Charles Reid](http://charlesmartinreid.com/wordpress/2012/08/d3-and-wordpress/).
