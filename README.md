# aRrgh: a newcomer's (angry) guide to data types in R
[Tim Smith](http://tim-smith.us) \<arrgh@tim-smith.us\>, [@biotimylated](http://twitter.com/biotimylated)

Kevin Ushey \<kevinushey@gmail.com\>, [@kevin_ushey](http://twitter.com/kevin_ushey)

# Introduction
R is a shockingly dreadful language for an exceptionally useful data analysis environment. The more you learn about the R language, the worse it will feel. The development environment suffers from literally decades of accretion of stupid hacks from a community containing, to a first-order approximation, zero software engineers. R makes me want to kick things almost every time I use it.

But there are a lot of great tools that are built in R. Bioconductor and ggplot2 are first-in-class. Sometimes there's aught to do but grin and bear (though never without a side of piss and moan).

The documentation is inanely bad. I can't explain it. aRrgh is my attempt to explain the language to myself. aRrgh exists as a living document and will continue to grow -- it is not complete, but it got to a point where it seemed like it was probably useful so I decided to toss it on the web. It should be correct and it's a bug if it isn't. Publishing on Github is an experiment; bug reports and pull requests are welcome and preferred over email when appropriate.

The goal of the document is to describe R's data types and structures while offering enough help with the syntax to get a programmer coming from another, saner language into a more comfortable place.

# Basic syntax and gotchas

* `;` or newline separate commands.
* Use hash-comments (`#` to end of line).
* Variable typing is weak and dynamic; variables are not declared before use. Like PHP and Javascript, variables have function (not block) scope.
* Whitespace is meaningless, unless it isn't. Some [parsing ambiguities](http://shape-of-code.coding-guidelines.com/2012/02/29/parsing-r-code-freedom-of-expression-is-not-always-a-good-idea/) are resolved by considering whitespace around operators. See and despair: `x<-y` (assignment) is parsed differently than `x < -y` (comparison)! Some might call this R's passive aggressive way of enforcing intelligent white-space usage in producing source code.
* Speaking of which, assignment looks stupid. I shit you not; these all have the equivalent effect of storing the value of `b` in `a`: `a <- b; b -> a; assign("a", b); a = b;`. There are [subtle differences](http://stackoverflow.com/questions/1741820/assignment-operators-in-r-and) and [some authorities](http://google-styleguide.googlecode.com/svn/trunk/google-r-style.html#assignment) prefer `<-` for assignment but I'm increasingly convinced that they are wrong; `=` is always safer. R doesn't allow expressions of the form `if (a = testSomething())` but does allow `if (a <- testSomething())`. To assign to globals, though, use `<<-`.
* Dots in identifier names are just part of the identifier. They are not scope operators. They are not operators at all. They are just a legal character to use in the names of things. They are often used where a normal human being would use underscores, since underscores were assignment operators in S, which I promise you don't even want to think about.
* If you squint, `$` acts kind of like the `.` scope operator in C-like languages, at least for data frames. If you'd write `struct.instance_variable` in C, you'd maybe write `frame$column.variable` in R. But be careful - `$` tries to do partial matching of names if you supply a name it cannot find, but returns `NULL` if multiple elements match. Eg: `x <- data.frame(var1="a", var123="b"); x$var12` will return `var123`. Somewhat thankfully, `x$var1` will still return `var1`.
* Sequence indexing is base-one. Accessing the zeroth element does not give an error but ~~is never useful~~ is, at least, a zero-length vector of the same type as the parent vector. More on this in the "Atomic vectors" section.
* Be careful with `for` loops. The syntax is vaguely Pythonic: `for(i in <sequence>) { do something; }`. You may be tempted to use the sequence operator, `:`, which is akin to `range()` in Python, to generate a list of integers to iterate over. Two cautions here. First, this is rarely the R idiom to use; as in MATLAB, vector operations are usually faster and harder to screw up than iteration (see **important caveat** in "vector operations" below). Reference the third chapter of the [R inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf) for advice on vectorizing. Second, if you do something like `i in 1:foo`, the **wrong thing** will happen if `foo` ever holds the value 0. `1:0` is the sequence (1, 0), since the `:` operator can and will count backwards. Always check whether `foo` is zero before you run your loop if you use `:`. If you're iterating over the indices of a sequence, always use `seq_along(x)` in preference to `1:length(x)`. If you need to do something for-loop-y, consider the family of `apply` functions, or a package like `plyr`.
* Execute external code in the current workspace using `source('filename.R')`.
* Pull in a library with `library(foo)` or `library('foo')` or `require(foo)` or `require('foo')`. `library()` is actually more stringent, and dies on an error if the library can't be found. The return value of `require()` is `TRUE` if loading the library was successful but failure to load the library is a warning and isn't fatal.
* Otherwise, fundamentals are just C-like enough to lull you into a false sense of security.
* Deep down, everything in R is a function, though. R is pretty scheme-y and LISP-y under the hood. One can write `"+"(2, 3)` to call the `+` operator on numbers 2 and 3, for example! Even crazier, `"for"( i, 1:10, print(i^2) )` shows that the keywords are functions themselves.

# Helping yourself

The R interpreter's built-in help feature is the only place I can consistently find documentation on anything. Try: `?function_name` or `??search_term`. 

Because even R's name is stupid, it's really hard to google R things in a useful way. There is a tool called RSeek that pretends to help but it just returns different useless results. Sorry. Welcome to R! Your life is now hell. [Stack Overflow](http://stackoverflow.com/questions/tagged/r) and the [R-Help](https://stat.ethz.ch/mailman/listinfo/r-help) mailing lists are pretty active, though. Be warned - R programmers can be especially surly if you ask an ill-posed question.

The [Hyperpolyglot](http://hyperpolyglot.org/numerical-analysis) page comparing MATLAB, R, and NumPy syntax is invaluable. John D. Cook's [R programming for those coming from other languages](http://www.johndcook.com/R_language_for_programmers.html) page is brief but useful. The [R inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf) offers complementary advice but blames the victim.


# Boolean primitives and special values
`TRUE`, `FALSE`, and `NA` are special logic values. (`NULL` is also defined and is a special vector of length zero.) Do not ever use T and F for `TRUE` and `FALSE`. You will see people doing it but they're assholes and not your friend; T and F are just variables with default values. Set `T <- F` and source their code and laugh as it burns. 

Secure yourself a special place in hell with `"+" <- function(x, y) { sum(x, y, 2*.Machine$double.eps) }`. Yes, you can re-define how the `+` operator works in the global environment. 

You can also re-define `for`, `if`, and all the others. R lets you ruin yours / others lives if you really want to.

    > "for" <- function(x, y, z) { return( runif(1) ) }
    > for( i in 1:10 ) { print(i) }
    [1] 0.797457

This also means that you shouldn't ever assign useful quantities to variables named T and F. Sorry. Other variable names that you cannot use are c, q, s, t (!), C, D, and I. Or, if you do it, do it inside a local scope / environment / in a function, so you don't mask these globally.

NA means "not available" and is a filler quantity for missing values. The result of all comparisons with `NA` is `NA`. Use `is.na(x)` to test whether a value is NA, not `x == NA`. `NA` has undefined truth value, and testing it raises an error:

    > if(NA) print ("Hello");
    Error in if (NA) print("Hello") : missing value where TRUE/FALSE needed

NULL, by the way, also has undefined truth value, raising an error if you test it:

    > if(NULL) print("Nope");
    Error in if (NULL) print("Nope") : argument is of length zero

The slightly awkward way of getting around `NA` and `NULL` values in `if` statements is to use the `isTRUE` function:

    > isTRUE(NA); isTRUE(NULL)
    [1] FALSE
    [1] FALSE

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

`isTRUE` can be helpful again here.

## Kinds of atomic vectors

* logical (contains TRUE, FALSE, NA)
* integer
* double (`real` is a deprecated alias, `numeric` is a synonym. It is a historical anomaly that R has three names for its floating-point vectors, double, numeric and real. See `?"numeric"` for more details.)
* complex (as in complex numbers; write as `0+0i`)
* character (pronounced "string" -- see next section)
* raw (for bitstreams; printed in hex by default. Logical operators magically operate bitwise on these.)

Integer and double atomic vectors are both numeric atomic vectors, i.e. `is.numeric(x)` is `TRUE`. Complex atomic vectors, duh???, are not numeric.

If you ask for a `numeric` vector using `numeric(42)` or `as.numeric(x)`, you will get a `double` vector. A perfect R-ism is that if you ask for a `single` vector, you'll still get a double-precision float vector, though it will have a flag set so that it will be passed into C APIs as single-width `float`s instead of `double`s. There is no single-precision storage type in R.

Check the type of your vector with `typeof(x)`, which returns a string. Note that this is not the same as `class(x)`.

A potential source of mischief is that if you try to place a value of a particular type into an atomic vector of a different type, R will—silently, natch—recast either the value you are trying to add or the entire vector (!) to the more permissive type. Witness:

    > a <- c(1L, 2L, 3L); typeof(a)
    [1] "integer"
    > a[1] = 2; typeof(a)
    [1] "double"
    > a[1] = '2'; typeof(a)
    [1] "character"


## Dealing with strings
When you see "character atomic vector" you should think "string atomic vector." `length('foo bar')` yields 1 because you have created a character atomic vector of length one, containing the character value 'foo bar'. (Yes. I know.) `length(c('foomp', 'barb', 'bazzle'))` is 3.

String primitives, which is to say the elements of a character atomic vector, are immutable.

Some other things that are true:

* `length('foo')` is 1 (see above).
* `nchar('foo')` is 3.
* Strings are indexed with `substr(x, start, stop)`. Base one, remember: `substr('foo', 1, 1)` is 'f'. `substr('foo', 2, 3)` is 'oo'.
* You can wrap strings in either single or double quotes. Escape with backslashes as in C, e.g. `'Tim\'s bad attitude.'`
* `paste()` is useful for a variety of string-concatenation operations. There is also a `sprintf()` function.
* You can almost pretend that you're working with a string if you use `unlist( strsplit(x, "") )` on a 'character vector of length 1'. Eg: `x <- "apple"; x <- unlist( strsplit( x, "" ) ); x[3:5]`. This is, unfortunately, quite slow and memory inefficient for large strings.

The `stringr` or `Biostrings` packages may ease the pain of string handling in R. Alternatively, get your feet wet and use Rcpp and use C++ code to do these things fast. I'd be more inclined to write anything substantial in Python, but horses for courses, etc.

## Vector operations

You can do vector math in R, which always operates elementwise, like the dot operators in MATLAB. R will not do linear algebra unless you explicitly ask it to. Vector math is fast and dangerous. Almost nothing you can do with vector math will raise an error. If your operands are different sizes, R will silently recycle your short vector until it's long enough to perform the operation, which is fucking ~~_ghastly_~~ _awesome_. (Presumably this is because of the pleasant effect it has in the case where one of of the operands is a "primitive" of length one.) R will, at least, raise a warning if your short vector does not fit into your long vector an integer number of times; fear it.

# Arrays
Atomic vectors are extended to multiple dimensions as arrays. A matrix is a two-dimensional array. One-dimensional arrays are possible; the primary difference between a one-dimensional array and a vector is that `dim(some.array)` will have length 1 and `dim(some.vector)` will be `NULL`.

# Indexing
When indexing matrices (or 2D arrays), we typically think of it in terms as `x[<row>,<col>]`. Some tricky yet useful indexing rules:

* If we leave `<row>` empty and specify `<col>`, we are essentially saying "give me _all_ of the rows for the given `<col>`s." Eg: `x[, c(1,2)]` gives columns 1 and 2 of `x`. Vice-versa for leaving `<col>` empty.
* We can pass in boolean vectors. In these situations it is helpful to internally translate `[` to the 'such that' operator. Eg: `x[ x[,1] > 4, ]` means "give me all the rows such that the entry in the first column is greater than 4 (and from those rows, get all the columns)".
* R will down-convert any element subsetted in this way to a 1-dimensional vector. Be very careful. Prevent it with the `drop=FALSE` argument.

See:

    > dat <- matrix(1:4, nrow=2, ncol=2)
    > typeof(dat)
    [1] "integer"
    > dim(dat)
    [1] 2 2
    > dim( dat[,1] )
    NULL
    > dim( dat[,1,drop=FALSE] )
    [1] 2 1
    
Higher-dimensional arrays are available as well, and indexing rules follow in a similar way. Constructing and using them is occasionally helpful (eg: 3D arrays as `k` `n` by `m` contigency tables):

    > x <- rep(1:2, each=10); y <- rep(1:2, times=10); z <- rep(1:4, each=5)
    > tab <- table(x, y, z)
    > dim( table(x, y, z) )
    > tab[1,,] ## returns the table of y, z values for which x == 1
           z
    y   1 2 3 4
      1 3 2 0 0
      2 2 3 0 0

# Data frames

Data frames are far and away the most useful structures in R in routine use for handling tabular data. They are a type of list vector, about which more later, so don't think about it yet. They act a little bit like a matrix. You've already exhausted many of the surprising parts of R by the time you get to data frames, so they're pretty well-behaved, considering.

A feature of R that is useful for pedagogy but otherwise unholy is that the default search path (at least for my install!) is polluted with example datasets from the `datasets` package. Run `data()` at the interpreter to take a peek. Most of these are data frames. I use the `Formaldehyde` frame below. Check out what's in your search path with `search()`. Whenever you call a variable reference, R searches through these 'environments', in order from first to last.

If anyone ever suggests that you use `attach()` while working with data frames, consider them pityingly for a moment and proceed as before. Do not take their advice.

## How is frame formed?
### From text data

The `read.table()` function, which is used to read data from delimited text files, gives you data frames. The syntax you will almost always want to use to get data into R is `read.table('path/to/file', header=TRUE, sep='\t')` (substituting your favorite separator, of course). `read.table` isn't the zippiest option for giant data sets but don't worry about it until you're sure you're already worried about it.

`read.table` will ignore lines starting with a `#` as well, by default.

If they don't already have them, you should give your data files column headers that are explanatory but easy to type, because it makes a lot of the access semantics for data frames much nicer.

R will silently interpret any `NA` in your data as a missing value. Be careful if you're working with data from North America. Remove this behavior by setting `na.strings=NULL`. Or if you have a database of names in capitals and you know a guy named 'Na'. Similarly, watch out for Mr. and Mrs. NaN, and uncle Inf. You will need `colClasses="character"` for these potential columns.

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
    
### From some text

You can also use the `text` argument, or create a `textConnection()` object. Try `read.table( text="a b c\nd e f", sep=" ")`. Or `x <- "a b c\nd e f"; read.table( textConnection(x) )`.
    
## Getting to know your data frame
There are a few functions which you should use extensively in figuring out what, exactly, your data frame looks like internally.
* `str` gives you the 'structure' of your data frame, and prints in a nice format the number of observations (rows), number of variables (columns), the variables within and their internal types. It is one of the most useful functions you will ever use.
* `head` and `tail` can give you a quick look at the heads and tails of your data frame.

eg:

    dat <- data.frame( x=c(1, 2, 3), y=c(1L, 2L, 3L), z=letters[1:3] )
    ## yes, that's right, 'letters' is a builtin vector of letters. R supremacy.
    ## LETTERS is a vector of capital letters, btw.
    
    str(dat)
    'data.frame':  3 obs. of  3 variables:
     $ x: num  1 2 3
     $ y: int  1 2 3
     $ z: Factor w/ 3 levels "a","b","c": 1 2 3
    
    ## what the hell is a 'factor', I thought I was putting in a character vector!?
    ## more on that later.
    
## Accessing data in frames
Frames can use MATLAB-like matrix addressing, as `frame[row,col]`. More conveniently!, `frame$colname` gives you the column vector `colname` (as a vector); `frame[foo,]` gives you the `foo`th row (as a single-row data frame). You can index both thusly: `frame$colname[foo]`. We also get some 'extra' subsetting operators, which involves using `[` sans any comma operator, and `[[`.

Be aware of the difference between `frame[foo]`, `frame[[foo]]`, and `frame[foo,bar]`. The former returns one or more columns from `frame` with names matching `foo`, and kindly does _not_ coerce the type even if only one vector is taken. The middle can and will return only _one_ item from `frame`, and returns it with that vector's type. The last will silently coerce the type if you are returning a single column, eg: `frame[ ,1]` will return the first column _not_ as a data.frame. You can prevent this with eg. `frame[,1,drop=FALSE]`

Moral of the story? If you want to subset columns from a data frame, use `frame[<names>]`, _not_ `frame[,c(<names>)]`.

You can see a list of columns with `names(frame)`. You rename columns by, spookily, assigning into `names(frame)`. This is because R secretly defines a `names<-` function, which is called and internally assigns names when caling eg. `names(frame) <- c("foo", "bar")`. That call is identical to `frame <- ``names<-``(frame, c("foo", "bar"))`. Almost all assignment operators act like this under the hood.

**R is secretly just an infixed LISPy language where everything is an object and keywords are just functions**

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

## Adding data to data frames
Adding columns is easy: `a <- Formaldehyde; a$bar <- seq(6)` creates a new column `bar`. Refreshingly, `a$bar <- seq(100)` fails, *BUT* `a$bar <- 1:2` works silently, repeating the sequence (1,2) down the column, so fuck everything. You do get an error if you assign a vector whose length isn't a multiple of the number of the rows in the data frame, eg `a$bar <- 1:4`.

Don't add data to the frame a row at a time. It is slower than molasses and just as [deadly](http://en.wikipedia.org/wiki/Boston_Molasses_Disaster); this is the second level of the [R inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf). Create your data frame from lists that are big enough to contain all of the data you expect the frame to contain, perhaps filling with `NA`. You may, however, merge two data frames vertically using `rbind()`.

## Removing columns from data frames
Now we can get frustrated. There isn't really a great syntax for 'removing' vectors from a data frame. A dumb workaround is to take every vector except the one we want to remove. An annoying way of being explicit is also illustrated here.

    > dat <- data.frame(x=1, y=2, z=3)
    > wanted_vectors <- c("x", "z")
    > get_rid_of <- "y"

    ## what if we want to remove column 2?
    > dat[ ,c(1,3)] ## get the 1st and 3rd columns
      x z
    1 1 3
    > dat[ ,c("x","z")] ## get the columns named "x" and "z"
      x z
    1 1 3
    > dat[ wanted_vectors ]
      x z
    1 1 3
    > dat[ !(names(dat) %in% "y") ] ## the terrible syntax for explicitly removing 'y'
      x z
    1 1 3
    > ## ie, 'give me the columns in dat such that their names aren't in the vector "y"'

# Lists

Lists are data frames where each vector can have different lengths. Or maybe it's better to describe data frames as lists where we restrict each vector to be of the same length. Lists can be named, can contain other lists, and elements within a list are ordered and numerically indexable. So they're like super-duper associative arrays. They're slow, bulky, and incredibly useful. Almost everything returned by more complex R functions, especially model-fitting functions, will be a big list full of things.

    > x <- 1:100; y <- x + runif(100)
    > dat <- lm( y ~ x )
    > str(dat)
    List of 12
     $ coefficients : Named num [1:2] 0.4 1
      ..- attr(*, "names")= chr [1:2] "(Intercept)" "x"
     $ residuals    : Named num [1:100] -0.2867 0.22725 -0.00831 0.57307 -0.15037 ...
      ..- attr(*, "names")= chr [1:100] "1" "2" "3" "4" ...
    ## etc, etc...
    
Take everything you know about data frames, except the columns are just elements of a list, and there is no internal concept of a row.
    
# Factors

Factors are 'special' character vectors. They're very 'special'.

First, let's talk about how to _avoid_ factors; not because they're not useful, but because they're incredibly frustrating when your data is silently coerced to a factor when you really thought you were working with a character vector.

Factors are responsible for 99% of indecipherable errors. They're very useful when you explicitly want and construct them as you need them, and terribly frustrating when they randomly appear because you called `data.frame`, `as.data.frame` or `read.table` and forgot that you didn't really want your character vectors to be silently coerced into factors.

    > dat <- data.frame( x=c(1,2,3), y=c("z","b","c") )
    ## oops - I want to set the first element in 'y' to 1. Let's do this!
    > dat$y[1] <- "a"
    Warning message:
    In `[<-.factor`(`*tmp*`, "a", value = c(NA, 1L, 2L)) :
      invalid factor level, NAs generated
    
    ## wat
    
    ## R silently turned that thing you thought was a character vector into a factor
    
Every R installation should come with `options(stringsAsFactors=FALSE)` as default. Unfortunately this isn't the case so you'll probably see all your supposed character vectors be silently converted into factors until you learn this terrible quirk:

    > dat <- data.frame( x=c(1,2,3), y=c("x","y","z") )
    > str(dat)
    'data.frame':  3 obs. of  2 variables:
     $ x: num  1 2 3
     $ y: Factor w/ 3 levels "x","y","z": 1 2 3
     
    ## prevent automatic factor generation
    > dat <- data.frame( x=c(1,2,3), y=c("x","y","z"), stringsAsFactors=FALSE )
    > str(dat)
    'data.frame':  3 obs. of  2 variables:
     $ x: num  1 2 3
     $ y: chr  "x" "y" "z"
    
    ## prevent it globally
    > options( stringsAsFactors=FALSE )
    > dat <- data.frame( x=c(1,2,3), y=c("x","y","z") )
    > str(x)
    'data.frame':  3 obs. of  2 variables:
     $ x: num  1 2 3
     $ y: chr  "x" "y" "z"
    
Note that setting `options(stringsAsFactors = FALSE)` will only persist throughout the current R session; the setting is removed as soon as you close R.

## What -is- a factor?

Under the hood, factors are actually integer vectors, with 'labels' that we call levels, to define in some sense the 'name' of a particular integer.

    > x <- factor( c("a", "b", "c") )
    > levels(x)
    [1] "a" "b" "c"
    > unclass(x) ## prints the internal representation of x
    [1] 1 2 3
    attr(,"levels")
    [1] "a" "b" "c"
    > str( unclass(x) )
     atomic [1:3] 1 2 3
     - attr(*, "levels")= chr [1:3] "a" "b" "c"
    
So factors are really just integers with this useful 'levels' attribute. 

SIDENOTE: Wait, what's an attribute? They're like the `names` of data.frames and such. All of these 'descriptive' variables are just 'attributes' that are handles in some special way.

    > dat <- data.frame( x=c(1, 2, 3), y=c("a", "b", "c") )
    > attributes(dat)
    $names
    [1] "x" "y"

    $row.names
    [1] 1 2 3

    $class
    [1] "data.frame"
    
    ## Let's make a 'special' data.frame with a 'name' attribute, just for fun.
    > attr(dat, "name") <- "hello!"
    
    ## let's change the column names in an ugly way
    > attr(dat, "names") <- c("foo", "bar")
    > print(dat)
      foo bar
    1   1   a
    2   2   b
    3   3   c
    
    ## so really, 'attributes' are just 'slots' for useful descriptors. you can
    ## break R objects by arbitrarily assigning over these if you really want.
    
So when are factors good?
* Generating tables with `table`.
* Treating a character vector as a categorical variable and explicitly controlling a 'reference' level.
* Plotting functions often handle factors in a `smart` way, over character variables.

## Subsetting a Factor

Be careful when subsetting a factor. Subsetted factors retain their levels. Often-times, this is not the desired behavior.

    > x <- factor( c("a", "b", "c") )
    > levels(x)
    [1] "a" "b" "c"
    > y <- x[1]
    > levels(y)
    [1] "a" "b" "c"
    > print(y)
    [1] a
    Levels: a b c
    
    > table(x)
    x
    a b c 
    1 1 1 
    
    > table(y)
    y
    a b c 
    1 0 0 
    
We can get rid of extraneous levels (that is, for elements that no longer exist in the factor) with `droplevels` (only in R >2.15.0):

    > y <- droplevels(y)
    > print(y)
    [1] a
    Levels: a
    
By default, the first level of a factor is interpreted as a 'reference' level, which is typically used in model-fitting functions, eg. `lm`. We can change the reference level with `relevel`:

    > x <- relevel(x, ref="b")
    > print(x)
    [1] a b c
    Levels: b a c

If you want to change the order of levels in a factor, just make a new factor. It is dangerous to just re-assign the levels.

    > print(x)
    [1] a b c
    Levels: b a c
    
    > xx <- x ## make a copy of x
    > levels(xx) <- c( "c", "b", "a" )
    > print(xx)
    [1] b c a 
    Levels: c b a
    
    ## wat
    
    ## why is xx now printing as 'b c a'!?
    
It seems like R has silently reordered the variables in our vector. That was very sweet of it to do so. But really, nothing has happened to the underlying vector representation of our variable - we just remapped those hidden internal integers to different levels. Regardless of the intention or result, it's almost always confusing.

    > unclass(x)
    [1] 2 1 3
    attr(,"levels")
    [1] "b" "a" "c"
    
    > unclass(xx)
    [1] 2 1 3
    attr(,"levels")
    [1] "c" "b" "a"
    
So, save yourself the grief and just generate a new factor with levels in the desired order.
    
    > z <- factor( as.character(x), levels=c("c", "b", "a") )
    
Moral of the story: **only use factors when you explicitly mean to, and stop R from coercing your character vectors into factors when you don't.**

# Colophon
© Tim Smith 2012. This work is made available under a [Creative Commons Attribution-ShareAlike 3.0 Unported](http://creativecommons.org/licenses/by-sa/3.0/deed.en_US) License. Some additional content added by Kevin Ushey.