## lesson01

### 1.1 Direct yourself in between and within directories
```pwd``` this command displays the pathway from home to current directory you are in

```ls``` this command as its name is, lists the directories and files within the directory you are in

```ls -l``` this command lists the modes of each file and directory within your current directory.
For example:
```
ls -l
total 843008
drwxr-xr-x  18 lyl  staff        576 Aug  3 04:00 data
drwxr-xr-x@  8 lyl  staff        256 Jan 20  2020 data 2
-rw-r--r--@  1 lyl  staff  431616000 Feb 20  2020 data.tar
drwxr-xr-x   5 lyl  staff        160 Aug  8 03:20 lesson01
-rw-r--r--   1 lyl  staff         81 Aug  6 06:27 mini.fastq
```
```ls -a ``` this command shows you every file even hidden ones within your current directory. Keep in mind that there will be two directroeis
```. ``` and ```.. ``` these are the ones that marks your previous and current directory.
If you want to direct yourself into another directory, use```cd```. When you use ```cd .``` it points to your current directory, but when using ```..```, it directs you the your previous directory in the pathway tree.
Now that you know how to direct yourself within and in between directories, you can use```mkdir``` to create a directory yourself.

### 1.2 Move and read a file

#### mv/cp/rm
Now, to move or copy the file, we can use ```mv``` to move it to another folder, ```rm``` to remove file pr directory, and ```cp``` to copy the file into another directory. Keep in mind that the order is:
```mv/cp filename.format target directory``` if file is in current directory. If not then point out the pathway as well.

#### cat/less/head/tail/sort/wc
To read a file, you use ```cat``` to display the text from the file. ```cat``` allows you to read the file at once.
For larger file that will be too long to display on terminal, simple use ```less``` to display one page at a time. Press ```ENTER``` to turn to next page, and press ```q``` to end your reading.
Use ```head ``` and ```tail ``` to display only the first and last ten lines of the file text. You can also display only five or six lines depending on your need. For example, if you only need to read the first three lines of the text, you can type
```
head -n 3 file.txt
```
or
```
head -3 file.txt
```
> As you can see, the freedom in Unix is pretty high. We are also gonna discuss about it and how this help us explore more pathways to solve each problem or request later.

You can also create your own file without need to exit your terminal. First type ```cat > *yourfilename.format```, press enter, then write things you wanna write in this file.
If you already have a file, say, ```text mytext.txt```, and you want to add something else into the file, but you be too lazy to open the file in your laptop folder, you can simply type ```cat >> mytext.txt``` and then type whatever you want to add them into the file.
If you list a bunch of items and input into a file, and now you want to sort the items you listed in alphabet table order, you can use ```sort < filename.format``` to display items in alphabet table order.
To sort lines, use ```sort -n filename.format```.
To sort in numerical order, use ```sort -r filename.format```.

If you want to count the lines and words in a file, use ```wc``` command. If you wanna count the lines, type ```wc -l filename.format```; to count the words in the file, type ```wc -w filename.format```.

#### wildcard
The wildcard helps you to find a file when you forget its name. If you wanna find ```list1/2/3```, use ```ls list*``` to find it. If you forget if you saved as biglist/longlist blahblahblah, use ```ls *list```, it will display all the files names that end with list. 
If you want to find a list that has exactly one character before the ```list```, use ```ls ?list```.

#### Grep
To display certain lines with the words you are looking for, use ```grep```. ```grep``` helps you to know which lines you want to look for when the file itself is too long to dig throughoutly. The ```grep``` format goes:
```
grep 'character/word' filename.format
```
To grep anything that begins with a symbol @, use ```grep '^@' filename.format```.
To grep anything that ends with 'ABC', use ```grep 'ABC$' filename.format```.
To also show the line numbers to the associated lines retrieved, we use ```grep -c 'character/word' filename.format```.

#### Pipe
Now, we know how to read and retrieve certain lines, but what if we want to make two commands at the same time? Say we want to first know which lines contain the word 'thing', and we want to count the number of the lines, we can type:
```
grep 'thing' filename.format | wc -l
```
Now, remember, pipe reads command 1 | command 2, and will only have output of command 2 because command 1 output is input of command 2.

### 1.3 Other stuff

#### echo
Echo can be used in a lot of ways. If you ```echo``` some text, say you ```echo mybodytea```, your terminal would literally echo:
```
mybodytea
```
Alternatively, you can first set a variable, and then use echo to print it out. For example:
```
name='My body is tea'
echo "$name"/$name #be careful that if you use '$name' instead of "$name" here, it thinks that you wanna print the literal $name. So please use double quotes OR don't use quotes
```
The output would be as:
```
my body is tea
```
Amazing! Now, we can make this a bit more complex. Say we name a command into a variable and echo it.
```
N=$(wc -l filename.format)
echo $N
```
The result will then give us the line number and the filename.
You can also echo text into new files to save them. Simply use ```echo 'text' > filename.format```.

#### printf
```printf``` prints formatted output. For example, ```printf "hello\n"``` prints hello and then a new line. If the ```\n``` is removed, the system will automatically add % as a symbol of changing the line.
For more formats of ```printf``` command, see below:
|  %s  | string |
|------|------- |
|  %d  | integer|
|  %f  | decimal number|
|  \n  | new line|

For example, if we want to print out the file name, we could type:
```
name="lexietheking.txt"
printf "File: %s\n" "name"
```
The output would look like:
```
File: lexietheking.txt
```

#### Uniq
```uniq``` removes the adjacent duplicated lines. ```uniq filename.format``` makes sure that there is no duplicated lines in that file. To display the file without lines repeating, you can sort it first, and then use pipe:
```
sort filename.format | uniq
```
So the lines that shows up are the "unique" ones.

You can also use pipe to count the occurance:
```sort filename.format | uniq -c
```
The output would show how many times each line displays:
```
   1 hello
   1 i am the king
   1 idk why people took golden shpwer
   1 ik they stink asf tho
   2 maybe they need amonia
```

#### gzip/gunzip/gzcat/zcat
To compress a file, use ```gzip filename.format``` to zip it into ```.gz``` format; to decompress any ```.gz``` zip file, unzip with ```gunzip```.
To read a zipped file without decompressing it, use ```gzcat``` or ```zcat```.

#### history, jobs
To check your coding history, use ```history```, you can also limit the number of history you wanna check, use ```history -500``` to see the most recent 500 histories you coded.
To check what jobs are running right now, use ```jobs```, and to put things to background/frontground, use ```bg``` or ```fg```.
### Cancel/suspend/end command or input
To cancel input/quote/command, press ```Ctrl C```.
To suspend a job, press ```Ctrl Z```.
To end the command or input after you finish, press ```Ctrl D```.

#### *awk*
Now, we finally start with the real programming, the ultimate command: ```awk```. This is super important throughout your Unix learning path. ```awk``` command allows you to put conditions and your actions upon a file.
```
awk 'condition {action}' file
```

Now you've basically known everything about how to process files and directories using Unix, and we are gonna learn more complicated commands and loops using the command we learned today in later lessons. See you in lesson02!

> Today's command list:
> |command|function|
> |-------|--------|
> |pwd|find the pathway|
> |ls|list the directories and files|
> |mkdir|make a directory|
> |cd|redirect you to another directory|
> |cat|read the entire file|
> |less|read the file one page at a time|
> |head|read the first ten lines of a file|
> |tail|read the last ten lines of a file|
> |cp|copy a file|
> |mv|move a file|
> |rm|remove a file|
> |wc|count lines/words in a file|
> |sort|sort the items/lines in a file|
> |grep|find the target line using key word|
> |uniq|print the line without duplication|
> |gzip|compress the file|
> |gunzip|decompress the file|
> |zcat|read the file without decompressing it|
> |echo|echo the input|
> |printf|print formatted output|
> |awk|command action based on condition|

