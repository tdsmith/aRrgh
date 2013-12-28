[<- Back to index](index.html)

# Atomic vectors
Jesus Christ, here we go.

> Important, re: notation! When you see a reference to a vector, the writers may well be talking specifically about an **atomic** vector. There is another important data type called a list or **generic** vector, with (naturally) different semantics. Lists are also vectors, but lists are not atomic vectors.

The atomic vector is the simplest R data type. Atomic vectors are linear vectors of a single primitive type, like an STL Vector in C++. There is no useful literal syntax for this fundamental data type. To create one of these stupid beasts, assign like:

    a <- c(1,2,3,4)

Haha, what is `c()`? It is a function, "c" means "concatenate," and it assembles the vectors you pass into it end-to-end. "But I passed in numerical primitives," you might think. Wrong! All naked numbers are double-width floating-point atomic vectors of length one. You're welcome. Consequences of this include:

* `a`, above, is a double-typed atomic vector.
* `is.integer(2)` yields FALSE, because `2` is interpreted as a floating-point value. This has implications for testing equality! You can type an integer literal by suffixing `L`, as in `2400L`.
* `is.integer(as.integer(c(1,2)))` yields TRUE, because you gave it an integer atomic vector.

Index this like a[1] ... a[4]. **All indexing in R is base-one.** Note that **no error is thrown** if you try to access a[0]; it always returns an atomic vector of the same type but of length zero, written like `numeric(0)`. Unaccountably, nobody's in jail for that decision, yet. Indexing past the end of the array, by contrast, yields NA. Assigning past the end of the vector (i.e. `a[10] <- 5`) works and extends the vector, filling with NA. (To get an zero-filled vector of a particular length and type to start with, write something like `a <- integer(42)`.)

Zero-length vectors like `numeric(0)` have undefined truth value, and testing the truth value raises an error:

    > if(numeric(0)) { print("Truth!"); } else {print("Folly.");}
    Error in if (numeric(0)) { : argument is of length zero

## Kinds of atomic vectors

* logical (may contain TRUE, FALSE, NA)
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

## Logic values
`TRUE`, `FALSE`, and `NA` are special logic values. `NULL` is also defined and is a special vector of length zero. Do not use T and F for `TRUE` and `FALSE`. You will see people doing it but they're assholes and not your friend; T and F are just variables with default values. Set `T <- F` and source their code and laugh as it burns.

This also means that you shouldn't ever assign useful quantities to variables named T and F. Sorry. Other variable names that you cannot use are c, q, t (!), C, D, and I. :(

NA means "not available" and is a filler quantity for missing values. The result of all comparisons with `NA` is `NA`. Use `is.na(x)` to test whether a value is NA, not `x == NA`. `NA` has undefined truth value, and testing it raises an error:

    > if(NA) print ("Hello");
    Error in if (NA) print("Hello") : missing value where TRUE/FALSE needed

NULL, by the way, also has undefined truth value, raising an error if you test it:

    > if(NULL) print("Nope");
    Error in if (NULL) print("Nope") : argument is of length zero

If you need to test the truth value of some `x` that may sometimes be `NA` or have zero length, you can test the charming and ever-so-concise expression `identical(TRUE, as.logical(x))`, which will always evaluate to true or false.

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

You can do vector math in R, which always operates elementwise, like the dot operators in MATLAB. R will not do linear algebra unless you explicitly ask it to. Vector math is fast and dangerous. Almost nothing you can do with vector math will raise an error. If your operands are different sizes, R will silently recycle your short vector until it's long enough to perform the operation, which is fucking _ghastly_. R will, at least, raise a warning if your short vector does not fit into your long vector an integer number of times; fear it.

# Arrays
Atomic vectors are extended to multiple dimensions as arrays. A matrix is a two-dimensional array. One-dimensional arrays are possible; the primary difference between a one-dimensional array and a vector is that `dim(some.array)` will have length 1 and `dim(some.vector)` will be `NULL`.

[<- Back to index](index.html)
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-6770445-5', 'tim-smith.us');
  ga('send', 'pageview');

</script>