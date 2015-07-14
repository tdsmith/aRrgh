---
title: Data frames
layout: page
---
Data frames are far and away the most useful structures in R in routine use for handling tabular data. They are a type of list vector, about which more later, so don't think about it yet. They act a little bit like a matrix. You've already exhausted many of the surprising parts of R by the time you get to data frames, so they're pretty well-behaved, considering.

A feature of R that is useful for pedagogy but otherwise unholy is that the default search path (at least for my install!) is polluted with example datasets from the `datasets` package. Run `data()` at the interpreter to take a peek. Most of these are data frames. I use the `Formaldehyde` frame below.

If anyone ever suggests that you use `attach()` while working with data frames, consider them pityingly for a moment and proceed as before. Do not take their advice.

## Creating data frames

### From text data

The `read.table()` function, which is used to read data from delimited text files, gives you data frames. The syntax you will almost always want to use to get data into R is `read.table('path/to/file', header=TRUE, sep='\t')` (substituting your favorite separator, of course). `read.table` isn't the zippiest option for giant data sets but don't worry about it until you're sure you're already worried about it.

Some things that you should know about `read.table` are:

* String values are, by default, treated as `factor`s, *not* as character atomic vectors. If the strings in your data file describe elements of a set of discrete possibilities, this is often convenient and desirable. If you are not expecting it, this behavior may threaten to [ruin your career](https://twitter.com/seanjtaylor/status/322410764151443457). Pass `stringsAsFactors=FALSE` to leave your character vectors alone.
* Lines starting with `#` will be silently ignored.
* Any values which are the string `NA` will be understood as a missing value, unless you pass `na.strings=NULL`.
* The strings `NaN` and `Inf` will be interpreted appropriately if they appear in columns that otherwise appear to be numeric.

If they don't already have them, you should give your data files column headers that are explanatory but easy to type, because it makes a lot of the access semantics for data frames much nicer.

The [R Data Import/Export page](http://cran.stat.ucla.edu/doc/manuals/R-data.html#Spreadsheet_002dlike-data) elaborates on the above at greater length. `?read.table` is also helpful. "An Introduction to R" warns that:

> R input facilities are simple and their requirements are fairly strict and even rather inflexible. There is a clear presumption by the designers of R that you will be able to modify your input files using other tools, such as file editors or Perl to fit in with the requirements of R.

"Generally," they add, with a straight face, "[modifying your input file with Perl] is very simple." Snort. Ahem.

### From the clipboard

Grabbing data from the clipboard is slightly wacky. `read.table('clipboard',…)` works on Windows (so don't name your input files 'clipboard'). Mac users will sigh and resort to the inelegant but functional `read.table(pipe('pbpaste'),…)`. `write.table('clipboard',…)` works similarly; Mac users should do `write.table(pipe('pbcopy'),…)` to write data back. (The `pbpaste` and `pbcopy` shell commands ship with OS X and are useful in many non-R contexts!)

### From vectors in your workspace

Let's take an example data set and put it back together:

    > Formaldehyde
      carb optden
    1  0.1  0.086
    2  0.3  0.269
    3  0.5  0.446
    4  0.6  0.538
    5  0.7  0.626
    6  0.9  0.782
    > mycarb <- Formaldehyde$carb
    > OD <- Formaldehyde$optden
    > my.new.frame <- data.frame(carb=mycarb, od=OD)
    > my.new.frame
      carb    od
    1  0.1 0.086
    2  0.3 0.269
    3  0.5 0.446
    4  0.6 0.538
    5  0.7 0.626
    6  0.9 0.782
    
## Accessing data in frames
Frames use MATLAB-like matrix addressing, as `frame[row,col]`. More conveniently!, `frame$colname` gives you the column vector `colname` (as a vector); `frame[foo,]` gives you the `foo`th row (as a single-row data frame). You can index both thusly: `frame$colname[foo]`.

You can see a list of columns with `names(frame)`. You rename columns by, spookily, assigning into `names(frame)`. Do you know how and why this works? Please educate me.

    > a <- Formaldehyde
    > a
      carb optden
    1  0.1  0.086
    2  0.3  0.269
    3  0.5  0.446
    4  0.6  0.538
    5  0.7  0.626
    6  0.9  0.782
    > names(a)
    [1] "carb"   "optden"
    > names(a)[2] <- "OD"
    > names(a)
    [1] "carb" "OD"
    > a
      carb    OD
    1  0.1 0.086
    2  0.3 0.269
    3  0.5 0.446
    4  0.6 0.538
    5  0.7 0.626
    6  0.9 0.782
    > a$OD
    [1] 0.086 0.269 0.446 0.538 0.626 0.782

Would you like to know how many rows you have in your data frame? Use `nrow(foo)`. Do not use `length(foo)`, which unhelpfully tells you how many columns you have (it is describing the length of the list object containing the column vectors!); if you actually want that number, you can use `ncol(foo)` for clarity. `dim(foo)` will give you a vector containing both dimensions.

## Adding data to frames
Adding columns is easy: `a <- Formaldehyde; a$bar <- seq(6)` creates a new column `bar`. Refreshingly, `a$bar <- seq(100)` fails (since 100 does not divide 6 evenly), *BUT* `a$bar <- 1:2` works silently, repeating the sequence (1,2) down the column, so fuck everything.

Don't add data to the frame a row at a time. It is slower than molasses and just as [deadly](http://en.wikipedia.org/wiki/Boston_Molasses_Disaster); this is the second level of the [R inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf). Create your data frame from lists that are big enough to contain all of the data you expect the frame to contain, perhaps filling with `NA`. You may, however, merge two data frames vertically using `rbind()`.
