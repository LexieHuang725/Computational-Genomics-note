## Lesson04: Sorting and merging genomic intervals
---
Lesson 03 introduced ```.bed``` file format. Lesson 04 focuses on a more complicated problem:

> How do we calculate coverage of genomic sequence?

Simply adding interval lengths gives the wrong answer when regions overlap. We must:
* validate the coordinates;
* sort the intervals;
* merge overlapping intervals;
* calculate unique coverage for peaks in chromosomes.

### Practice data
Create a file named ```regions.bed```:
```
chr2	300	420	peakF
chr1	100	200	peakA
chr1	180	260	peakB
chr2	100	160	peakD
chr1	500	550	peakC
chr2	150	240	peakE
chr1	260	300	peakG
chrX	50	90	peakH
chr2	500	500	badZero
chr1	900	850	badReverse
chr3	start	400	badText
chr1	900	1000	peakI
```
The columns are:
|$1|Chromosome|
|--|-----------|
|$2|Start coordinate|
|$3|End coordinate|
|$4|Region name|
---
### Why overlap changes the calculation
Consider:
```
chr1	100	200	peakA
chr1	180	260	peakB
```
Their individual lengths are:
```
200−100=100
260−180=80
```
As a result, a naïve sum gives:
```
100+80=180
```
Cutting off the overlap, the entire occupation of sum of peaks on chr1 would be:
```
chr1	100	260
```
Therefore, unique covered length is:
```
260−100=160
```
The 20-bp overlap must not be counted twice.
This distinction appears constantly in genomics:
* total called peak length;
* exon coverage;
* accessible chromatin coverage;
* aligned reference coverage;
* repeat coverage;
* genomic territory covered by annotations.

### Part 1 - Validate the coordinates first
Before doing crazy math, check errors of files and data first:
* check if the file exists
* check if the start point/end point is numeric
* check if start point is bigger than end point
* check if the length is zero
```
awk '
$2 !~/^[0-9]+$/ || $3 !~/^[0-9]+$/ {
print "NONNUMERIC", $4}
$2~/^[0-9]+$/ && $3~/^[0-9]+$/ && $3<=$2 {
print "INVALID RANGE", $4
```
If you want to use if statement:
```
awk {
if ($2 !~/^[0-9]+$/ || $3 !~/^[0-9]+$/) {
print "NONNUMERIC", $4
} else {
if ($3<=$2) {
print "INVALID RANGE", $4}
}
}
' regions.bed
```
### Part 2 - Sort genomic intervals correctly
The valid records are not in genomic order. Before merging, sort by:
* chromosome;
* numeric start coordinate.
To sort the items in order, we can use thus command line:
```
sort -k1,1 -k2,2 file.format
```
```-k1,1``` here means key1 start sorting from column 1 and ends at column 1. Putting k1,1 first puts the column number in the first order of sorting. Using this you can now sort text. If you want to sort in numerical order, add n.
For example:
```
sort -k1,1n -k2,2n file.format
```
Here, since chromosome is not entirely numeric, we want to sort it by text. The column of start point is numeric, so we want to sort by numbers:
```
sort -k1,1 -k2,2n regions.bed
```
The output should look like this:
```
chr1	100	200	peakA
chr1	180	260	peakB
chr1	260	300	peakG
chr1	500	550	peakC
chr1	900	1000	peakI
chr1	900	850	badReverse
chr2	100	160	peakD
chr2	150	240	peakE
chr2	300	420	peakF
chr2	500	500	badZero
chr3	start	400	badText
chrX	50	90	peakH
```
*Note: if you use ```-k1,2```, you sort both columns as one key, which could cause a false result in the same order. For example, it could possibly cause a result of:*
```
chr1 100
chr 200
```
*Also, if you don't sort numbers with the command n, it could possibly cause the same result.*
> Exercise: Try to sort the peak.
> ```sort -k4,4 regions.bed```
> ```
> chr1	900	850	badReverse
> chr3	start	400	badText
> chr2	500	500	badZero
> chr1	100	200	peakA
> chr1	180	260	peakB
> chr1	500	550	peakC
> chr2	100	160	peakD
> chr2	150	240	peakE
> chr2	300	420	peakF
> chr1	260	300	peakG
> chrX	50	90	peakH
> chr1	900	1000	peakI
> ```


### Part 3 — Build a validation-and-sorting pipeline
We want to create a ```sorted_regions.bed``` file to put sorted results from the original file so that it would be easier for us to do further work. You can also work with original file, it's totally up to you. Let's just use this as a backup.
```
awk '
$2~/^[0-9]+$/ && $3~/^[0-9]+$/ && $3>$2
' regions.bed |
sort -k1,1 -k2,2n > sorted_regions.bed
```

### Practice 1 - Merge the intervals
We can see that some of the peaks have overlap with one another. We want to deduct the repeatedly counted overlapping regions. How do we achieve that?
First, we want to define one chromosome, and then work on that chromosome first. We can use the sorted file as the source, or we can sort from original file. We want to store the data of previous start and end point and compare the current line when we go thru the dataset.
This can be tricky, but I do have some hint here for you.
In ```awk``` command, we have something called ```next```, it stops processing the current input line and immediately reads the next line. For example, we have some bad reads in a file:
```
chr1    100    200
chr1    bad    300
chr1    400    500
```
Then we can use ```next``` to stop generating the second line with bad reads and skip to the third one.
```
awk '
$2!~/^[0-9]+$/ || $3!~/^[0-9]+$/ {
next
}
{print $0}
' regions.bed
```
For this problem, we want to first store the first line as a reference, and go through the same chromosome first to compare the start points of each peak, see if there is any overlap in the regions. After storing the value, we don't need to process this line no more, so we simply use ```next``` to move on to next line.
```
awk '
NR==1 {
chr=$1
start=$2
end=$3
next
}
'
```
And now we move on to the next line, we want to compare current line with what we stored. What we can do is that we can find those peaks whose start point is within the region of previous peak. Then we can update the end point of the region so that we can cumulate its length.
```
$1 == chr && $2 <= end {
end=$3
next}
```
Now we have everything ready, we want to print out the results. When going through the lines, we want to print out what is already stored. After printing command, we still need to update the data. For example, we need to move to the next chromosome.
```
{print chr, start, end
chr=$1
start=$2
end=$3
}
```
Now it's time to be cautious. A common bug can happen because the print only has the compared results. During normal processing, a merged interval is printed only when a gap or new chromosome is encountered. Nothing comes after the last interval to trigger that print. Therefore, we need to print the last line:
```
END {
print chr, start, end
}
```
Now we can assemble everything together:
```
awk '
$2~/^[0-9]+$/ && $3~/^[0-9]+$/ && $3>$2
' regions.bed |
sort -k1,1 -k2,2n |
awk '
NR==1 {
chr=$1
start=$2
end=$3
next
}

$1 == chr && $2 <= end {
end=$3
next
}

{print chr, start, end
chr=$1
start=$2
end=$3
}

END {
print chr, start, end
}
' > merged_regions.bed
```
bash the script, and then cat the file to check your result.

### Practice 2 — Per-chromosome coverage
We can calculate the unique coverage of each chromosome by adding the regions the peaks cover. Since the ```merged_region.bed``` file already has the overlapped regions cut off, we can simply add the lengths together.
```
awk '{
coverage[$1] += $3-$2
}
END {
for (chr in coverage) {
print chr, coverage[chr]
}
' merged_regions.bed
```
The for loop works like this:
```
for (key in array) {
    action
}
```
In this case, chr is a new variable we create along the way, and the ```awk``` command recognize it as we formally define of the coverage. Going through the different keys makes the code walks through different chromosomes. You can also sort the chromosomes by order.


### Practice — Build ```unique_coverage.sh```
Now we want to calculate the unique coverage of each chromosome from the original file. We can use the same method without the for loop to sum up the coverage together without overlap and gap.
The task for this problem would be:
* calculate unique coverage of each chromosome
* count the number of intervals
* list peaks within each of the chromosome regions

```diff
awk '
$2~/^[0-9]+$/ && $3~/^[0-9]+$/ && $3>$2
' regions.bed |
sort -k1,1 -k2,2n |
awk '
BEGIN {
OFS="\t"
print "chromosome", "unique_coverage", "count", "peaks"
}
NR==1 {
chr=$1
start=$2
end=$3
peaks=$4
count=1
covered=0
next
}
+###+when overlapping, redefine the end point; when having gap, define covered as the previous merged interval, and redefine the end and start point as the current one after the gap.
$1==chr {
count++
peaks = peaks "," $4
if ($2<=end) {
end=$3
} else {
covered+=end-start
end=$3
start=$2
}
next
}

+###add the unique coverage together by adding covered(merged interval) and end and start for the last line of the same chromosome together
{
unique=covered+end-start
print chr, unique, count, peaks
chr=$1
start=$2
end=$3
peaks=$4
covered=0
count=1
}

END {
unique=covered+end-start
print chr, unique, count, peaks
}
'
```
---
## Problem Set
Now let's practice!
### Problem 1 — Count rejected records by reason
Read ```regions.bed``` once and produce:
```
Nonnumeric coordinates: 1
Zero-length intervals: 1
Reversed intervals: 1
Valid intervals: 8
```
*Requirements:*
* Check numeric validity before coordinate arithmetic.
* Use four counters.
* Do not print individual records.
* Use next to keep the categories mutually exclusive.

```
awk '{
if ($2!~/^[0-9]+$/ || $3!~/^[0-9]+$/) {
count_nonnum++
} else {
if ($3=$2) {
count_zero++
} else if ($3<$2) {
count_reverse++
} else {
count_valid++}
}
}
END {
print "Nonnumeric coordinates:", count_nonnum
print "Zero-length intervals:", count_zero
print "Reversed intervals:", count_reverse
print "Valid intervals:", count_valid
}
' regions.bed
```
---
### Problem 2 — Preserve rejected records
Create two output files in one awk command:
```
clean_regions.bed
rejected_regions.tsv
```
The rejection file should have this structure:
```
badZero	zero_length
badReverse	reversed
badText	nonnumeric
```
Question: what problem might occur if you rerun the command and the old files still exist?

```
#!/bin/bash
awk '
$2~/^[0-9]+$/ && $3~/^[0-9]+$/ && $3>$2 {
print $0
}
' "$1" > "$2"

awk '{
if ($2!~/^[0-9]+$/ || $3!~/^[0-9]+$/) {
nonnum = $4
} else {
if ($3<$2) {
reversed = $4
} else if ($3==$2) {
zero_length = $4}
}
}
END {
print nonnum, "nonnumeric"
print reversed, "reversed"
print zero_length, "zero_length"
}
' "$1" > "$3"
```
---
### Problem 3 — Detect unsorted input
Write an awk program that examines a BED file and reports any record that violates chromosome/start ordering.
For example:
```
UNSORTED at line 2: chr1 100
```
You will need to remember:
```
previous_chr
previous_start
```
The current record is out of order if:
* its chromosome sorts before the preceding chromosome; or
* it has the same chromosome but a smaller start coordinate.
This problem is about comparing consecutive records, not sorting them.

```
awk '
$2~/^[0-9]+$/ && $3~/^[0-9]+$/
' regions.bed |
awk '
BEGIN {
OFS="\t"
}
NR==1 {
chr = $1
start = $2
next
}
{
if ($1==chr) {
if ($2<start) {
print "UNSORTED at line " NR ":", $1, $2
} else {
start = $2}
}
else if ($1<chr) {
print "UNSORTED at line " NR ":", $1, $2
}
chr=$1
start=$2
}
'
```
or simpler:
```
awk '
BEGIN {
    OFS = "\t"
}
$2 !~ /^[0-9]+$/ || $3 !~ /^[0-9]+$/ {
    next
}
!have_previous {
    previous_chr = $1
    previous_start = $2
    have_previous = 1
    next
}
$1 < previous_chr ||
($1 == previous_chr && $2 < previous_start) {
    print "UNSORTED at line " NR ":", $1, $2
}
{
    previous_chr = $1
    previous_start = $2
}
' regions.bed
```

---
### Problem 4 — Find the most redundant chromosome
For each chromosome, calculate:
```
redundant bp=naive interval bp−unique merged bp
```
Expected values include:
```
chr1 20
chr2 50
chrX 0
```
Then report the chromosome with the greatest redundant coverage:
```
Most redundant chromosome: chr2 50 bp
```
You may use intermediate files, but the stronger solution uses two pipelines and associative arrays.

```diff
awk '
$2~ /^[0-9]+$/ && $3~ /^[0-9]+$/ && $3>$2
' regions.bed |
sort -k1,1 -k2,2n |
awk '
BEGIN {
OFS="\t"
}
NR==1 {
chr=$1
start=$2
end=$3
naive=end-start
next
}
$1 == chr {
naive+=$3-$2
if ($2 <= end) {
end=$3
} else {
unique_interval+=end-start
end=$3
start=$2
}
next
}

+###print out the rows of chr and redundent bp lengths. Then compare the dedundent lengths.
{
unique = unique_interval+end-start
redundent = naive - unique
print chr, redundent
if (!have_max || redundent > maximum_red) {
maximum_red = redundent
max_chr = chr
have_max = 1
}
+###reset the variables after printing
chr=$1
start=$2
end=$3
naive = $3 - $2
unique_interval=0
}
END {
if (NR > 0) {
unique = unique_interval + end - start
redundent = naive - unique
print chr, redundent
if (!have_max || redundent > maximum_red) {
maximum_red = redundent
max_chr = chr
}
print "max:", max_chr, maximum_red
}
}
'
```
---
## Final challenge — Coverage report script
Write ```genome_coverage.sh```
Run it as:
```
bash genome_coverage.sh regions.bed coverage_report.tsv
```
The output file should contain:
```
chromosome	original_intervals	merged_intervals	naive_bp	unique_bp	redundant_bp
chr1	4	2	270	250	20
chr2	3	2	310	260	50
chrX	1	1	40	40	0
```
Your script must:
* reject nonnumeric, zero-length, and reversed intervals;
* sort valid intervals;
* merge overlaps and adjacency;
* calculate per-chromosome statistics;
* preserve the header above sorted results;
* avoid leaving temporary files behind if you choose a pipeline-only design;
* produce a meaningful message when no valid intervals exist.
The difficult part is that the original and merged datasets contain different information. You must decide how to preserve the naïve totals and original counts while separately calculating merged totals and counts.

```

```
