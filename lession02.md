## Lesson02
Now that we know basic commands, we can use it to process genomic sequencing data.
> In the book [Computational Genomics Tutorial](https://genomics.sschmeier.com/_downloads/e9803bd48e71605daa1e34ee3ddde426/Genomics.pdf), it is too fast for beginners to understand. Therefore, I'm going to break down this into slower lessons, with practice sets that helps us to understand each cases more.

Today we are focusing on a specific file format ```.fastq```. This format allows you to store sequence reads into a file and read/execute on it.
Our sample file would be:
```
@read1
ACGTTGCA
+
IIIIIIII
@read2
ACGNTGCA
+
III!IIII
@read3
TTTTGGCC
+
HHHHHHHH
```
Copy and paste this read and try to create a file by using ```>```.
```
cat > mini.fastq
```

Now that you've created the file under your current directory, check to make sure your file exists by using ```ls``` or ```cat``` to directly read your file. Now try to compress the file and read from it without decompressing it.
You can zip it without need to replace the original file by using:
```
gzip mini.fastq > mini.fastq.gz
```
Check if both files are there. To read it, use:
```
gzip -dc mini.fastq.gz
```
Or:
```
gzcat mini.fastq.gz
```
---

Now that we have created the files, we can go on with the practice sets:
### Part A: Basic Exercises

*1. List everything in the lesson02 directory.*

*2. List only files inside data.*

*3. Display the complete FASTQ file.*

*4. Display only the first four lines.*

*5. Display only the final four lines.*

*6. Count the total number of lines.*

*7. Explain why the line count should be divisible by four.*

*8. Print every line beginning with @.*

*9. Count how many lines begin with @.*

*10. Copy tiny.fastq into the results directory.*

*11. Rename the copied file to tiny_backup.fastq.*

*12. Compare the original and copied files using diff.*

*13. Display the file with line numbers.*

*14. Write the output of wc -l into results/line_count.txt.*

---
#### Answers:
1. List everything in the lesson02 directory.
   ```
   ls -a lesson02
   ```
2. List only files inside data.
   ```
   ls data
   ```
3. Display the complete FASTQ file.
   ```
   cat mini.fastq
   ```
4. Display only the first four lines.
   ```
   head -4 mini.fastq
   ```
5. Display only the final four lines.
   ```
   tail -4 mini.fastq
   ```
6. Count the total number of lines.
   ```
   wc -l mini.fastq
   ```
7. Count the lines of reads.
   ```
   awk 'END {print NR/4}' mini.fastq
   ```
   > Here, END means it only prints out the final output. NR means the number of the lines in a file. Since our read is every four lines, we divide the line number by 4 for the number of read/sequence lines. When NR/4 is going through first line, second line, etc...This command only prints out the end of the run, which shows 12 lines divided by 4, which is 3. If you didn't use END, then your output is going to look like:
   > ```
   > 0.25
   > 0.5
   > 0.75
   > 1
   > ...
   > 3
   > ```

8. Print every line beginning with @.
   ```
   grep '@' mini.fastq
   ```
9. Count how many lines begin with @.
    ```
    grep '@' mini.fastq | wc -l
    ```
    You can also use awk:
    ```
    awk '/^@/' mini.fastq | wc -l
    ```
    > Keep in mind that ^@ means it searches up the line starting with @.
    > 
    Or, if you don't want to use pipe:
    ```
    awk '/^@/ {count++} END {print count}' mini.fastq
    ```
    > Note: ```count++``` here means that every time command reads a line containing @, the count increases by one.
    
10. Copy mini.fastq into the lesson01 directory.
    ```
    cp mini.fastq > lesson02
    ```
11. Rename the copied file to tiny_backup.fastq.
    ```
    mv mini.fastq tiny_backup.fastq
    ```
    > This allows to rename the file
    
12. Compare the original and copied files using diff.
    ```
    diff tiny_backup.fastq lesson02/mini.fastq
    ```
13. Display the file with line numbers.
    ```
    cat -n mini.fastq
    ```
    >```diff
    >@@if you want to print certain lines you want with their line numbers, use grep -n '@' mini.fastq instead.@@
    >```
14. Write the output of wc -l into lesson01/line_count.txt.
    ```
    wc -l mini.fastq > lesson02/line_count.txt
    ```
    > If you only want to print out the number of the lines, use:
    > ```
    > wc -l < mini.fastq > lesson02/line_count.txt
    > ```
    > This format will only form the number of the lines without stating the file name itself.

---

### Part B: Introductory FASTQ exercises

*1. Print only sequence lines: lines 2, 6 and 10.*

*2. Print the length of every sequence.*

*3. Determine which read contains N.*

*4. Determine which read contains a low-quality ! character.*

*5. Print the read name and sequence together.*

*6. Save all sequence lines into results/sequences.txt.*

*7. Check that every sequence and corresponding quality line have equal lengths.*

---
#### Answer
1. Print only sequence lines: lines 2, 6 and 10.
   ```
   awk '
   NR%4==2{
   print $0}
   ' mini.fastq
   ```
   > This can be confusing to understand if you are a beginner. NR means the number of the lines, and condition here ```NR%4==2``` means that the line number divided by 4 should be a leftover of 2. $0 points to the current output of this condition. Therefore, the output should look like:
   > ```
   > ACGTTGCA
   > ACGNTGCA
   > TTTTGGCC
   > ```
2. Print the length of every sequence.
   ```
   awk '
   NR%4==2 {
   print length($0)}
   ' mini.fastq
   ```
3. Determine which read contains N.
   ```
   awk '
   NR%4==1 {
   read=$0}
   NR%4==2 && /N/ {
   print read, $0
   }
   ' mini.fastq
   ```
4. Determine which read contains a low-quality ! character.
   ```
   awk '
   NR%4==1 {
   read=$0}
   NR%4==0 && /!/ {
   print read, $0}
   ' mini.fastq
   ```
   > For this command you have to be patient to delete things pop out because ```/!/ ``` will pump out your pwd for python somehow. Don't ask author why cuz Lexie was confused too. Just don't change your line to a new one for the curly brackets ```{```, because it will not make the second condition functional and the output will write out literally every line of your file. Been there.
5. Print the read name and sequence together.
   ```
   awk '
   NR%4==1 {
   read=$0}
   NR%4==2 {
   print read, $0}
   ' mini.fastq
   ```
6. Save all sequence lines into lesson02/sequences.txt.
   ```
   awk 'NR%4==2 {print $0}' mini.fastq > lesson02/sequences.txt
   ```
7. Check that every sequence and corresponding quality line have equal lengths.
   ```
   awk '
   NR%4==1 {
   read=$0
   }
   NR%4==2 {
   seq_length=length($0)
   }
   NR%4==0 {
   quality_length=length($0)
   if seq_length == quality_length {
   print read,"OK", seq_length
   }
   else {
   print "Not aligned", seq_length, quality_length}
   }
   ' mini.fastq
   ```
   > For this command, keep in mind that when the program reads the NR%4==0 line, it will mark it as final $0 position, so include the final action into this condition. Also, when we move to comparing two different variable we stored, please use space in between them.
   > 
   > Here is a program using bash also as an answer. This provides more clarity to verify if this file exists, and uses ```bash``` commands except ```awk``` command:
   > ```diff
   > #!/bin/bash
   > +# Require exactly one command-line argument.
   > if [ "$#" -ne 1 ]; then
   >   printf 'Usage: %s FASTQ_FILE\n' "$0" >&2
   >   exit 1
   > fi
   >
   > file="$1"
   > 
   > +# Confirm that the supplied file exists.
   > if [ ! -f "$file" ]; then
   >   printf 'Error: file not found: %s\n' "$file" >&2
   >   exit 1
   > fi
   > 
   > +# Count the lines.
   > lines=$(wc -l < "$file")
   > 
   > +# Check whether the line count is compatible with FASTQ format.
   > if [ $((lines % 4)) -ne 0 ]; then
   >   printf 'Warning: the line count is not divisible by 4.\n' >&2
   > fi
   > 
   > +# Calculate the number of FASTQ records.
   > reads=$((lines / 4))
   > +# Obtain the first read identifier.
   > first_read=$(head -n 1 "$file")
   > 
   > +# Obtain the length of the first sequence.
   > read_length=$(awk 'NR==2 {print length($0); exit}' "$file")
   >
   > +# Display the results.
   > printf 'File: %s\n' "$file"
   > printf 'Lines: %s\n' "$lines"
   > printf 'Reads: %s\n' "$reads"
   > printf 'First read: %s\n' "$first_read"
   > printf 'Read length: %s\n' "$read_length"
   > ```
This code looks confusing because it uses bash commands. We will discuss more about bash command and difference between it and ```awk``` command next lesson. See you there!
