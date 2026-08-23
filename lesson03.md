## Lesson03
---
For today, we are gonna introduce a new file format: ```.bed```. This file can show a matrix of gene with chromosomes they are in, charge of their DNA strand, start bp and end bp, and the feature numbers.
We can start with creating a mini ```gene.bed```:
```
chr1	100	200	geneA	15	+
chr1	450	700	geneB	42	-
chr2	300	380	geneC	8	+
chr2	800	1200	geneD	67	-
chrX	150	225	geneE	21	+
chr1	900	1000	geneF	42	+
chr2	1500	1600	geneG	5	-
chrX	500	900	geneH	80	-
```
Use your skill to create this file and save it into ```lesson03/genes.bed```

Now we have our file, notice this file has several columns and rows. For Unix, when we have a file like this, we can use ```$number``` to track down or refer to a certain column.
For example, if we want to print the entire input line, we can use ```$0```; first column is ```$1``` and so on.
> For this bed file, we can see that:
> |`$1`|chromosome number|
> |:--:|:---------------:|
> |`$2`|start point|
> |`$3`|end point|
> |`$4`|gene number|
> |`$5`|expression|
> |`$6`|strand charge|
---
### Practice 1: try to only display the chromosome number and their gene features.
```
awk '{print $1, $4}' genes.bed
```
The result would look like this:
```
chr1 geneA
chr1 geneB
chr2 geneC
chr2 geneD
chrX geneE
chr1 geneF
chr2 geneG
chrX geneH
```
### Practice 2: find the genes input lines on chromosome 2.
> hint: try to only look for the lines which the column matches the chromosome number
```
awk '$1=="chr2" {print $0}' genes.bed
```
the result would look like:
```
chr2	300	380	geneC	8	+
chr2	800	1200	geneD	67	-
chr2	1500	1600	geneG	5	-
```
### Practice 3: reformat the file:
In this problem, try to make your output look like this:
```
geneA: chr1:100-200 (+)
geneB: chr1:450-700 (-)
```
We see that the output lists the genes on chromosome 1, and it displays the chromosome, gene, and start and end point of the gene. with the charge of the strands.
Let's figure it out:
```
awk '$1=="chr1" {print $4 ":", $1 ":", $2 "-" $3, "(" $6 ")"}' genes.bed
```
### Practice 4: Require two biological conditions
Try to print the name, interval length, and expression of every feature that:
* is longer than or equal to 100 bp;
* has expression greater than 20.
We require two conditions, which we learned from last class that we can use ```&&``` to include them both in one condition set. Interval length would be the length of the gene, which should be end point minus start point.
```
awk '
$3-$2>=100 && $5>20 {
print $4}
' genes.bed
```
The output should look like:
```
geneB
geneD
geneF
geneH
```
### Practice 5: boundary trap
Print the genes whose lengths are strictly between 100bp and 400bp, meaning that their lengths must be greater than 100bp but smaller than 400bp.
Still, we use ```&&```:
```
awk '
$3-$2>100 && $3-$2<400 {
print $4}
' genes.bed
```
Only geneB should match the requirements.

### Practice 6: counting without printing every match
Try to count the number of negative strands and positive strands in the file. Since counting is cumulative, we shall use ```count++``` to count the total numbers, and remember to use ```END``` we learned from last class to print the final result instead of printing everytime the count goes up.
```
awk '
$6 == "-" {
count_neg++
}
$6 == "+" {
count_pos++
}
END {
print "negative features:", count_neg
print "positive features:", count_pos
}
' genes.bed
```
> if you want to use ```if``` statement, make sure that ```if``` stands within the action quote.
> ```
> awk '
> {if ($6 == "-") {
> count_neg++
> } else {
> count_pos++}
> }
> END {
> print "negative features:", count_neg
> print "positive features:", count_pos}
> }
> ' genes.bed
> ```
> Should have the same output.

### Practice 7: calculate specific chromosome genes mean length
Now try to calculate gene average lengths of chromosome 1.
```
awk '
$1 == "chr1" {
total_length += $3 - $2
count++
}
END {
print "chromosome 1"
print "features:", count
print "mean length:", total_length/count
}
' genes.bed
```
*Brainstorm: what if we want to change the chromosome, what if we want to command another chromosome without need to change the code? What can we do? We will discuss this later when I introduce```bash``` command later in this lesson.*

### Practice 8.1: find the longest gene
For this problem, use the ```awk``` command to find the *first* gene with the longest length.
Inside the main block:
* Calculate the current length.
* Compare it with the longest length seen so far.
* If it is larger, remember both the length and name.
* Print the stored result in END.
* If there is a tie, print out the first one you see with the longest length.
```
awk '
{current = $3 - $2
if (current > maximum) {
maximum = current
gene = $4}
}
END {
print "The longest feature:", gene, maximum "bp"}
' genes.bed
```
*Think about what you should do if we wanted to print out the *last* gene with the longest length.

### Practice 8.2: find the shortest gene
For this problem, use ```awk``` to find the *first* gene with the shortest length.
* Without assigning a value, ```minimum``` will always be 0, therefore, if we only change ```>``` to ```<```, this time it won't work. What we can do is that we can assign the first line value to the minimum by using ```or``` statement, which writes as ```||``` in ```awk``` command.
```
awk '
{current = $3 - $2
if (NR==1 || current < shortest) {
shortest = current
gene = $4}
}
END {
print "The shortest feature:", gene, shortest "bp"}
' genes.bed
```
### Practice 9: report every longest feature
Now that we want to report every longest feature in the file, we want to print everytime we see a gene with the longest length. What we are gonna do here is that we want to store the longest length as a value first, and then use ```awk``` to match each gene with the value.
How do we store something as a variable with value so that we can use it later? We can use define command to define an entire ```awk``` command first, because we can't know what is the longest length until walking through the entire bed file. When defining with ```awk``` command, we use ```awk=$(awk '...' file.format)```.
Therefore, the first part, which is the defining part would be like:
```
maximum=$(awk '
{current = $3 - $2
if (current > maximum) {
maximum = current}
}
END {
print maximum}
' genes.bed)
```
If you didn't use ```nano```, you will be sent back to the command base after running this without outcome. This means that your variable is defined and the value is stored under the name of your variable.
Now we just need to look for every gene features that match with the longest length. For this part, we use ```-v``` to acquire the value of the input.
```
awk -v max="$maximum" '
{if ($3-$2==max) {
print $4, max "bp"}
}
' genes.bed
```
There, you now learned how to define a value and then acquire the matching features using the values stored.

### Practice 10: detect genes with errors
Sometimes not everything is peachy and perfect. It is normal for a gene dataset to have small issues, such as negative length, nonnumeric values, etc. We will start with a mini file with errors of the data. Copy the data below and save it into a file named ```genes_with_error.bed```.
```
chr3	500	400	badGene1	18	+
chr3	700	700	badGene2	25	-
chr3	start	900	badGene3	30	+
```
Now print out the issues with a desired format:
```
ERROR badGene1: end <= start
ERROR badGene2: end <= start
ERROR badGene3: nonnumeric coordinate
```
We can define two conditions using ```if``` or just awk condition depends on your preference. Try to practice by yourself without looking at the answer.
> Hint: to make sure something is numerical, we use ```~/condition/``` to check. ```^``` checks if the line begin with certain things, and ```$``` checks the end.
```
awk '
{if ($3<=$2) {
print "ERROR", $4 ": end <= start"
}
if ($3 !~/^[0-9]+$/ || $2 !~/^[0-9]+$/) {
print "ERROR", $4 ": nonnumeric coordinate"}
}
' genes_with_error.bed
```
```
awk '
$3<=$2 {
print "ERROR", $4 ": end <= start"
}
$3 !~/^[0-9]+$/ || $2 !~/^[0-9]+$/ {
print "ERROR", $4 ": nonnumeric coordinate"}
' genes_with_error.bed
```
### Practice 11: create an output file
Imagine a file with wrong gene data but also valid gene data, and we want to only collect the valid ones into another file, what could we do?
Create a file named ```all_genes.bed```, from the file, create ```valid_genes.tsv``` containing only records that:
* have numeric coordinates;
* have end greater than start;
* have expression of at least 10.

Output four tab-separated columns:
```
feature	chromosome	length	expression
geneA  chr1  100  15
...
```
To format the columns with ```tab``` space, we can use something called "Output Field Separator", in command as ```OFS```.
How do we use OFS is that we define the separation symbols using ```OFS``` and store it as value in ```awk```.
*For example: we want to print out a b c with separation space of ```tab```, we can do:*
*```echo 'a b c' | awk -v OFS='\t' '{$1=$1; print}'```*
*or*
*```echo 'a b c' | awk -v OFS='\t' '{print $1, $2, $3}'```*
After we know how to use ```OFS``` to assign the separation space of each columns, we can now go ahead and create the file:
```
awk -v OFS='\t' '
BEGIN {
print "feature", "chromosome", "length", "expression"}
$2 ~/^[0-9]+$/ && $3 ~/^[0-9]+$/ && $3>$2 && $5>=10 {
print $4, $1, $3-$2, $5}
' all_genes.bed > valid_genes.tsv
```
Perfect! Now practice with other conditions and more!

---
### Bash Section: build a bash script one step by another
To start a bash command, we need to start the entire script with ```#!/bin/bash```. This guides us to the bash command. ```bash``` is a little bit different than that of ```awk```, whether in its format or usage. It might be confusing at first to differentiate these two but we will slowly build our blocks to learn how to use it as a tool.

Here comes the question tho, why *do* we need bash? To make our coding learning harder? To make students suffer? Eh maybe, but the most important part is that bash helps wrapping around the ```awk``` command. Imagine inputing a file that does not exist, bash can help imprint the error so that you understand what's wrong. You can clarify errors, process the file without needing to change your code, have free input to search within files, etc.

"WOW this sounds so cool~ how do we do all these things?" I know, but hold your horses. We gonna start with this specific problem, and expand one by one.
First, if you remember clearly, we had a mini showcase of bash script at the end of lesson02. In that scenario, we covered the possibility of reporting error of a non-existing file.
```
if [! -f "$1"]; then
echo "FileNotFound: $1"
exit 1
fi
```
For bash, when you use if, the format is ```if[condition]``` for file name and strings; for regex matching such as matching the column with their strings or symbols, use ```if[[condition]]```.
```bash``` also gives much more freedom to input the file you want. ```"$1"``` means the first input, in this case, your file. You can also add more input and simply refer to as ```"$2"``` or ```"$3"```. Remember to distinguish it with the column symbol in ```awk```. This always has quote marks.

Now we can print out the ```awk``` command to finish the entire script.
```
#!/bin/bash
if [! -f "$1"]; then
echo "FileNotFound: $1"
exit 1
fi

awk '
{gene_length=$3-$2
total+=gene_length}
END {
"gene number:", NR
"total length:", total
"mean length:", total/NR "bp"}
' "$1"
```

Now we want to progress to other conditions that might appear as errors when we input something outside the shell. For example, what if we didn't know how many input we are supposed to put, the order of the input, or if we had a typo.

### Final challenge: bed_report.sh
Final part of this lesson is a long bash script that covers detailed possible errors.
Your awk program must:
* Select the requested chromosome.
* Require length to be at least the requested minimum.
* Count matching features.
* Accumulate their total length.
* Calculate their mean length.
* Avoid dividing by zero when nothing matches.
* Require three input, and the minimal length has to be numeric
* Chromosome must starts with "chr"
The zero-matching output should look like:
```
Matching features: 0
No mean can be calculated.
```
Think about this question before rush and see the answer. You can also try see if you could cover more errors in your script.
```diff
#!/bin/bash
if [! -f "$1"]; then
echo "FileNotFound $1"
exit 1
fi

+# Now we check the number of our inputs. "$#" means the number of the inputs we have; ne means not equal, and 3 means that we want three inputs.
if [ "$#" ne 3 ]; then
echo "Usage: $0 File Chromosome MinimumLength"
exit 1
fi

+# Now we want to check if the input of chromosome starts with "chr"
if [[ ! "$2" =~ ^chr ]]; then
echo "Chromosome name must begin with chr"
exit 1
fi

+# Now we want to make sure the minimal length input is numeric
if [[! "$3" =~ ^[0+9]+$ ]]; then
echo "minimum length must be a number"
exit 1
fi

awk -v chr="$2" -v minimum="$3" '
$1=chr && $3-$2>=minimum {
total_length += $3-$2
count++
}
END {
if count>0 {
printprint "Chromosome:", chromosome
print "Minimum length:", minimum "bp"
print "Matched genes:", count
print "Total matching length:", total_length
print "Mean matching length:", total_length/count}
} else {
print "Matching features:", count
print"No mean can be calculated."}
' "$1"
```
