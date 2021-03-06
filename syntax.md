---
title: Basic syntax and gotchas
layout: page
---

Here, in no particular order, is a list of things that will help you get a sense of the shape of the language if you're already familiar with other curly-brace or interpreted languages.

* `;` or newline separates commands.

* Use hash-comments (`#` to end of line).

* Variable typing is weak and dynamic; variables are not declared before use. Like PHP and Javascript, variables have function (not block) scope.

* Whitespace is meaningless, unless it isn't. Some [parsing ambiguities](http://shape-of-code.coding-guidelines.com/2012/02/29/parsing-r-code-freedom-of-expression-is-not-always-a-good-idea/) are resolved by considering whitespace around operators. See and despair: `x<-y` (assignment) is parsed differently than `x < -y` (comparison)!

* Speaking of which, assignment looks stupid. I shit you not; these all have the equivalent effect of storing the value of `b` in `a`:

    *  `a <- b;` (the most common form in the wild)
    *  `b -> a;`
    * `assign("a", b);`
    * `a = b;`

    There are [subtle differences](http://stackoverflow.com/questions/1741820/assignment-operators-in-r-and) and [some authorities](https://google.github.io/styleguide/Rguide.xml) prefer `<-` for assignment but I'm increasingly convinced that they are wrong; `=` is always safer. R doesn't allow expressions of the form `if (a = testSomething())` but does allow `if (a <- testSomething())`. To assign to *globals*[^globals], use `<<-`.

* Dots in identifier names are just part of the identifier. They are not scope operators. They are not operators at all. They are just a legal character to use in the names of things. They are often used where a normal human being would use underscores, since underscores were assignment operators in S, which I promise you don't even want to think about.

* If you squint, `$` acts kind of like the `.` scope operator in C-like languages, at least for data frames and lists. If you'd write `struct.instance_variable` in C, you'd maybe write `frame$column.variable` in R.

* Sequence indexing is base-one. Accessing the zeroth element does not give an error but is never useful. More on this in the "Atomic vectors" section.

* Be careful with `for` loops. The syntax is vaguely Pythonic: `for(i in <sequence>) { do something; }`. You may be tempted to use the sequence operator, `:`, which is akin to `range()` in Python, to generate a list of integers to iterate over. Two cautions here. First, this is rarely the R idiom to use; as in MATLAB, vector operations are usually faster and harder to screw up than iteration. Reference the third chapter of the [R inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf) for advice on vectorizing. Second, if you do something like `i in 1:foo`, the *wrong thing* will happen if `foo` ever holds the value 0. `1:0` is the sequence (1, 0), since the `:` operator can and will count backwards. Always check whether `foo` is zero before you run your loop if you use `:`. If you're iterating over the indices of a sequence, always use `seq_along(x)` in preference to `1:length(x)`.

* Execute external code in the current workspace using `source('filename.R')`. If you have a unit of code that you want to spread over multiple files, the most proper way to do it is to build an [R package](http://r-pkgs.had.co.nz/), which is not as simple as you would like.

* Pull in a library with `library(foo)` or `library('foo')` or `require(foo)` or `require('foo')`. `library()` is actually more stringent, and dies on an error if the library can't be found. The return value of `require()` is `TRUE` if loading the library was successful but failure to load the library is a warning and isn't fatal.

* If something works on your machine but not your collaborator's, ask them if they have any `option()` calls in their `.Rprofile` file. This is a way to change the default behavior of some functions and it's an awful idea, because it will change the behavior of anybody else's code (including libraries) that depends on the default settings, in a way that's really hard to debug when you forget about it.

* Otherwise, fundamentals are just C-like enough to lull you into a false sense of security.

[^globals]: Technically, this finds the closest parent scope that contains a variable of that name, falling back to assignment in the global scope if the name isn't found in any of the parent environments. Consult `?"<<-"` for more. It turns out name resolution in R [is complicated](http://blog.obeautifulcode.com/R/How-R-Searches-And-Finds-Stuff/).

# Helping yourself

The R interpreter's built-in help feature is the only place I can consistently find documentation on anything. Try: `?function_name` or `??search_term`.

Because even R's name is stupid, it's really hard to google R things in a useful way. Sorry. Welcome to R!

[![This is fine.](https://i.imgur.com/c4jt321.png)](http://gunshowcomic.com/648)

The [Hyperpolyglot](http://hyperpolyglot.org/numerical-analysis) page comparing MATLAB, R, and NumPy syntax is helpful. John D. Cook's [R programming for those coming from other languages](http://www.johndcook.com/R_language_for_programmers.html) page is brief but useful. The [R inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf) offers complementary advice but blames the victim.

R messiah Hadley Wickham has a [very useful wiki-book](https://github.com/hadley/devtools/wiki) on advanced R development.  The [vocabulary](https://github.com/hadley/devtools/wiki/Vocabulary) appendix is an excellent list of things to learn.

[Stack Overflow](http://stackoverflow.com/questions/tagged/r), [#R on Freenode](https://webchat.freenode.net/?channels=R), [#rstats on Twitter](https://twitter.com/search?q=%23Rstats&src=typd), and the [R-Help](https://stat.ethz.ch/mailman/listinfo/r-help) mailing list are good places to ask for help. There is [offputtingly dense](http://www.r-project.org/posting-guide.html) advice on asking good questions.
