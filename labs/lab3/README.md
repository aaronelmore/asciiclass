# Lab 3

*Assigned: sometime*

*Due: Sometime (just before class)*

In this lab, you will use various types of tools -- from low-level tools like sed and awk to high-level tools like Data Wrangler -- to perform data parsing and extraction from data encoded into a text file.  The goal of this lab is simply to gain experience with these tools and compare and contrast their usage.


# Setup

To start, go to the AWS console and start your EC2 instance.  SSH into it.
First, you need to check out/update the files for lab3:

    cd asciiclass/labs/
    git pull

Then :

    cd lab3

The `lab3` directory contains two datasets (in addition to the datasets used in class):

1. A dataset of synonyms and their meanings (`synsets.txt`). Each line contains one synset with the following format:

    ID, &lt;synonyms separated by spaces&gt;, &lt;different meanings separated by semicolons&gt;

1. The second dataset (`worldcup.txt`) is a snippet of the following Wikipedia webpage on [FIFA (Soccer) World Cup](http://en.wikipedia.org/wiki/FIFA_World_Cup).
Specifically it is the source for the table toward the end, that lists the teams reaching the top four. 

# Wrangler

Go to the [Data Wrangler website](http://vis.stanford.edu/wrangler/app/).  Load each of the datasets (we recommend a small subset -- 100~ lines) into Data Wrangler and try playing with the tool.

## Tasks:

1. For the synsets data set, use the Data Wrangler tool to generate a list of word-meaning pairs. The output should look like:

        'hood,(slang) a neighborhood
        1530s,the decade from 1530 to 1539
        ...
        angstrom,a metric unit of length equal to one ten billionth of a meter (or 0.0001 micron)
        angstrom, used to specify wavelengths of electromagnetic radiation
        angstrom\_unit,a metric unit of length equal to one ten billionth of a meter (or 0.0001 micron)
        angstrom\_unit, used to specify wavelengths of electromagnetic radiation
        ...

2. For the FIFA dataset, use the tool to generate output as follows.

        Brazil, 1962, 1
        Brazil, 1970, 1
        Brazil, 1994, 1
        Brazil, 2002, 1
        Brazil, 1958, 1
        Brazil, 1998, 2
        Brazil, 1950, 2
        ...

i.e., each line in the output contains a country, a year, and the position of the county in that year (if within top 4).


NOT EDITED BEYOND THIS

5. now use wrangler to extract the structured content.  What was easy to do?  What was difficult?
6. dump the structured content into sqlite3 or postgresql
1. run some queries on the extracted text to prove you've done it.


Some tips:

1. Undo tool
1. Export tool

# Grep, Sed & Awk

The set of three UNIX tools, `sed`, `awk`, and `grep`, can be very useful for quickly cleaning up and transforming data for further analysis
(and have been around since the inception of UNIX). 
In conjunction with other unix utilities like `sort`, `uniq`, `tail`, `head`, etc., you can accomplish many simple data parsing and cleaning 
tasks with these tools. 
You are encouraged to play with these tools and familiarize yourselves with the basic usage of these tools. However, there is no explicit 
deliverable in this lab.

As an example, the following sequence of commands can be used to answer the third question from the (lab 2)[../lab2/] ("Find the five uids that have tweeted the most").

	grep "created\_at" twitter.json | sed 's/"user":{"id":\([0-9]*\).*/XXXXX\1/' | sed 's/.*XXXXX\([0-9]*\)$/\1/' | sort | uniq -c | sort -n | tail -5

The first command (`grep`) discards the deleted tweets, the `sed` commands extract the first "user-id" from each line, `sort` sorts the user ids, and `uniq -c` counts the unique entries (i.e., user ids). The final `sort -n | tail -5` return the top 5 uids.
Note that, combining the two `sed` commands as follows does not do the right thing -- we will let you figure out why.

	grep "created\_at" twitter.json | sed 's/.*"user":{"id":\([0-9]*\).*/\1/' | sort | uniq -c | sort -n | tail -5"

To get into some details:

## grep

The basic syntax for `grep` is: 

	 grep 'regexp' filename

or equivalently (using UNIX pipelining):

	cat filename | grep 'regexp'

The output contains only those lines from the file that match the regular expression. Two options to grep are useful: `grep -v` will output those lines that
*do not* match the regular expression, and `grep -i` will ignore case while matching. See the manual (`man grep`) (or online resources) for more details.

## sed
Sed stands for _stream editor_. Basic syntax for `sed` is:

	sed 's/regexp/replacement/g' filename

For each line in the intput, the portion of the line that matches _regexp_ (if any) is replaced with _replacement_. Sed is quite powerful within the limits of
operating on single line at a time. You can use \\( \\) to refer to parts of the pattern match. In the first sed command above, the sub-expression within \\( \\)
extracts the user id, which is available to be used in the _replacement_ as \1. 


## awk 

Finally, `awk` is a powerful scripting language (not unlike perl). The basic syntax of `awk` is: 

	awk -F',' 'BEGIN{commands} /regexp1/ {command1} /regexp2/ {command2} END{commands}' 

For each line, the regular expressions are matched in order, and if there is a match, the corresponding command is executed (multiple commands may be executed
for the same line). BEGIN and END are both optional. The `-F','` specifies that the lines should be _split_ into fields using the separator _,_, and those fields are available to the regular
expressions and the commands as $1, $2, etc. See the manual or online resources for further details. 

## Comparing to Data Wrangler

The above tools can do many of the things that Data Wrangler enables you to do. E.g., most of the _map_ operations in Data Wrangler directly map to the tools above.

## Examples 

A few examples to give you a flavor of the tools and what one can do with them.

1. _wrap_ on labor.csv (i.e., merge consecutive groups of lines referring to the same record)

    	cat labor.csv | awk '/^Series Id:/ {print combined; combined = $0} 
                            !/^Series Id:/ {combined = combined", "$0;} '
    	                    END {print combined}'

1. On the crime data, the following command does _fill_ (first row of output: "Alabama, 2004, 4029.3".

    	cat crime.txt | grep -v '^,$' | awk '/^[A-Z]/ {state = $4} !/^[A-Z]/ {print state, $0}'
    
1. Wrangle the crimes data as done in the Wrangler demo. The following works assuming perfectly homogenous data (as the provided dataset is).

    	cat crime.txt | grep -v '^,$' | sed 's/,$//g; s/Reported crime in //; s/[0-9]*,//' | 
            awk -F',' 'BEGIN {printf "State, 2004, 2005, 2006, 2007, 2008"} 
                /^[A-Z]/ {print c; c=$0} 
                !/^[A-Z]/ {c=c", "$0;} 
                END {print c}'

1. Same as above but allows the data to contain incomplete information (e.g., some years may be missing).

    	cat crime.txt | grep -v '^,$' | sed 's/Reported crime in //;' | 
                awk -F',' 'BEGIN {printf "State, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008"} 
                           /^[A-Z]/ || /^$/ {if(state) {
                                        printf(state); 
                                        for(i = 2000; i <= 2008; i++) {
                                               if(array[i]) {printf("%s,", array[i])} else {printf("0,")}
                                        }; printf("\n");} 
                                     state=$0; 
                                     delete array} 
                          !/^[A-Z]/ {array[$1] = $2}'

We provided the last example to show how powerful `awk` can be. However if you need to write a long command like that, you may be better
off using a proper scripting language like `perl` or `python`.
    
## Tasks:

Perform the above cleaning tasks using these tools. QUESTION: Should we provide a portion of the command for the second one?


NOT EDITED BEYOND THIS

2. use sed/awk to extract out the descriptions of the events, the tags, and the hours
3. what's the most popular 1/2-grams?
4. what are the most popular hours?  for partying?



### Questions

1. What does fold/un-fold do?


**Handing in your work**:

Your task is to write a query that XXXX.

You should create a text file with your name, XXX.  Upload it to the [course Stellar site](http://stellar.mit.edu/S/course/6/fa13/6.885/) as the "lab3" assignment.

Now you're almost done!  Go read the assigned paper(s) for today.

You can always feel free to email us with questions at [6885staff@mit.edu](mailto:6885staff@mit.edu).