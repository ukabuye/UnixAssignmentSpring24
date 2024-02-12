# Usamah Kabuye's unix assignment | BCB 546 | Spring 2024

## Data inspection

### Exploring the attributes of the fang_et_al_genotypes.txt data

1. I used the du (disk usage) command with the -h (human readable) option to explore the size of the fang_et_al_genotypes.txt data file. 

2. I used the command 'file' to determine the type of file (i.e. file format or encoding) that the "fang_et_al_genotypes.txt" was--based on its content.

3. I used the wc (word count) command explore the number of rows aka newline, word and byte aka character count of the fang_et_al_genotypes.txt data.

4. I used the grep command and piped it with the awk command to determine the number of columns of the fang_et_al_genotypes.txt file. 

5. I used the grep command piped with the cut, sort and uniq commands to get the respective group counts for both maize and teosinte, sorted by count in ascending order.

Below are the (snippets of the) command syntaxes I used for inspecting the fang_et_al_genotypes.txt data file

```
$ du -h fang_et_al_genotypes.txt # file size

$ file fang_et_al_genotypes.txt # file formart or encoding (ASCII text)

$ wc fang_et_al_genotypes.txt # number of rows aka newline, word and byte aka character count 

$ grep -v "^#" fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}' # number of columns

$ grep -v "^#" fang_et_al_genotypes.txt | cut -f3 | sort | uniq -c | sort -n # levels of maize and teosinte in the third column
```
By inspecting the fang_et_al_genotypes.txt data file, I learned that:

1. It was 6.7 Mbs in size

2. It was encoded using ASCII characters (i.e. human-readable ASCII text) with very long lines

3. It had 2783 rows, 2744038 words and 11051939 bytes aka characters

4. It had 986 columns

5. The counts for the groups in maize were 27 (ZMMMR), 290 (ZMMIL), 1256 (ZMMLR) while the group counts for teosinite were 34 (ZMPJA), 41 (ZMPIL), 900 (ZMPBA)

### Exploring the attributes of the snp_position.txt data

1. I used the du (disk usage) command with the -h (human readable) option to explore the size of the snp_position.txt data file. 

2. I used the command 'file' to determine the type of file (i.e. file format or encoding) that the "snp_position.txt" was--based on its content.

3. I used the wc (word count) command explore the number of rows aka newline, word and byte aka character count of the snp_position.txt data.

4. I used the grep command and piped it with the awk command to determine the number of columns of the snp_position.txt file. 

5. I used the grep command piped with the cut, sort and uniq commands to inspect the number of chromosomes and corresponding count from the third column of the `snp_position.txt` data file.


Below are the (snippets of the) command syntaxes I used for inspecting the snp_position.txt data file

```
$ du -h snp_position.txt # file size

$ file snp_position.txt # file formart or encoding (ASCII text)

$ wc snp_position.txt # number of rows aka newline, word and byte aka character count 

$ grep -v "^#" snp_position.txt | awk -F "\t" '{print NF; exit}' # number of columns

$ grep -v "^#" snp_position.txt | cut -f3 | sort | uniq -c | sort -n # Chromosome number and SNPs
```
By inspecting the snp_position.txt data file, I learned that:

1. It was 49Kb in size

2. It was encoded using ASCII characters (i.e. human-readable ASCII text)

3. It had 984 rows, 13198 words and 82763 bytes aka characters

4. It had 15 columns

5. There were 10 chromosomes. Chromosome 1 had the highest (155) SNPs while chromosome 10 had the least (53) SNPs. Additionally, there were 27 SNPs with unknown chromosome location while 6 SNPs appeared on multiple chromosomes

# Data processing

## SNP position data file

I removed the second column that had the **cdv_marker_id** in order to replace it with the **Chromosome** column. I saved its header in a separate file and created another file with the `snp_position` file that did not have headers. I sorted this unheaded file and echoed it to confirm the sorting. The "0" standard out indicated that it had been successfully sorted. Below is the snippet of the commands I used.

```
$ cut -f1,3,4,5,6,7,8,9,10,11,12,13,14,15 snp_position.txt > snp_less_cdv.txt
$ head -n 1 snp_less_cdv.txt > headers_snp_position.txt
$ tail -n +2 snp_less_cdv.txt > unheaded_snp.txt
$ sort -k1,1 unheaded_snp.txt > sorted_unheaded_snp.txt
$ sort -k1,1 -c sorted_unheaded_snp.txt
$ echo $?
```
## Processing maize data files

I first created a separate file having headers of the fang_et_al_genotypes.txt.

```
$ head -n 1 fang_et_al_genotypes.txt > headers_fang.txt
```

I then separated the "ZMM" data from the `fang_et_al_genotypes.txt` into a separate file, attached headers to it and then transposed it. 

```
$ grep 'ZMM' fang_et_al_genotypes.txt > maize_data.txt
$ cat headers_fang.txt maize_data.txt > headed_maize.txt
$ awk -f transpose.awk headed_maize.txt > transposed_maize.txt
```
I then went on to create a transposed maize file that doesnt have headers so that I could sort it. I sorted this and it was ready for merging with the `snp-file`. I confirmed that it had been merged by checking on its number of rows with the `wc -ls` command and also echoed it to comfirm it had been successfully sorted. The "0" echo confirmed it had been sorted while the `wc -l` giving me 983 rows, which was the same number of rows for the snp-file confirmed it had been transposed and ready for merging.

```
$ tail -n +4 transposed_maize.txt > unheaded_transposed_maize.txt
$ sort -k1,1 unheaded_transposed_maize.txt > sorted_unheaded_maize.txt
$ sort -k1,1 -c sorted_unheaded_maize.txt
$ echo $?
$ wc -l sorted_unheaded_maize.txt
```
I then merged the snp-file with this maize file to form one composite data file and attached snp-headers to ensure that the joint file had SNP ID, Chromosome and SNP position in the first, second and third columns respectively. I confirmed this cutting the first 4 columns. I explored the merged file to ascertain successful file joining by grepping the number of columns it had, which must be the sum of the columns of the two separate file less by one (the common SNP_ID column). 

```
$ join -1 1 -2 1 -t $'\t' sorted_unheaded_snp.txt sorted_unheaded_maize.txt > joined_maize.txt
$ cut -f1,2,3,4 joined_maize.txt | head -n 4
$ tail -n +5 joined_maize.txt | awk -F "\t" '{print NF; exit}'
```
I then attached headers to this file

```
$ cat headers_snp_position.txt joined_maize.txt > head_joined_maize.txt
```
## Processing the 22 snp files for maize

### Maize SNPs ordered according to increasing position values
 
I loaded and used the `bioawk` command with the `-c hdr` that identified column named "Chromosome", selected values equal to the chromosome number and then piped the results for sorting according to column 3, by treating them as numeric.
 
At this point, I ignored the missing values in the original file that had been recorded as "?". I then sent the result in a separate file I named according to the respective chromosome. This was repeated for all the 10 chromosomes.
 
```
$ module load bioawk
$ bioawk -c hdr '$Chromosome == "1" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr1_maize.txt
$ bioawk -c hdr '$Chromosome == "2" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr2_maize.txt
$ bioawk -c hdr '$Chromosome == "3" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr3_maize.txt
$ bioawk -c hdr '$Chromosome == "4" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr4_maize.txt
$ bioawk -c hdr '$Chromosome == "5" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr5_maize.txt
$ bioawk -c hdr '$Chromosome == "6" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr6_maize.txt
$ bioawk -c hdr '$Chromosome == "7" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr7_maize.txt
$ bioawk -c hdr '$Chromosome == "8" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr8_maize.txt
$ bioawk -c hdr '$Chromosome == "9" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr9_maize.txt
$ bioawk -c hdr '$Chromosome == "10" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr10_maize.txt
```
 
### Maize SNPs ordered based on decreasing position values and missing data encoded by the symbol: - 
 
I used the same `bioawk` commands (as above) but "tweaked" the sort options in the reversed order using `rn`.
 
I then piped the results of `sort` to `sed` to identify and replace all the missing values recorded as "?" with "-".
 
I then sent the final results to a separate file. I used the same command for all the chromosomes
 
```
$ bioawk -c hdr '$Chromosome == "1" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr1_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "2" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr2_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "3" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr3_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "4" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr4_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "5" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr5_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "6" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr6_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "7" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr7_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "8" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr8_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "9" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr9_rev_maize.txt
$ bioawk -c hdr '$Chromosome == "10" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr10_rev_maize.txt
```
To ensure successful find and replacement of all symbols, I used the `less` command on the created file and then searched for the new replacements. This was done for all the files. Below is the snippet of the command I used.

```
$ less chr3_rev_maize.txt
#/-
```
### Maize SNPs with unknown positions

I used the `awk` command to search and find SNPs with unknown positions. To be certain of the file I was working with, I `cut` the first 5 columns and looked at the first 20 rows using `head` and the last 20 rows using `tail` command. 

```
$ awk -F "\t" '$3 ~ /unknown/' head_joined_maize.txt > unknown_position_maize.txt
$ cut -f1,2,3,4,5 unknown_position_maize.txt | head -n 20
$ cut -f1,2,3,4,5 unknown_position_maize.txt | tail -n 20
```

### Maize SNPs with multiple positions

I also used `awk` to obtain this file and as well checked it using the `cut` command.

```
$ awk -F "\t" '$3 ~ /multiple/' head_joined_maize.txt > multiple_position_maize.txt
$ cut -f1,2,3,4,5 multiple_position_maize.txt | head -n 20
$ cut -f1,2,3,4,5 multiple_position_maize.txt | tail -n 20
```

## Processing teosinte data files

I took similar steps to those used for processing the maize data files-with just a few tweaks to get the teosinte data files 

### The joint teosinte file

Snippet of the unix commands used (similar to those used for maize data files processing)

```
$ grep 'ZMP' fang_et_al_genotypes.txt > teosinite_data.txt
$ cat headers_fang.txt teosinite_data.txt > headed_teosinite.txt
$  awk -f transpose.awk headed_teosinite.txt > transposed_teosinite.txt
$ tail -n +4 transposed_teosinite.txt > unheaded_transposed_teosinite.txt
$ sort -k1,1 unheaded_transposed_teosinite.txt > sorted_unheaded_teosinite.txt
$ sort -k1,1 -c sorted_unheaded_teosinite.txt
$ echo $?
$ wc -l sorted_unheaded_teosinite.txt
$ join -1 1 -2 1 -t $'\t' sorted_unheaded_snp.txt sorted_unheaded_teosinite.txt > joined_teosinite.txt
$ cut -f1,2,3,4 joined_teosinite.txt | head -n 4
$ tail -n +5 joined_teosinite.txt | awk -F "\t" '{print NF; exit}'
$ cat headers_snp_position.txt joined_teosinite.txt > head_joined_teosinite.txt
```

## The 22 teosinte snp files 

### Teosinte SNPs ordered according to increasing position values

```
$ bioawk -c hdr '$Chromosome == "1" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr1_teosinite.txt
$ bioawk -c hdr '$Chromosome == "2" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr2_teosinite.txt
$ bioawk -c hdr '$Chromosome == "3" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr3_teosinite.txt
$ bioawk -c hdr '$Chromosome == "4" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr4_teosinite.txt
$ bioawk -c hdr '$Chromosome == "5" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr5_teosinite.txt
$ bioawk -c hdr '$Chromosome == "6" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr6_teosinite.txt
$ bioawk -c hdr '$Chromosome == "7" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr7_teosinite.txt
$ bioawk -c hdr '$Chromosome == "8" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr8_teosinite.txt
$ bioawk -c hdr '$Chromosome == "9" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr9_teosinite.txt
$ bioawk -c hdr '$Chromosome == "10" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr10_teosinite.txt
```

### Teosinte SNPs ordered according to decreasing position values and missing data encoded by the symbol: - 

```
$ bioawk -c hdr '$Chromosome == "1" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr1_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "2" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr2_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "3" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr3_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "4" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr4_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "5" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr5_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "6" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr6_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "7" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr7_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "8" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr8_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "9" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr9_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "10" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr10_rev_teosinite.txt
```
  
Ascertaining successful replacement was done using the `less` command and searched for the "-" using `#/-`
  
```
$ less chr3_rev_teosinite.txt
#/-
```
### Teosinte SNPs with unknown positions 

```
$ awk -F "\t" '$3 ~ /unknown/' head_joined_teosinite.txt > unknown_position_teosinite.txt
$ cut -f1,2,3,4,5 unknown_position_teosinite.txt | head -n 20
$ cut -f1,2,3,4,5 unknown_position_teosinite.txt | tail -n 20
```

### Teosinte SNPs with multiple positions

```
$ awk -F "\t" '$3 ~ /multiple/' head_joined_teosinite.txt > multiple_position_teosinite.txt
$ cut -f1,2,3,4,5 multiple_position_teosinite.txt | head -n 20
$ cut -f1,2,3,4,5 multiple_position_teosinite.txt | tail -n 20
```
