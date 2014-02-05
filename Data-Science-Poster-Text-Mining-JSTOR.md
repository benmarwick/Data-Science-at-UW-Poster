Text mining JSTOR: Quantitative approaches to histories of science
---
Ben Marwick & Ian Kretzler, UW Anthropology

This markdown file accompanies the poster presented at the UW Data Science event on 7 Feb 2014. It contains the R code used to generate the figures on the poster and additional details about methods that are not on the poster. 

Google’s Ngram viewer is a tool for visualizing the popularity of words over time in 5 million books digitized by Google. It has been described as the ‘gateway drug’  for data science in the humanities. While it is fun for casual searches, it has substantial limitations that prevent it from being a scholarly tool. The main problem is we simply don’t know what is in the corpus. This makes validation through close reading of the texts impossible.

Inspired by the Google Ngram viewer, we wrote JSTORr, an R package that visualizes Ngrams for corpora held by JSTOR, a digital library of 2000 scholarly journals. JSTOR’s Data for Research service allows users to download full text of journals. With JSTORr, the full text can be analysed for ngrams, word correlations, document clustering and topic modelling. This means that we can now carefully build a sizable corpora of scholarly literature on specific topics and investigate them with text mining methods. In this poster we demonstrate some of the basic functions of the package to get insights into the history of science. 

The package is hosted on github and can be installed locally using the following code:

```{r load-libraries}
library(devtools)
install_github(repo = "JSTORr", username = "UW-ARCHY-textual-macroanalysis-lab")
library(JSTORr)
```

For this poster we selected three journals by searching the Thomson Reuters ISI Journal Citation Reports for 'Statistics and Probability', 'Biology' and 'Anthropology' (more specifically archaeology) and ranking the search results by the journals' Eigenfactor Scores. We then obtained from JSTOR all the available articles of the highest ranking journal for each subject in JSTOR's collection. For 'Statistics and Probability' this was _Biometrika_, for 'Biology' this was _Philosophical Transactions of the Royal Society B_, and for 'Anthropology' it was _American Antiquity_. The journal articles were obtained from JSTOR's Data for Research service. The data were then analysed using the JSTORr package (1.0.20140204) with R (3.0.2 on x86_64-pc-linux-gnu 64-bit). 

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
 
 
Similarly, a prominent biology journal reveals a turn from macrostructures to a focus on cells and proteins. We might hypothesize that these intellectual trends are related to technological changes. This hypothesis could be tested by close reading of a small sample of articles.

```{r Biometrika}
### unpack and get nouns Biometrika
path_Biometrika <-'/home/two/teamviewer/BS'
unpack1grams <- JSTOR_unpack1grams(path = path_Biometrika)
nouns <-  JSTOR_dtmofnouns(unpack1grams, sparse = 0.75)

# most frequent words over time
JSTOR_freqwords(unpack1grams, nouns)
```



```{r PTRS_B}
### unpack and get nouns PTRS_B
path_PTRS_B <-'/home/two/teamviewer/PTRS_B'
unpack1grams <- JSTOR_unpack1grams(path = path_PTRS_B)
nouns <-  JSTOR_dtmofnouns(unpack1grams, sparse = 0.75)

# most frequent words over time
JSTOR_freqwords(unpack1grams, nouns)
```


```{r AA}
### unpack and get nouns American Antiquity
path_AmAnt <-'/home/two/teamviewer/AA'
unpack1grams <- JSTOR_unpack1grams(path = path_AmAnt)
nouns <-  JSTOR_dtmofnouns(unpack1grams, sparse = 0.75)

# most frequent words over time
JSTOR_freqwords(unpack1grams, nouns)
```


```{r AA-topic-model}
# topic model for American Antiquity
K = 200
topmod <- JSTOR_lda(unpack1grams, nouns, K, alpha = 50/K)
# plot hot and cold topics
htoncold <- JSTOR_lda_hotncoldtopics(topmod, pval = 0.01, ma = 5)
```

```{r AA-ngram-over-time}




