Text mining JSTOR: Quantitative approaches to histories of science
---
Ben Marwick & Ian Kretzler, UW Anthropology

This markdown file accompanies the poster presented at the UW Data Science event on 7 Feb 2014. It contains the R code used to generate the figures on the poster and additional details about methods that are not on the poster. 

[Google’s Ngram viewer](https://books.google.com/ngrams) is a tool for visualizing the popularity of words over time in 5 million books digitized by Google. It has been described as the ‘gateway drug’  for data science in the humanities ([1](http://mainedh.org/2013/09/09/google-ngrams-the-gateway-drug-to-digital-humanities/), [2](http://www.dancohen.org/2010/12/19/initial-thoughts-on-the-google-books-ngram-viewer-and-datasets/)). While it is fun for casual searches, it has substantial limitations that prevent it from being a scholarly tool. The main problem is we simply don’t know what is in the corpus. This makes validation through close reading of the texts impossible.

Inspired by the Google Ngram viewer, we wrote [JSTORr](https://github.com/UW-ARCHY-textual-macroanalysis-lab/JSTORr), an R package that visualizes Ngrams for corpora held by JSTOR, a digital library of 2000 scholarly journals. [JSTOR’s Data for Research](http://about.jstor.org/service/data-for-research) service allows users to download full text of journals. With JSTORr, the full text can be analysed for ngrams, word correlations, document clustering and topic modelling. This means that we can now carefully build a sizable corpora of scholarly literature on specific topics and investigate them with text mining methods. In this poster we demonstrate some of the basic functions of the package to get insights into the history of science. 

The package is hosted on github and can be installed locally using the following code:

```{r load-libraries}
library(devtools)
install_github(repo = "JSTORr", username = "UW-ARCHY-textual-macroanalysis-lab")
library(JSTORr)
```

For this poster we selected three journals by searching the Thomson Reuters ISI Journal Citation Reports for 'Statistics and Probability', 'Biology' and 'Anthropology' (more specifically archaeology) and ranking the search results by the journals' Eigenfactor Scores. We then obtained from JSTOR all the available articles of the highest ranking journal for each of the three subject areas. For 'Statistics and Probability' this was _Biometrika_, for 'Biology' this was _Philosophical Transactions of the Royal Society B_, and for 'Anthropology' it was _American Antiquity_. The journal articles were obtained from JSTOR's Data for Research service. The data were then analysed using the JSTORr package (1.0.20140204) with R (3.0.2 on x86_64-pc-linux-gnu 64-bit). 

```{r unzip-data}
# My local path where the zip files are - yours will be different
path <- '/home/two/teamviewer'
setwd(path)
## unzip journal archives obtained from dfr.jstor.org
# American Antiquity
unzip('2013.1.1.E84E4jrp.AA.zip', exdir = "AA") 
# Philosophical Transactions of the Royal Society B
unzip('2014.1.20.CjjqAWEE-PTRS_B.zip', exdir = "PTRS_B")
# Biometrika
unzip('2014.1.20.YFVATDRz-Biometrika.zip', exdir = "Biometrika")
```

Looking first at the statistics journal _Biometrika_, we can quickly see intellectual shifts in a high-impact statistics journal (above) from correlation to model-based analyses. We can also clearly see the impact of the two world wars on the journal with a drop in word frequencies during 1916-1920 and 1941-45. 

```{r Biometrika}
### unpack and get nouns Biometrika
path_Biometrika <-'/home/two/teamviewer/BS'
unpack1grams <- JSTOR_unpack1grams(path = path_Biometrika)
nouns <-  JSTOR_dtmofnouns(unpack1grams, sparse = 0.75)

# most frequent words over time
JSTOR_freqwords(unpack1grams, nouns)
```

Similarly, the prominent biology journal _Philosophical Transactions of the Royal Society B_ reveals a turn from macrostructures to a focus on cells and proteins. We might hypothesize that these intellectual trends are related to technological changes in imaging and biochemistry. This hypothesis could be tested by close reading of a small sample of articles.

```{r PTRS_B}
### unpack and get nouns PTRS_B
path_PTRS_B <-'/home/two/teamviewer/PTRS_B'
unpack1grams <- JSTOR_unpack1grams(path = path_PTRS_B)
nouns <-  JSTOR_dtmofnouns(unpack1grams, sparse = 0.75)

# most frequent words over time
JSTOR_freqwords(unpack1grams, nouns)
```

In archaeology we see a turn from typological analyses of pottery to work on population-level behaviors and settlement patterns. Dating becomes frequently discussed after radiocarbon methods become widely available. 

```{r AA}
### unpack and get nouns American Antiquity
path_AmAnt <-'/home/two/teamviewer/AA'
unpack1grams <- JSTOR_unpack1grams(path = path_AmAnt)
nouns <-  JSTOR_dtmofnouns(unpack1grams, sparse = 0.99)

# most frequent words over time
JSTOR_freqwords(unpack1grams, nouns)
```

Topic models generated by latent Dirichlet allocation show hot and cold topics over time in American archaeology. These validate and add context to the plot of most frequent words per decade generated in the previous chunk. We get a clear picture of how the intellectual history of American archaeology has changed over time, with the hot topics more abstract and theory-focused, and the cold topics more concrete and materials-based. 

```{r AA-topic-model}
# topic model for American Antiquity
K = 200 # set the number of topics to identify
topmod <- JSTOR_lda(unpack1grams, nouns, K, alpha = 50/K) # generate topic model

# plot hot and cold topics
htoncold <- JSTOR_lda_hotncoldtopics(topmod, pval = 0.01, ma = 5)
```

Ngrams over time show the complex relationship between gender and feminism in American archaeology. The two terms appear together initially, then diverge in their frequencies. 

```{r AA-ngram-over-time}
# plot two words over time
JSTOR_2words(unpack1grams, "gender", c("feminist", 'feminism'), span = 0.4)
```

K-means clustering of journal articles using word frequencies (right) reveals distinct approaches to gender in the American archaeological literature. In the upper right we have a cluster of articles on the archaeological record of the US Southwest. The large lower cluster is articles about gender theory and philosophy. 

```{r AA-k-means}
# plot k-means clusters of documents containing the word 'gender'
gender <- JSTOR_clusterbywords(unpack1grams$wordcounts, 'gender', f = 0.01)
```





