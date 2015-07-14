---
title: Factors
layout: page
---
Factors are very useful and responsible for many unpleasant surprises. Many people find that this is the single most important thing to learn about factors (but read on!):

> In keeping with R's "maximal-mystery" design philosophy, whenever a data frame is instantiated with data from a character atomic vector or with character data read from a file with `read.table`, the resulting column will be a factor instead of a character atomic vector. Passing `stringsAsFactors = FALSE` to `data.frame()` or `read.table()` will prevent this behavior and leave your character vectors alone.

Factors are useful, though, to represent small sets of discrete possibilities. They have a family resemblance to `enum`s in C and, more loosely, `:symbols` in Ruby. A R factor is a sequence type much like a character atomic vector except that the values of the factor are constrained to a set of string values, called "levels". For example, if you have a table of measurements of some widgets and each row corresponds to a single measurement of a single widget, you could have a factor-typed column called measurement.type containing the values "length", "width", "height", "weight", and "hue", with the corresponding numeric measurements stored in a "value" column.

Factors are useful for a few reasons:

* they provide validation; R will throw warnings if you try to assign an undeclared level (e.g. because you miskeyed it)
* they have a more compact representation in memory than character vectors
* they provide semantic information about the structure of your data to e.g. model-fitting and plotting tools

Factors are unordered by default. The sort order of the levels of an unordered factor is determined lexiographically in the current locale. If your factors have some sort of innate order, you can create an "ordered factor" with `ordered()`. Even for nominally unordered factors, the "first" value of the factor can have semantic importance for e.g. linear regression with `lm`. If you find that you need to change the first value of an unordered factor, use the `relevel` function as `my.factor <- relevel(my.factor, 'weight')`.

Comparisons for factors look the same as they do for character atomic vectors. For example, `measurements[measurements$measurement.type == 'weight',]` will give you all of the weight rows of the measurement table.

Factors have a numeric representation underneath that you can see by calling `as.integer` against a factor. The [R language documentation](http://cran.r-project.org/doc/manuals/r-release/R-lang.html#Factors) rolls its eyes, sighs, pushes up its glasses, and stops just short of telling you not to look. Caveat emptor.

All of the levels that a factor will ever have must be defined at the time the factor is created. If you want to add new levels or remove existing levels, you have to build a new factor. If you attempt to assign a value to an element of a factor that is not one of the pre-existing levels, you will get a warning for your troubles and the element will take the value `NA`.
