# Manual for generating features

### 1. Reference codes and datasets

Codes: https://github.com/vagarwal87/saluki_paper/blob/main/Fig3_S4/generate_training_input.pl &

https://github.com/vagarwal87/saluki_paper/blob/main/Fig3_S4/runme.sh

Datasets: https://zenodo.org/record/6326409

lternatively, you can directly use the data I have downloaded: /mnt/data1/ying/saluki_paper/datasets

### 2. Preparation

Saluki codes (runme.sh in Fig3_S4/):

```
./generate_training_input.pl all_HLs_human_PC1.txt | gzip -c >all_HLs_human_featTable.txt.gz

Rscript calc_kmer_freqs.R
#manually added 1st column name to seqFeatWithKmerFreqs.txt
gzip seqFeatWithKmerFreqs.txt
```

To facilitate the smooth generation of features, please make the modifications as the following instructions.

#### 2.1 Perl codes (generate_training_input.pl)

The headerlines of `generate_training_input.pl`:

```
#!/usr/bin/perl

use allfxns;
```

The allfxns is a perl package from TargetScanTools, please downloaded it and move it to your perl lib (When you run it directly, you will be notified that "allfxns" is not in your lib, and at that point, the location of the lib will be provided.):

```
wget https://github.com/vagarwal87/TargetScanTools/blob/master/allfxns.pm
```

As we typically lack root privileges, we are unable to place this package in the lib of `/usr/bin/perl`. Therefore, please delete the first line: `#!/usr/bin/perl`, and when using it, we will execute it using `perl xxx.pl`. (So please make sure that you have your own perl)

Then, in `allfxns.pm`, please pay attention to the Line 8 where configuring Inline::Python can be quite challenging. However, it's important to note that `generate_training_input.pl` doesn't actually require the Python functionality from `allfxns`. Therefore, please go ahead and directly comment out this line.

![](https://github.com/Liying1996/Generate_features/blob/main/Figs/2_1.jpg)

Alternatively, you can directly copy it from my path: `/home/ying/localperl/lib/site_perl/5.38.0/x86_64-linux/allfxns.pm`

#### 2.2 R codes (calc_kmer_freqs.R)

First, please make sure that you have the following packages:

```
library(data.table)
library(Biostrings)
library(reshape2)
```

Second, the last line:

```
 writefile(inp.tbl,"seqFeatWithKmerFreqs.txt", row.names=T)
```

![](https://github.com/Liying1996/Generate_features/blob/main/Figs/2_2_1.jpg)

You might encounter the issue mentioned above; in this case, you can modify it to:

```
write.table(inp.tbl,"seqFeatWithKmerFreqs.txt", row.names=T, sep="\t", quote=FALSE)
```

Last, but significantly important, if your second column "Halflife" doesn't contain numeric values, it will result in an error in the following code:

```
inp.tbl=inp.tbl[,apply(inp.tbl,2,var)!=0]
```

The reason for this is that when there are no numeric values, the variance calculation results in NA, so the code mentioned above is not boolean variables.. Consequently, it will trigger an "Undefined column" error.

![](https://github.com/Liying1996/Generate_features/blob/main/Figs/2_2_2.jpg)

So, here you must modify the `inp.tbl=inp.tbl[,apply(inp.tbl,2,var)!=0]` to :

```
seleted_col = apply(inp.tbl,2,var)!=0
seleted_col[1]=TRUE
inp.tbl=inp.tbl[,seleted_col!=0]
```

Or, you can directly copy it from my path:  

`/mnt/data1/ying/saluki_paper/us/saluki_data/calc_kmer_freqs.R`



Ps. `#manually added 1st column name to seqFeatWithKmerFreqs.txt` that means the first row is missing "GENEID" at the beginning, so you need to add it manually.

```
sed -i '1s/^/GENEID\t/' seqFeatWithKmerFreqs.txt
```



### 3. Input and Output



`perl generate_training_input.pl all_HLs_human_PC1.txt | gzip -c > all_HLs_human_featTable.txt.gz`

eg.The  `all_HLs_mouse_PC1.txt` file:

![](https://github.com/Liying1996/Generate_features/blob/main/Figs/3.jpg)

**Please note that the file names must include the keywords "Human" or "Mouse".**

Additionally, you will need the following files, which can be downloaded from the datasets provided by the author or copied from my folder (/mnt/data1/ying/saluki_paper/us/saluki_data).
- mm10_ensembl90_3utrs.fa.gz (mouse)
- mm10_ensembl90_5utrs.fa.gz (mouse)
- mm10_ensembl90_orfs.fa.gz (mouse)
- Mus_musculus.GRCm3.90.chosenTranscript.gtf.gz (mouse)

The third column "PC1" is the "Halflife" in `all_HLs_human_featTable.txt.gz` actually. **You can also provide only the first two columns. The first two columns are gene names, but in the result, the "Halflife" column will be empty. At this point, you can provide your own list of genes, as needed.**

`Rscript calc_kmer_freqs.R`

Please note that during execution, "all_HLs_human_featTable.txt.gz" must be present in the current directory. Also, please be aware that this file is set to read human data by default. If you intend to work with mouse data, make sure to modify the script's line 7 accordingly.

Upon completion, a file named "seqFeatWithKmerFreqs.txt" will be generated.


In conclusion, 
```
# Input: Gene list; First col - ensembl ID; Second: gene symbol; (Please note that the input file name must have the keyword: "Human" or "Mouse")
perl generate_training_input.pl all_HLs_human_PC1.txt | gzip -c > all_HLs_human_featTable.txt.gz 
# Input: all_HLs_human_featTable.txt.gz (Or other file names, please modify the calc_kmer_freqs.R) 
Rscript calc_kmer_freqs.R # Output: seqFeatWithKmerFreqs.txt
sed -i '1s/^/GENEID\t/' seqFeatWithKmerFreqs.txt
```
