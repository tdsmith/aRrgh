---
title: Factors
layout: page
---
Factors are very useful and responsible for many unpleasant surprises. Many people find that this is the single most important thing to learn about factors (but read on!):

> In keeping with R's "maximal-mystery" design philosophy, whenever a data frame is instantiated with data from a character atomic vector or with character data read from a file with `read.table`, the resulting column will be a factor instead of a character atomic vector. Passing `stringsAsFactors = FALSE` to `data.frame()` or `read.table()` will prevent this behavior and leave your character vectors alone.[^options]

[^options]: Some people find this so offensive that they will urge you to change the default behavior of `data.frame()` by calling `options(stringsAsFactors=FALSE)` at R startup time by adding it to your ~/.Rprofile. I believe that this is misguided because it means code that doesn't explicitly provide a `stringsAsFactors` argument to `data.frame()` or `read.whatever()` will execute differently on your machine than it will execute on a default installation of R, in an action-at-a-distance way that is difficult to troubleshoot. To recall the words of the [Zen of Python](https://www.python.org/dev/peps/pep-0020/), "Explicit is better than implicit."

Factors are useful, though, to represent small sets of discrete possibilities. They have a family resemblance to `enum`s in C and, more loosely, `:symbols` in Ruby. A R factor is a sequence type much like a character atomic vector except that the values of the factor are constrained to a set of string values, called "levels". For example, if you have a table of measurements of some widgets and each row corresponds to a single measurement of a single widget, you could have a factor-typed column called measurement.type containing the values "length", "width", "height", "weight", and "hue", with the corresponding numeric measurements stored in a "value" column.

Factors are useful for a few reasons:

* they provide validation; R will throw warnings if you try to assign an undeclared level (e.g. because you miskeyed it)
* they have a more compact representation in memory than character vectors
* they provide semantic information about the structure of your data to e.g. model-fitting and plotting tools

When you create a factor with `factor(some_vector)`, the levels (i.e. possible values) of the factor are inferred from the unique values of the vector, and the sort order of the levels of an factor is determined lexicographically in the current locale. You can explicitly provide a `levels=` argument to specify the allowed values and their sort order. If your factors have some sort of innate order, you can create an "ordered factor" with `ordered()`, but ordered vectors and unordered vectors have identical behavior (i.e. both are sortable); the difference is purely semantic. Even for nominally unordered factors, the "first" value of the factor can have semantic importance for e.g. linear regression with `lm`. If you find that you need to change the first value of an unordered factor, use the `relevel` function as `my.factor <- relevel(my.factor, 'the_new_first_level')`.

Comparisons for factors look the same as they do for character atomic vectors. For example, `measurements[measurements$measurement.type == 'weight',]` will give you all of the weight rows of the measurement table.

Factors have a numeric representation underneath that you can see by calling `as.integer` against a factor. The [R language documentation](http://cran.r-project.org/doc/manuals/r-release/R-lang.html#Factors) rolls its eyes, sighs, pushes up its glasses, and stops just short of telling you not to look. Caveat emptor.

All of the levels that a factor will ever have must be defined at the time the factor is created. If you want to add new levels or remove existing levels, you have to build a new factor. If you attempt to assign a value to an element of a factor that is not one of the pre-existing levels, you will get a warning for your troubles and the element will take the value `NA`.
