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
released version (2.0) package. Amongst the changes in the new version
is the includsion of the *udpipe* backend, which allows for parsing
text without the need to install Python or Java.

## Quickstart

### Step 1: Install package

First, install the latest version of the **cleanNLP** package:

{% highlight r %}
install.packages("cleanNLP")
{% endhighlight %}
You'll need to have a version of cleanNLP >2.0 to follow along
the rest of this blog post.

### Step 2: Initialize the udpipe backend

We will be using the excellent [udpipe backend](https://cran.r-project.org/web/packages/udpipe/index.html)
for this quickstart because it requires no external dependencies, is
quite fast, and can produce the majority of available annotation
tasks. Let's load the package and initialize the backend:

{% highlight r %}
library(cleanNLP)
cnlp_init_udpipe()
{% endhighlight %}
The first time you run this, R will download a 16Mb file. It will
be stored automatically between R sessions.

### Step 3: Format the input data

There are many formats for inputing data to cleanNLP package. We
will use the simplest format that consists of simply taking a
character vector containing one element per document. Here I will
create a small input vector of three well-known quotes:


{% highlight r %}
input <- c("It is better to be looked over than overlooked.",
           "Real stupidity beats artificial intelligence every time.",
           "The secret of getting ahead is getting started.")
{% endhighlight %}

You can, of course, create your own input data in any number of
formats. We are now ready to annotate text with **cleanNLP**.

### Step 4: Annotate the text

Now we a ready to annotate the input text with the function
`cnlp_annotate_tif`:

{% highlight r %}
anno <- cnlp_annotate(input)
anno
{% endhighlight %}



{% highlight text %}
## 
## A CleanNLP Annotation:
##   num. documents: 3
{% endhighlight %}
The output is an annotation object; there are many things you can do with the
annotation object, but for most users a good starting place is to turn it into
a single data frame:

{% highlight r %}
output <- cnlp_get_tif(anno)
print.data.frame(head(output))
{% endhighlight %}



{% highlight text %}
##     id sid tid   word  lemma upos pos cid pid case definite degree gender
## 1 doc1   1   1     It     it PRON PRP   0   1  Nom     <NA>   <NA>   Neut
## 2 doc1   1   2     is     be  AUX VBZ   3   1 <NA>     <NA>   <NA>   <NA>
## 3 doc1   1   3 better better  ADJ JJR   6   1 <NA>     <NA>    Cmp   <NA>
## 4 doc1   1   4     to     to PART  TO  13   1 <NA>     <NA>   <NA>   <NA>
## 5 doc1   1   5     be     be  AUX  VB  16   1 <NA>     <NA>   <NA>   <NA>
## 6 doc1   1   6 looked   look VERB VBN  19   1 <NA>     <NA>   <NA>   <NA>
##   mood number person pron_type tense verb_form voice source relation
## 1 <NA>   Sing      3       Prs  <NA>      <NA>  <NA>      3     expl
## 2  Ind   Sing      3      <NA>  Pres       Fin  <NA>      3      cop
## 3 <NA>   <NA>   <NA>      <NA>  <NA>      <NA>  <NA>      0     root
## 4 <NA>   <NA>   <NA>      <NA>  <NA>      <NA>  <NA>      6     mark
## 5 <NA>   <NA>   <NA>      <NA>  <NA>       Inf  <NA>      6 aux:pass
## 6 <NA>   <NA>   <NA>      <NA>  Past      Part  Pass      3    csubj
##   word_source lemma_source spaces
## 1      better       better      1
## 2      better       better      1
## 3        ROOT         ROOT      1
## 4      looked         look      1
## 5      looked         look      1
## 6      better       better      1
{% endhighlight %}
Each row in the output corresponds to a single word in the original
documents. The `id` column tells us which document the word came
from and other columns give the annotated information about each word.

### Step 5: Share and enjoy

The output object above is a data frame and can be stored as a csv file,
manipulated, and plotted just as any other data frame can in R. Check out
the next section for more details about how the annotation process outlined
above can be used and customized for your own needs.

## Next steps

This guide is meant to show a simple way to get started with the **cleanNLP**
package. Here are some ideas for moving from this guide to making the best
of what the package has to offer:

- if you work with non-English language, look at the help pages
for `cnlp_init_udpipe` to see how to initialize other natural languages
- for named entities and word vectors, look at using the spacy backend
in place of udpipe: `cnlp_init_spacy`. It takes a bit more more to set
up, however, as it requires Python and several Python modules.
- look into the structure of the raw annotation object; it consists of a
list of normalized data frames that can be filtered and joined to produce
sophisticated analyses of data

Future blog posts will highlight particular applications of the package
to specific textual corpora.




