---
author: aihardin
comments: true
date: 2014-08-05 19:38:55+00:00
layout: post
slug: getting-sequencing-metadata-from-ncbi-sra
title: Getting sequencing metadata from NCBI SRA
wordpress_id: 147
categories:
- benchwork
- bioinformatics
tags:
- bioconductor
- fastq
- R
- SRAdb
---

The GEO page [http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE52386](http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE52386) has a link to the SRA study ftp page which you can use to get all the sequencing reads with wget:

    
     wget -r ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByStudy/sra/SRP/SRP033/SRP033009/


After that, the fast-dump utility from the sra-toolkit can be used to convert the files to gripped fastq files, ready to be mapped with Bowtie, BWA, or other mappers.  However, the file names are uninformative!  Is SRR1029869.fastq.gz a sample of the brain or liver? Is this embryonic or postnatal?

To get the metadata for a study we can use the R package [SRAdb](http://www.bioconductor.org/packages/release/bioc/html/SRAdb.html)  and a bit of simple SQL.

Set up by installing SRAdb with Bioconductor and downloading the SRA metainfo database.  This is a large file > 7 GB so keep this in mind for download times and space on your computer.

[code language="r"]
source("http://bioconductor.org/biocLite.R")
biocLite("SRAdb")
library(SRAdb)
sqlfile <- getSRAdbFile()
sra_con <- dbConnect(SQLite(),sqlfile)
[/code]

We can get information about each run by looking in the run table:
[code language="r"]run <- dbGetQuery(sra_con, "select * from run where run_accession = 'SRR1029856’")[/code]

[table id=1 /]

This doesn’t tell us very much but it does contain the crucial field that lets us know what experiment it came from: SRX377662.

Using this information, we can look in the experiment table with:

[code language="r"]exp <- dbGetQuery(sra_con, "select * from experiment where experiment_accession = 'SRX377662’")[/code]

[table id=2 /]

This has the title info which we need to make sense of the data: what tissue and when it was collected.

Now we can use the power of SQL and get all the data at once by joining the run and experiment tables and only getting the rows that match the study ID.

[code language="r"]runs <- dbGetQuery(sra_con, "select run_accession, title from run JOIN experiment USING (experiment_accession) where experiment.study_accession = 'SRP033009’”)[/code]

[table id=3 /]
