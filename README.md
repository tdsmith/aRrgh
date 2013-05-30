# aRrgh: a newcomer's (angry) guide to data types in R
[Tim Smith](http://tim-smith.us) \<arrgh@tim-smith.us\>, [@biotimylated](http://twitter.com/biotimylated)

with Kevin Ushey \<kevinushey@gmail.com\>, [@kevin_ushey](http://twitter.com/kevin_ushey)

# Introduction
R is a shockingly dreadful language for an exceptionally useful data analysis environment. The more you learn about the R language, the worse it will feel. The development environment suffers from literally decades of accretion of stupid hacks from a community containing, to a first-order approximation, zero software engineers. R makes me want to kick things almost every time I use it.

But there are a lot of great tools that are built in R. Bioconductor and ggplot2 are first-in-class. Sometimes there's aught to do but grin and bear (though never without a side of piss and moan).

The documentation is inanely bad. I can't explain it. aRrgh is my attempt to explain the language to myself. aRrgh exists as a living document and will continue to grow -- it is not complete, but it got to a point where it seemed like it was probably useful so I decided to toss it on the web. It should be correct and it's a bug if it isn't. Publishing on Github is an experiment; bug reports and pull requests are welcome and preferred over email when appropriate.

The goal of the document is to describe R's data types and structures while offering enough help with the syntax to get a programmer coming from another, saner language into a more comfortable place.

# Basic syntax and gotchas

* `;` or newline separate commands.
* Use hash-comments (`#` to end of line).
* Variable typing is weak and dynamic; variables are not declared before use. Like PHP and Javascript, variables have function (not block) scope.
* Whitespace is meaningless, unless it isn't. Some [parsing ambiguities](http://shape-of-code.coding-guidelines.com/2012/02/29/parsing-r-code-freedom-of-expression-is-not-always-a-good-idea/) are resolved by considering whitespace around operators. See and despair: `x<-y` (assignment) is parsed differently than `x < -y` (comparison)!
* Speaking of which, assignment looks stupid. I shit you not; these all have the equivalent effect of storing the value of `b` in `a`: `a <- b; b -> a; assign("a", b); a = b;`. There are [subtle differences](http://stackoverflow.com/questions/1741820/assignment-operators-in-r-and) and [some authorities](http://google-styleguide.googlecode.com/svn/trunk/google-r-style.html#assignment) prefer `<-` for assignment but I'm increasingly convinced that they are wrong; `=` is always safer. R doesn't allow expressions of the form `if (a = testSomething())` but does allow `if (a <- testSomething())`. To assign to *globals*, though, use `<<-`.
* Dots in identifier names are just part of the identifier. They are not scope operators. They are not operators at all. They are just a legal character to use in the names of things. They are often used where a normal human being would use underscores, since underscores were assignment operators in S, which I promise you don't even want to think about.
* If you squint, `$` acts kind of like the `.` scope operator in C-like languages, at least for data frames. If you'd write `struct.instance_variable` in C, you'd maybe write `frame$column.variable` in R.
* Sequence indexing is base-one. Accessing the zeroth element does not give an error but is never useful. More on this in the "Atomic vectors" section.
* Be careful with `for` loops. The syntax is vaguely Pythonic: `for(i in <sequence>) { do something; }`. You may be tempted to use the sequence operator, `:`, which is akin to `range()` in Python, to generate a list of integers to iterate over. Two cautions here. First, this is rarely the R idiom to use; as in MATLAB, vector operations are usually faster and harder to screw up than iteration (see **important caveat** in "vector operations" below). Reference the third chapter of the [R inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf) for advice on vectorizing. Second, if you do something like `i in 1:foo`, the **wrong thing** will happen if `foo` ever holds the value 0. `1:0` is the sequence (1, 0), since the `:` operator can and will count backwards. Always check whether `foo` is zero before you run your loop if you use `:`. If you're iterating over the indices of a sequence, always use `seq_along(x)` in preference to `1:length(x)`.
* Execute external code in the current workspace using `source('filename.R')`.
* Pull in a library with `library(foo)` or `library('foo')` or `require(foo)` or `require('foo')`. `library()` is actually more stringent, and dies on an error if the library can't be found. The return value of `require()` is `TRUE` if loading the library was successful but failure to load the library is a warning and isn't fatal.
* If something works on your machine but not your collaborator's, ask them if they have any `option()` calls in their `.Rprofile` file. This is a way to change the default behavior of some functions and it's an awful idea, because it will change the behavior of anybody else's code (including libraries) that depends on the default settings, in a way that's really hard to debug when you forget about it.
* Otherwise, fundamentals are just C-like enough to lull you into a false sense of security.

# Helping yourself

The R interpreter's built-in help feature is the only place I can consistently find documentation on anything. Try: `?function_name` or `??search_term`.

Because even R's name is stupid, it's really hard to google R things in a useful way. There is a tool called RSeek that pretends to help but it just returns different useless results. Sorry. Welcome to R! Your life is now hell.

The [Hyperpolyglot](http://hyperpolyglot.org/numerical-analysis) page comparing MATLAB, R, and NumPy syntax is helpful. John D. Cook's [R programming for those coming from other languages](http://www.johndcook.com/R_language_for_programmers.html) page is brief but useful. The [R inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf) offers complementary advice but blames the victim.

R messiah Hadley Wickham has a [very useful wiki-book](https://github.com/hadley/devtools/wiki) on advanced R development.  The [vocabulary](https://github.com/hadley/devtools/wiki/Vocabulary) appendix is an excellent list of things to learn.

[Stack Overflow](http://stackoverflow.com/questions/tagged/r) and the [R-Help](https://stat.ethz.ch/mailman/listinfo/r-help) mailing list are good places to ask for help. There is [offputtingly dense](http://www.r-project.org/posting-guide.html) advice on asking good questions.

# Boolean primitives and special values
`TRUE`, `FALSE`, and `NA` are special logic values. (`NULL` is also defined and is a special vector of length zero.) Do not ever use T and F for `TRUE` and `FALSE`. You will see people doing it but they're assholes and not your friend; T and F are just variables with default values. Set `T <- F` and source their code and laugh as it burns.

This also means that you shouldn't ever assign useful quantities to variables named T and F. Sorry. Other variable names that you cannot use are c, q, t (!), C, D, and I. :(

NA means "not available" and is a filler quantity for missing values. The result of all comparisons with `NA` is `NA`. Use `is.na(x)` to test whether a value is NA, not `x == NA`. `NA` has undefined truth value, and testing it raises an error:

    > if(NA) print ("Hello");
    Error in if (NA) print("Hello") : missing value where TRUE/FALSE needed

NULL, by the way, also has undefined truth value, raising an error if you test it:

    > if(NULL) print("Nope");
    Error in if (NULL) print("Nope") : argument is of length zero

If you need to test the truth value of some `x` that may sometimes be `NA` or have zero length, you can test the charming and ever-so-concise expression `identical(TRUE, as.logical(x))`, which will always evaluate to true or false.

# Atomic vectors
Jesus Christ, here we go.

> Important, re: notation! When you see a reference to a vector, the writers may well be talking specifically about an **atomic** vector. There is another important data type called a list or generic vector, with (naturally) different semantics. Lists are also vectors, but lists are not atomic vectors.

The atomic vector is the simplest R data type. Atomic vectors are linear vectors of a single primitive type, like an STL Vector in C++. There is no useful literal syntax for this fundamental data type. To create one of these stupid beasts, assign like:

    a <- c(1,2,3,4)

Haha, what is `c()`? It is a function, "c" means "concatenate," and it assembles the vectors you pass into it end-to-end. "But I passed in numerical primitives," you might think. Wrong! All naked numbers are double-width floating-point atomic vectors of length one. You're welcome. Consequences of this include:

* `a` is a double-typed atomic vector.
* `is.integer(2)` yields FALSE, because `2` is interpreted as a floating-point value. This has implications for testing equality! You can type an integer literal by suffixing `L`, as in `2400L`.
* `is.integer(as.integer(c(1,2)))` yields TRUE, because you gave it an integer atomic vector.

Index this like a[1] ... a[4]. **All indexing in R is base-one.** Note that **no error is thrown** if you try to access a[0]; it always returns an atomic vector of the same type but of length zero, written like `numeric(0)`. Unaccountably, nobody's in jail for that decision, yet. Indexing past the end of the array, by contrast, yields NA. Assigning past the end of the vector (i.e. `a[10] <- 5`) works and extends the vector, filling with NA. (To get an zero-filled vector of a particular length and type to start with, write something like `a <- integer(42)`.)

`numeric(0)` has undefined truth value, and testing it raises an error:

    > if(numeric(0)) { print("Truth!"); } else {print("Folly.");}
    Error in if (numeric(0)) { : argument is of length zero

## Kinds of atomic vectors

* logical (contains TRUE, FALSE, NA)
* integer
* double (`real` is a deprecated alias)
* complex (as in complex numbers; write as `0+0i`)
* character (pronounced "string" -- see next section)
* raw (for bitstreams; printed in hex by default. Logical operators magically operate bitwise on these.)

Integer and double atomic vectors are both numeric atomic vectors, i.e. `is.numeric(x)` is `TRUE`. Complex atomic vectors, duh???, are not numeric.

If you ask for a `numeric` vector using `numeric(42)` or `as.numeric(x)`, you will get a `double` vector. A perfect R-ism is that if you ask for a `single` vector, you'll still get a double-precision float vector, though it will have a flag set so that it will be passed into C APIs as single-width `float`s instead of `double`s. There is no single-precision storage type in R.

Check the type of your vector with `typeof(x)`, which returns a string.

A potential source of mischief is that if you try to place a value of a particular type into an atomic vector of a different type, R will—silently, natch—recast either the value you are trying to add or the entire vector (!) to the more permissive type. Witness:

    > a <- c(1L, 2L, 3L); typeof(a)
    [1] "integer"
    > a[1] = 2; typeof(a)
    [1] "double"
    > a[1] = '2'; typeof(a)
    [1] "character"


## Dealing with strings
When you see "character atomic vector" you should think "string atomic vector." `length('foo bar')` yields 1 because you have created a character atomic vector of length one, containing the character value 'foo bar'. (Yes. I know.) `length(c('to be', 'or not', 'to be'))` is 3.

String primitives, which is to say the elements of a character atomic vector, are immutable.

Some other things that are true:

* `length('foo')` is 1 (see above).
* `nchar('foo')` is 3.
* Strings are indexed with `substr(x, start, stop)`. Base one, remember: `substr('foo', 1, 1)` is 'f'. `substr('foo', 2, 3)` is 'oo'.
* You can wrap strings in either single or double quotes. Escape with backslashes as in C, e.g. `'Tim\'s bad attitude.'`
* `paste()` is useful for a variety of string-concatenation operations. There is also a `sprintf()` function.


The [stringr](http://cran.r-project.org/web/packages/stringr/index.html) or [Biostrings](http://www.bioconductor.org/packages/2.11/bioc/html/Biostrings.html) packages may ease the pain of string handling in R. I'd be more inclined to write anything substantial in Python, but horses for courses, etc.

## Vector operations

You can do vector math in R, which always operates elementwise, like the dot operators in MATLAB. R will not do linear algebra unless you explicitly ask it to. Vector math is fast and dangerous. Almost nothing you can do with vector math will raise an error. If your operands are different sizes, R will silently recycle your short vector until it's long enough to perform the operation, which is fucking _ghastly_. (Presumably this is because of the pleasant effect it has in the case where one of of the operands is a "primitive" of length one.) R will, at least, raise a warning if your short vector does not fit into your long vector an integer number of times; fear it.

# Arrays
Atomic vectors are extended to multiple dimensions as arrays. A matrix is a two-dimensional array. One-dimensional arrays are possible; the primary difference between a one-dimensional array and a vector is that `dim(some.array)` will have length 1 and `dim(some.vector)` will be `NULL`.

# Indexing

# Factors

Factors are very useful and responsible for many unpleasant surprises. You should use factors to represent small sets of discrete possibilities. They have a family resemblance to `enum`s in C and, more loosely, `:symbols` in Ruby. A R factor is a sequence type much like a character atomic vector except that the values of the factor are constrained to a set of string values, called "levels". For example, if you have a table of measurements of some widgets and each row corresponds to a single measurement of a single widget, you could have a factor-typed column called measurement.type containing the values "length", "width", "height", "weight", and "hue", with the corresponding numeric measurements stored in a "value" column.

Factors are useful for a few reasons:

* they provide validation; R will throw warnings if you try to assign an undeclared level (e.g. because you miskeyed it)
* they have a more compact representation in memory than character vectors
* they provide semantic information about the structure of your data to e.g. model-fitting and plotting tools

Factors are unordered by default. The sort order of the levels of an unordered factor is determined lexiographically in the current locale. If your factors have some sort of innate order, you can create an "ordered factor" with `ordered()`. Even for nominally unordered factors, the "first" value of the factor can have semantic importance for e.g. linear regression with `lm`. If you find that you need to change the first value of an unordered factor, use the `relevel` function as `my.factor <- relevel(my.factor, 'weight')`.

Comparisons for factors look the same as they do for character atomic vectors. For example, `measurements[measurements$measurement.type == 'weight',]` will give you all of the weight rows of the measurement table.

Factors have a numeric representation underneath that you can see by calling `as.integer` against a factor. The [R language documentation](http://cran.r-project.org/doc/manuals/r-release/R-lang.html#Factors) rolls its eyes, sighs, pushes up its glasses, and stops just short of telling you not to look. Caveat emptor.

All of the levels that a factor will ever have must be defined at the time the factor is created. If you want to add new levels or remove existing levels, you have to build a new factor. If you attempt to assign a value to an element of a factor that is not one of the pre-existing levels, you will get a warning for your troubles and the element will take the value `NA`.

In keeping with R's "maximal-mystery" design philosophy, whenever a data frame is instantiated with data from a character atomic vector or with character data read from a file with `read.table`, the resulting column will be a factor instead of a character atomic vector. Passing `stringsAsFactors = FALSE` to `data.frame()` or `read.table()` will prevent this behavior and leave your character vectors alone.

# Data frames

Data frames are far and away the most useful structures in R in routine use for handling tabular data. They are a type of list vector, about which more later, so don't think about it yet. They act a little bit like a matrix. You've already exhausted many of the surprising parts of R by the time you get to data frames, so they're pretty well-behaved, considering.

A feature of R that is useful for pedagogy but otherwise unholy is that the default search path (at least for my install!) is polluted with example datasets from the `datasets` package. Run `data()` at the interpreter to take a peek. Most of these are data frames. I use the `Formaldehyde` frame below.

If anyone ever suggests that you use `attach()` while working with data frames, consider them pityingly for a moment and proceed as before. Do not take their advice.

## How is frame formed?
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

# Lists


# Colophon
© Tim Smith 2012-3. This work is made available under a [Creative Commons Attribution-ShareAlike 3.0 Unported](http://creativecommons.org/licenses/by-sa/3.0/deed.en_US) License.

If you enjoyed this, you will probably enjoy [PHP: A Fractal of Bad Design](http://me.veekun.com/blog/2012/04/09/php-a-fractal-of-bad-design/), which is even more cathartic.

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/e0799fb6c38624dd1955cfbc75756167 "githalytics.com")](http://githalytics.com/tdsmith/aRrgh)
