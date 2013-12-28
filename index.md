# aRrgh: a newcomer's (angry) guide to R
[Tim Smith](http://tim-smith.us) \<arrgh@tim-smith.us\>, [@biotimylated](http://twitter.com/biotimylated)

with Kevin Ushey \<kevinushey@gmail.com\>, [@kevin_ushey](http://twitter.com/kevin_ushey)

# Introduction
R is a shockingly dreadful language for an exceptionally useful data analysis environment. The more you learn about the R language, the worse it will feel. The development environment suffers from literally decades of accretion of stupid hacks from a community containing, to a first-order approximation, zero software engineers. R makes me want to kick things almost every time I use it.

But there are a lot of great tools that are built in R. ggplot2 is first-in-class and Bioconductor packages are often essential. Sometimes there's aught to do but grin and bear (though never without a side of piss and moan).

The documentation is inanely bad. I can't explain it. aRrgh is my attempt to explain the language to myself. aRrgh exists as a living document and will continue to grow -- it is not complete, but it got to a point where it seemed like it was probably useful so I decided to toss it on the web. It should be correct and it's a bug if it isn't. Please email me or [file issues on Github](https://github.com/tdsmith/aRrgh/issues).

The goal of the document is to describe R's data types and structures while offering enough help with the syntax to get a programmer coming from another, saner language into a more comfortable place.

# Table of contents

 * [Basic syntax](syntax.html): crash course and gotchas; finding help
 * [Atomic vectors](atomic.html): R's simplest data types; logic values; vectorizing; arrays
 * [Factors](factors.html): A useful and misunderstood data type. Where they come from, how to handle them
 * [Data frames](data_frames.html): R's structure for tabular data. How to create them, access semantics
 
To come?:

 * Indexing
 * Lists
 * Namespaces

# Colophon
© Tim Smith 2012-3. This work is made available under a [Creative Commons Attribution-ShareAlike 3.0 Unported](http://creativecommons.org/licenses/by-sa/3.0/deed.en_US) License.

If you enjoyed this, you will probably enjoy [PHP: A Fractal of Bad Design](http://me.veekun.com/blog/2012/04/09/php-a-fractal-of-bad-design/), which is even more cathartic.

<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-6770445-5', 'tim-smith.us');
  ga('send', 'pageview');

</script>