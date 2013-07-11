# Regular Expressions

## Introduction
Regular expressions are a powerful way (read: [language](http://en.wikipedia.org/wiki/Regular_language)) to express text patterns.  The goal of this presentation is to give you a high level knowledge about what regular expressions are, why they're useful, and ways to make use of them from the command line.  However with great power...Regular expressions can be very useful, but also very difficult to interpret.

We'll be working with some example data related to the nirK gene, a set of [annotations](nirk_annotations.txt) and [sequences](nirk_seqs.fa) from the [FunGene Repository](http://fungene.cme.msu.edu).

Commonly abbreviated as regex, regexp, re.

*NOTE* Regular expressions are _CASE SENSITIVE_

### A few common applications
* Input validation
  * Phone numbers
  * Email
* Text parsing
  * area code from a phone number [ex: (xxx) xxx-xxxx]
  * Dates [ex: xx/xx/xxxx, xx-xx-xxx]
  * Quick and dirty html parsing
* Bioinformatics
  * Motif searching
  * Primer matching

### Where you can use regular expressions
* Most programming languages have built in support
  * Perl
  * Python
  * Java
  * ...etc etc
* Many common command line tools
  * grep
  * sed
  * awk

## Structure of a regular expression

Regular expressions are a string of characters that define a pattern.  These characters can be
* Literals: most any character (abc, 123)
* Meta-characters
  * . -- Matches any literal
  * $, ^ -- beginning of line, end of line
  * <, > -- beginning of word, end of word
  * \w \W \s \S \d -- letter, not a letter, whitespace, not whitespace, digit
* Operators
  * ? -- preceding literal is optional
  * *, + -- match the preceding literal 0/1 or more times
  * [] -- Set of literals
  * () -- Groupings
  * \ -- Allows a meta-character or operator to be used as a literal

### Example Regular Expressions

* colou?r -- Matches the two variations of spelling the word 'color'
* gr[ae]y -- Matches the two variations of the spelling of the color 'gray'
* (\d\d\d) ?\d\d\d-\d\d\d\d -- Telephone
* [a-z]+\d -- Lower case letters followed by a number (walla2)
* \d*\w\d -- 0 or more numbers followed by a single letter followed by a single number (ex: 9251a5)

## Working with regular expressions

We'll be working with egrep as an easy way of exploring the power of regular expressions.  Since grep by default outputs the entire line when it encounters matching text we'll be using the '-o' option to force grep to output only the matching text.

### Primer Matching

Suppose we have a primer, aaccacaa*y*ctgatcgaggc, and we want to find all the matching regions in our set of nirk sequences.  (Note: the IUPAC ambiguity codon Y can be either C or T).  The most straight forward way of doing this would be exact matching of the two possible strings:
> aaccacaacctgatcgaggc

> aaccacaatctgatcgaggc

which we could do with

``` bash
egrep -o aaccacaacctgatcgaggc nirk_seqs.fa
egrep -o aaccacaatctgatcgaggc nirk_seqs.fa
```

Suppose however we were interested in the primer gg*m*atggt*k*cc*s*tggca (M=A/C, K=T/G, S=G/C)...

We can use the [] operator to specify a set of characters to match at any given position, turning the above primer string in to the regular expression

> gg[ac]atggt[tg]cc[gc]tggca

``` bash
egrep -o gg[ac]atggt[tg]cc[gc]tggca nirk_seqs.fa
```

### Extracting information from annotations

Suppose we wanted to check and see how many of our nirK sequences came from the genus Azospirillum.  We could use a regular expression to match 'Azospirillum' and have egrep count the matching lines.

``` bash 
egrep -c Azospirillum nirk_annotations.txt 
```

We can use an exact text pattern for the word Azospirillum.

Now suppose instead we wanted to see the genus and species name for all sequences from the genus Azospirillum.

``` bash
egrep -o 'Azospirillum [a-z\.]+' nirk_annotations.txt
```

We know that a species name is always the genus followed by a space followed by the species epithet.


But wait, some lines contain strain information as well...

``` bash
egrep -o 'Azospirillum [a-z\.]+ [a-zA-Z0-9]+' nirk_annotations.txt
```

or

``` bash
egrep -o 'Azospirillum [a-z\.]+( [^       ]+)?' nirk_annotations.txt
```

## More regex magic!

The annotation dataset was actually made using regular expressions as well, the full command is below (bonus points: See if you can figure out what's going on!)

``` bash
grep '>' fungene_7.3_nirK_7643_unaligned_nucleotide_seqs.fa | sed 's/>//g' | tr ',' '\t' | sed -E 's/[^a-zA-Z0-9][^=]+=([^      $]+)/   \1/g' > nirk_annotations.txt
```

