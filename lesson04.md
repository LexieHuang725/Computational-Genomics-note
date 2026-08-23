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
Create a file named regions.bed:
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

### Validate the coordinates first

Before doing crazy math, check errors of files and data first:
* check if the file exists
* check if the start point/end point is numeric
* check if start point is bigger than end point
* check if the length is zero
