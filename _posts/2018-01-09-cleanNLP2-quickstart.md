---
layout: post
title: "cleanNLP 2.0: Quickstart Guide"
categories: rlang
---



**This post is a draft and will be officially released when
the updated package is available on CRAN.**

The **cleanNLP** package takes raw text and outputs a structured
representation of the text annotated with auto-generated annotations.
These annotations capture elements of the text such as detecting word
boundaries, giving base form of words into a base form (i.e., 'dog' is
the base of the word 'dogs'), and identifying parts of speech. This
document gives a quick guide to getting started with the recently
released version (2.0) package.

First, install the latest version of the **cleanNLP** package:

{% highlight r %}
devtools::install_github("statsmaths/cleanNLP")
{% endhighlight %}
We will be using the excellent [udpipe backend](https://cran.r-project.org/web/packages/udpipe/index.html)
for this tutorial because it requires no external dependencies, is
quite fast, and can produce the majority of available annotation
tasks. So, let's load the package and initialize the backend:

{% highlight r %}
library(cleanNLP)
cnlp_init_udpipe()
{% endhighlight %}
The first time you run this, R will download a 16Mb file. It will
be stored automatically between R sessions.

We are now ready to annotate text with **cleanNLP**. Here, I will
use a data set the comes from cleanNLP package. The data is stored
in a data frame with the first column giving the id of the document
and the second column giving the text of each document (documents
are individual articles). I will show a truncated version of the
text to make it clear what the data frame looks like:

{% highlight r %}
data(un)
dplyr::mutate(un, text = substr(text, 1, 45))
{% endhighlight %}



{% highlight text %}
## # A tibble: 30 x 2
##       doc_id                                          text
##       <fctr>                                         <chr>
##  1 article01 All human beings are born free and equal in d
##  2 article02 Everyone is entitled to all the rights and fr
##  3 article03 Everyone has the right to life, liberty and s
##  4 article04 No one shall be held in slavery or servitude;
##  5 article05 No one shall be subjected to torture or to cr
##  6 article06 Everyone has the right to recognition everywh
##  7 article07 All are equal before the law and are entitled
##  8 article08 Everyone has the right to an effective remedy
##  9 article09 No one shall be subjected to arbitrary arrest
## 10 article10 Everyone is entitled in full equality to a fa
## # ... with 20 more rows
{% endhighlight %}
Then, all one needs to do is run `cnlp_annotate_tif` on the `un` object.

{% highlight r %}
anno <- cnlp_annotate_tif(un)
anno
{% endhighlight %}



{% highlight text %}
##
## A CleanNLP Annotation:
##   num. documents: 30
{% endhighlight %}
There are many things you can do with the annotation object, but for
most users a good starting place is to turn it into a single data
frame:

{% highlight r %}
output <- cnlp_get_tif(anno)
print.data.frame(head(output))
{% endhighlight %}



{% highlight text %}
##      doc_id sid tid   word lemma upos pos cid pid case definite degree
## 1 article01   1   1    All   all  DET  DT   0   1 <NA>     <NA>   <NA>
## 2 article01   1   2  human human  ADJ  JJ   4   1 <NA>     <NA>    Pos
## 3 article01   1   3 beings being NOUN NNS  10   1 <NA>     <NA>   <NA>
## 4 article01   1   4    are    be  AUX VBP  17   1 <NA>     <NA>   <NA>
## 5 article01   1   5   born  bear VERB VBN  21   1 <NA>     <NA>   <NA>
## 6 article01   1   6   free  free  ADJ  JJ  26   1 <NA>     <NA>    Pos
##   gender mood num_type number person poss pron_type reflex tense verb_form
## 1   <NA> <NA>     <NA>   <NA>   <NA> <NA>      <NA>   <NA>  <NA>      <NA>
## 2   <NA> <NA>     <NA>   <NA>   <NA> <NA>      <NA>   <NA>  <NA>      <NA>
## 3   <NA> <NA>     <NA>   Plur   <NA> <NA>      <NA>   <NA>  <NA>      <NA>
## 4   <NA>  Ind     <NA>   <NA>   <NA> <NA>      <NA>   <NA>  Pres       Fin
## 5   <NA> <NA>     <NA>   <NA>   <NA> <NA>      <NA>   <NA>  Past      Part
## 6   <NA> <NA>     <NA>   <NA>   <NA> <NA>      <NA>   <NA>  <NA>      <NA>
##   voice source   relation word_source lemma_source spaces
## 1  <NA>      3        det      beings        being      1
## 2  <NA>      3       amod      beings        being      1
## 3  <NA>      5 nsubj:pass        born         bear      1
## 4  <NA>      5   aux:pass        born         bear      1
## 5  Pass      0       root        ROOT         ROOT      1
## 6  <NA>      5       conj        born         bear      1
{% endhighlight %}
Each row in the output corresponds to a single word in the original
documents. The `doc_id` column tells us which document the word came
from and other columns give the annotated information about each word.

In order to use on your own text, you just need to construct a new data
frame to input. For example, we could take three famous quotes:

{% highlight r %}
text <- c("It is better to be looked over than overlooked.",
         "Real stupidity beats artificial intelligence every time.",
         "The secret of getting ahead is getting started.")
input <- data.frame(doc_id = c("West", "Pratchett", "Twain"),
                        text = text,
                        stringsAsFactors = FALSE)
{% endhighlight %}
And then, putting all of the commands together, parse these as:

{% highlight r %}
library(cleanNLP)
cnlp_init_udpipe()
output <- cnlp_get_tif(cnlp_annotate_tif(input))
{% endhighlight %}
The output has the output format as above:

{% highlight r %}
print.data.frame(head(output))
{% endhighlight %}



{% highlight text %}
##   doc_id sid tid   word  lemma upos pos cid pid case definite degree
## 1   West   1   1     It     it PRON PRP   0   1  Nom     <NA>   <NA>
## 2   West   1   2     is     be  AUX VBZ   3   1 <NA>     <NA>   <NA>
## 3   West   1   3 better better  ADJ JJR   6   1 <NA>     <NA>    Cmp
## 4   West   1   4     to     to PART  TO  13   1 <NA>     <NA>   <NA>
## 5   West   1   5     be     be  AUX  VB  16   1 <NA>     <NA>   <NA>
## 6   West   1   6 looked   look VERB VBN  19   1 <NA>     <NA>   <NA>
##   gender mood number person pron_type tense verb_form voice source
## 1   Neut <NA>   Sing      3       Prs  <NA>      <NA>  <NA>      3
## 2   <NA>  Ind   Sing      3      <NA>  Pres       Fin  <NA>      3
## 3   <NA> <NA>   <NA>   <NA>      <NA>  <NA>      <NA>  <NA>      0
## 4   <NA> <NA>   <NA>   <NA>      <NA>  <NA>      <NA>  <NA>      6
## 5   <NA> <NA>   <NA>   <NA>      <NA>  <NA>       Inf  <NA>      6
## 6   <NA> <NA>   <NA>   <NA>      <NA>  Past      Part  Pass      3
##   relation word_source lemma_source spaces
## 1     expl      better       better      1
## 2      cop      better       better      1
## 3     root        ROOT         ROOT      1
## 4     mark      looked         look      1
## 5 aux:pass      looked         look      1
## 6    csubj      better       better      1
{% endhighlight %}
Check out the next section for more details about how this process can
be used and customized for your needs.

## Next steps
