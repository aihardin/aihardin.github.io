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

```R
source("http://bioconductor.org/biocLite.R")
biocLite("SRAdb")
library(SRAdb)
sqlfile <- getSRAdbFile()
sra_con <- dbConnect(SQLite(),sqlfile)
```

We can get information about each run by looking in the run table:

```R
run <- dbGetQuery(sra_con, "select * from run where run_accession = 'SRR1029856'")
```

| RUN_ID | BAMFILE | RUN_ALIAS     | RUN_ACCESSION | BROKER_NAME | INSTRUMENT_NAME | RUN_DATE      | RUN_FILE | RUN_CENTER | TOTAL_DATA_BLOCKS | EXPERIMENT_ACCESSION | EXPERIMENT_NAME | SRA_LINK | RUN_URL_LINK | XREF_LINK | RUN_ENTREZ_LINK | DDBJ_LINK | ENA_LINK | RUN_ATTRIBUTE | SUBMISSION_ACCESSION | SRADB_UPDATED |
|--------|---------|---------------|---------------|-------------|-----------------|---------------|----------|------------|-------------------|----------------------|-----------------|----------|--------------|-----------|-----------------|-----------|----------|---------------|----------------------|---------------|
| 611310 | NA      | GSM1264352_r1 | SRR1029856    | NA          | NA              | 11/14/13 0:00 | NA       | GEO        | NA                | SRX377662            | GSM1264352      | NA       | NA           | NA        | NA              | NA        | NA       | NA            | SRA111212            | 6/10/14 12:20 |

This doesn’t tell us very much but it does contain the crucial field that lets us know what experiment it came from: SRX377662.

Using this information, we can look in the experiment table with:

```R
exp <- dbGetQuery(sra_con, "select * from experiment where experiment_accession = 'SRX377662'")
```

| EXPERIMENT_ID | BAMFILE | FASTQFTP | EXPERIMENT_ALIAS | EXPERIMENT_ACCESSION | BROKER_NAME | CENTER_NAME | TITLE                                                            | STUDY_NAME | STUDY_ACCESSION | DESIGN_DESCRIPTION | SAMPLE_NAME | SAMPLE_ACCESSION | SAMPLE_MEMBER | LIBRARY_NAME | LIBRARY_STRATEGY | LIBRARY_SOURCE | LIBRARY_SELECTION | LIBRARY_LAYOUT | TARGETED_LOCI | LIBRARY_CONSTRUCTION_PROTOCOL                                                                                                                                                                                                                                                           | SPOT_LENGTH | ADAPTER_SPEC | READ_SPEC | PLATFORM | INSTRUMENT_MODEL    | PLATFORM_PARAMETERS                   | SEQUENCE_SPACE | BASE_CALLER | QUALITY_SCORER | NUMBER_OF_LEVELS | MULTIPLIER | QTYPE | SRA_LINK | EXPERIMENT_URL_LINK                                                                 | XREF_LINK      | EXPERIMENT_ENTREZ_LINK | DDBJ_LINK | ENA_LINK | EXPERIMENT_ATTRIBUTE      | SUBMISSION_ACCESSION | SRADB_UPDATED |
|---------------|---------|----------|------------------|----------------------|-------------|-------------|------------------------------------------------------------------|------------|-----------------|--------------------|-------------|------------------|---------------|--------------|------------------|----------------|-------------------|----------------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|--------------|-----------|----------|---------------------|---------------------------------------|----------------|-------------|----------------|------------------|------------|-------|----------|-------------------------------------------------------------------------------------|----------------|------------------------|-----------|----------|---------------------------|----------------------|---------------|
| 430097        | NA      | NA       | GSM1264352       | SRX377662            | NA          | GEO         | GSM1264352: Forebrain E11.5 H3K27ac Rep1; Mus musculus; ChIP-Seq | GSE52386   | SRP033009       | NA                 | NA          | SRS502469        | NA            | NA           | ChIP-Seq         | GENOMIC        | ChIP              | SINGLE -       | NA            | Tissues were cross-linked, lysed, and sonicated and immunoprecipation was performed using H3K27ac (Abcam Ab4729). Input samples represent sonicated DNA before immunoprecipitation Libraries were constructed using standard protocols and the Illumina TruSeq kit with index barcodes. | NA          | NA           | NA        | ILLUMINA | Illumina HiSeq 2000 | INSTRUMENT_MODEL: Illumina HiSeq 2000 | NA             | NA          | NA             | NA               | NA         | NA    | NA       | GEO Sample GSM1264352: http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM1264352 | gds: 301264352 | NA                     | NA        | NA       | GEO Accession: GSM1264352 | SRA111212            | 6/8/14 22:14  |

This has the title info which we need to make sense of the data: what tissue and when it was collected.

Now we can use the power of SQL and get all the data at once by joining the run and experiment tables and only getting the rows that match the study ID.

```R
runs <- dbGetQuery(sra_con, "select run_accession, title from run JOIN experiment USING (experiment_accession) where experiment.study_accession = 'SRP033009'")
```

| RUN_ACCESSION | TITLE                                                            |
|---------------|------------------------------------------------------------------|
| SRR1029856    | GSM1264352: Forebrain E11.5 H3K27ac Rep1; Mus musculus; ChIP-Seq |
| SRR1029857    | GSM1264353: Forebrain E11.5 input Rep1; Mus musculus; ChIP-Seq   |
| SRR1029858    | GSM1264354: Forebrain E11.5 H3K27ac Rep2; Mus musculus; ChIP-Seq |
| SRR1029859    | GSM1264355: Forebrain E11.5 input Rep2; Mus musculus; ChIP-Seq   |
| SRR1029860    | GSM1264356: Forebrain E14.5 H3K27ac; Mus musculus; ChIP-Seq      |
| SRR1029861    | GSM1264357: Forebrain E14.5 input; Mus musculus; ChIP-Seq        |
| SRR1029862    | GSM1264358: Forebrain E17.5 H3K27ac; Mus musculus; ChIP-Seq      |
| SRR1029863    | GSM1264359: Forebrain E17.5 input; Mus musculus; ChIP-Seq        |
| SRR1029864    | GSM1264360: Forebrain P0 H3K27ac; Mus musculus; ChIP-Seq         |
| SRR1029865    | GSM1264361: Forebrain P0 input; Mus musculus; ChIP-Seq           |
| SRR1029866    | GSM1264362: Forebrain P7 H3K27ac; Mus musculus; ChIP-Seq         |
| SRR1029867    | GSM1264363: Forebrain P7 input; Mus musculus; ChIP-Seq           |
| SRR1029868    | GSM1264364: Forebrain P21 H3K27ac; Mus musculus; ChIP-Seq        |
| SRR1029869    | GSM1264365: Forebrain P21 input; Mus musculus; ChIP-Seq          |
| SRR1029870    | GSM1264366: Forebrain P56 H3K27ac Rep1; Mus musculus; ChIP-Seq   |
| SRR1029871    | GSM1264367: Forebrain P56 input Rep1; Mus musculus; ChIP-Seq     |
| SRR1029872    | GSM1264368: Forebrain P56 H3K27ac Rep2; Mus musculus; ChIP-Seq   |
| SRR1029873    | GSM1264369: Forebrain P56 input Rep2; Mus musculus; ChIP-Seq     |
| SRR1029874    | GSM1264370: Heart E11.5 H3K27ac Rep1; Mus musculus; ChIP-Seq     |
| SRR1029875    | GSM1264371: Heart E11.5 input Rep1; Mus musculus; ChIP-Seq       |
| SRR1029876    | GSM1264372: Heart E11.5 H3K27ac Rep2; Mus musculus; ChIP-Seq     |
| SRR1029877    | GSM1264373: Heart E11.5 input Rep2; Mus musculus; ChIP-Seq       |
| SRR1029878    | GSM1264374: Heart E14.5 H3K27ac; Mus musculus; ChIP-Seq          |
| SRR1029879    | GSM1264375: Heart E14.5 input; Mus musculus; ChIP-Seq            |
| SRR1029880    | GSM1264376: Heart E17.5 H3K27ac; Mus musculus; ChIP-Seq          |
| SRR1029881    | GSM1264377: Heart E17.5 input; Mus musculus; ChIP-Seq            |
| SRR1029882    | GSM1264378: Heart P0 H3K27ac; Mus musculus; ChIP-Seq             |
| SRR1029883    | GSM1264379: Heart P0 input; Mus musculus; ChIP-Seq               |
| SRR1029884    | GSM1264380: Heart P7 H3K27ac; Mus musculus; ChIP-Seq             |
| SRR1029885    | GSM1264381: Heart P7 input; Mus musculus; ChIP-Seq               |
| SRR1029886    | GSM1264382: Heart P21 H3K27ac; Mus musculus; ChIP-Seq            |
| SRR1029887    | GSM1264383: Heart P21 input; Mus musculus; ChIP-Seq              |
| SRR1029888    | GSM1264384: Heart P56 H3K27ac; Mus musculus; ChIP-Seq            |
| SRR1029889    | GSM1264385: Heart P56 input; Mus musculus; ChIP-Seq              |
| SRR1029890    | GSM1264386: Liver E11.5 H3K27ac; Mus musculus; ChIP-Seq          |
| SRR1029891    | GSM1264387: Liver E11.5 input; Mus musculus; ChIP-Seq            |
| SRR1029892    | GSM1264388: Liver E14.5 H3K27ac; Mus musculus; ChIP-Seq          |
| SRR1029893    | GSM1264389: Liver E14.5 input; Mus musculus; ChIP-Seq            |
| SRR1029894    | GSM1264390: Liver E17.5 H3K27ac; Mus musculus; ChIP-Seq          |
| SRR1029895    | GSM1264391: Liver E17.5 input; Mus musculus; ChIP-Seq            |
| SRR1029896    | GSM1264392: Liver P0 H3K27ac; Mus musculus; ChIP-Seq             |
| SRR1029897    | GSM1264393: Liver P0 input; Mus musculus; ChIP-Seq               |
| SRR1029898    | GSM1264394: Liver P7 H3K27ac; Mus musculus; ChIP-Seq             |
| SRR1029899    | GSM1264395: Liver P7 input; Mus musculus; ChIP-Seq               |
| SRR1029900    | GSM1264396: Liver P21 H3K27ac; Mus musculus; ChIP-Seq            |
| SRR1029901    | GSM1264397: Liver P21 input; Mus musculus; ChIP-Seq              |
| SRR1029902    | GSM1264398: Liver P56 H3K27ac; Mus musculus; ChIP-Seq            |
| SRR1029903    | GSM1264399: Liver P56 input; Mus musculus; ChIP-Seq              |
