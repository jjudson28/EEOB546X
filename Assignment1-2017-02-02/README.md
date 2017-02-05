#Assignment #1: Jessica Judson

##Setting up 
My github repository is [EEOB546X](https://github.com/jjudson28/EEOB546X).

First, I **made a new dated directory** in my github repository using `mkdir Assignment1-$(date +%F)` in my terminal (where all of this assignment will be taking place). This directory will house all files and the README.md for this assignment.

I **pulled** from the github repository for the class `git pull origin master` and **copied** the two text files for this assignment into my new Assignment1 directory `cp fang_et_al_genotypes.txt ~/Desktop/EEOB546X/Assignment1-2017-02-02/`, `cp snp_position.txt ~/Desktop/EEOB546X/Assignment1-2017-02-02/`, and the awk file.

##Inspecting the Data Files-Workflow
My workflow is as follows:  
1. Investigate datafile sizes `du -h fang_et_al_genotypes.txt snp_position.txt`   
2. Find out what type of characters are in the file `file fang_et_al_genotypes.txt snp_position.txt`  
3. Get a count of lines, words, characters `wc fang_et_al_genotypes.txt snp_position.txt`  
4. Look at format of headers (if present) `head fang_et_al_genotypes.txt``head snp_position.txt` 
5. Confirm our understanding of the number of columns using awk `awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt` `awk -F "\t" '{print NF; exit}' snp_position.txt`  

##Inspecting the Data Files-Results
1. **The filesizes are:**  
	* 11M for `fang_et_al_genotypes.txt`  
	* 84K for `snp_position.txt`  

2. **The characters are all ASCII text**
3. **The line count, word count, and character counts are:**

    |File Name|Lines|Words|Characters|
|---|---|---|---|
|`fang_et_al_genotypes.txt`|2783|2744038|11051938|
|`snp_position.txt`|984|13198|82763|

4. **There are lots of columns in** `fang_et_al_genotypes.txt`, so instead I use the awk command to count the columns. See step 5. Same for `snp_position.txt`.  
5. **The column numbers are:**  

    |File Name|Columns|
|---|---|
|`fang_et_al_genotypes.txt`|986|
|`snp_position.txt`|15|  

	It looks like `fang_et_al_genotypes.txt` is a large genotype file with samples genotyped at many SNP loci (around 980 loci), though some of the labels are somewhat ambiguous. On the other hand, `snp_position.txt` is a much smaller file. It has details including the chromosome position and type of marker for each SNP. 

##Data Processing-Workflow
1. Exploring the Group column with grep extended regular expression (ERE)  
	* Grep for the maize group and send to a file `grep -E "(Group|ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > maizegenotypes.txt`  
	* Grep for the teosinte group and send to a file `grep -E "(Group|ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > teosintegenotypes.txt`  
	* **VALIDATE**: Do the counts for the two groups and the excluded groups add up to the number of lines in the data? `grep  -c -E "(ZMMIL|ZMMLR|ZMMMR|ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt` `grep -v -c -E "(ZMMIL|ZMMLR|ZMMMR|ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt`  
	* **Yes, they match!**  
2. Get rid of extraneous columns in the genotype data (Individual ID, other ID, and group). We just want the names of the SNPs and the data. `cut -f 4-986 maizegenotypes.txt` First I tested this with `| head`, and when I verified that this is what I wanted (excluding the first three columns of the data), I combined the first grep step with the cut step and sent to a file, so that I only have one maize genotype file and one teosinte genotype file. `grep -E "(Group|ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt | cut -f 4-986 > maizegenotypes.txt` and `grep -E "(Group|ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt | cut -f 4-986 > teosintegenotypes.txt`   
3. Get rid of extraneous columns in the snp description file (cdv marker ID and everything after position). `cut -f 1,3,4 snp_position.txt > snp_position_cut.txt` 
4. Transpose the data in the new `maizegenotypes.txt` file and the `teosintegenotypes.txt` file and send the transposed data to two new files. `awk -f transpose.awk maizegenotypes.txt > transposed_maize_genotypes.txt` and `awk -f transpose.awk teosintegenotypes.txt > transposed_teosinte_genotypes.txt`  
5. Check to see if the files are sorted according to SNP_ID. `sort -k1,1 -c snp_position_cut.txt` **SNP description file is not sorted!** `echo $? = 1` **Transposed genotype files are not sorted!** Sort the genotype and SNP description files by the SNP name, the column that the two files have in common. `sort -k1,1 snp_position_cut.txt > sorted_snp_position_cut.txt`, `sort -k1,1 transposed_maize_genotypes.txt > sorted_transposed_maize_genotypes.txt` and `sort -k1,1 transposed_teosinte_genotypes.txt >sorted_transposed_teosinte_genotypes.txt`  
6. Now **Join** the files! `join -t $'\t' -1 1 -2 1 sorted_snp_position_cut.txt sorted_transposed_maize_genotypes.txt > maize_joined_file.txt` `join -t $'\t' -1 1 -2 1 sorted_snp_position_cut.txt sorted_transposed_teosinte_genotypes.txt > teosinte_joined_file.txt`. **Confirm** that the files joined correctly using `awk -F "\t" '{print NF; exit}' maize_joined_file.txt` and head command.
7. Next, we need to isolate each chromosome and sort by position. For the first 10 files, we need 1 for each chromosome with SNPs ordered by increasing position values and missing data shown by '?'. To do this:
	* awk out all entries for a single chromosome, using: `awk '$2==1' maize_joined_file.txt`  
	* add to that a sort function to sort by position, `| sort -k3,3g`  
	* Since unknown SNPs are already coded with a '?', I don't need to modify anything, so the final command is `awk '$2==1' maize_joined_file.txt | sort -k3,3g > maizechromosomeq1.txt`  

8. I repeated this code for each chromosome. Then I did the same with the teosinte files.
9. For the next ten files of maize and teosinte with '-' denoting missing values:
	* awk out all entries for a single chromosome, using: `awk '$2==1' maize_joined_file.txt`
	* sort by descending position `| sort -k3,3gr`
	* change all '?' to '-' using `sed 's/?/-/g'`
	* `awk '$2==10' teosinte_joined_file.txt | sort -k3,3gr | sed 's/?/-/g' > teosintechromosome-10.txt`


##Data Processing-Results

The joined data file for maize consists of 1576 columns and 983 rows. The joined data file for teosinte consists of 978 columns and 983 rows. The column layout for both files is:

     SNP_ID | Chromosome | Position | Genotype data
     
The 40 generated files are in the folder **Results** and the intermediate files are in **Intermediate**. The results files are named either maize... or teosinte... and the 'q' or the '-' denote which punctuation depicts missing information. 






 
	

