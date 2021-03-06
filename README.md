# Literatures

## papers
1. Show, Attend and Tell : https://arxiv.org/pdf/1502.03044v2.pdf#cite.Pascanu2014

2. Attention is all you need : https://arxiv.org/pdf/1502.03044v2.pdf#cite.Pascanu2014
Notes : 
- Overall architecture follows encoder-decoder style, encoder has 6 layers, each with 2 sublayers (attention layer and a feed-forward layer), decoder also has 6 layers, each with 3 sublayers (two attention layers and a feed-forward layer, the first of which is masked to ensure the ith sequence is predicted based on known outputs before the ith position.
- attention layer consists of : query, key-value pair, which are mapped to an output. Weight for each value is calculated through a compatibility function between the query and key, scaled by 1/sqrt(dk) where dk is the dimension of query or key, author suspects that without scaling, large dk leads to a softmax output nearing the diminishing gradient.
-

3. A multi-task U-Net for segmentation with lazy labels
- Separate into three tasks : detection, separation and segmentation
- Detection : using the weak annotations, produce classification of rough regions through the under-segmentation mask.
- Separation : using rough scribbles of fine separations, separate instances connected without a clear boundary.
- Segmentation : using a few strong annotations, produce pixel-wise classification.
- All three tasks share the same contracting path, but this paper introduces a new multi-task block for the expansive path.
- Use a weighted loss function over the samples to account for missing annotations as the weak/strong annotations are shared between the three tasks
- For each multi-task block : Detection and Segmentation have a common path similar to decoder part of standard U-net, they share the same weights, use the same concatenation with feature maps from contracting path via the skip connections. However, inserted an additional residual sub-block for Segmentation to allow for extra network parameters to learn about object boundary localization. For Separation, path built on the top of Detection/Segmentation, with long skip connections starting from the sub-blocks of the Detection/Segmentation path, these extract higher resolution features from Segmentation task and use them in the Separation. Apply softmax after the last multi-task block.
- 6 spatial resolutions, input images of 256x256. For contracting path : down-sampling via max-pooling, each contracting block has a conv layer, a batch normalization, followed by leaky ReLU 0.01. For expansive path : up-sampled by bilinear interpolation
- Lazy-labelling : first step consists of rough scribbles on the detection regions, second step is on labelling the interface of close objects, for the third step, Grabcut (graph-cut based method) is used to generate accurate (strong) labels, where corrections will be needed.
- dataset : 20 images, only 2 with strong annotations, 15 weak annotations for task 1, and 20 interfaces annotations for task 2. Validation set with 6 images containing all annotations.
- data augmentation : random rotation, random crop, random flipping
- training : batch size 16, Adam optimizer

## useful learning links
understanding LSTM : https://colah.github.io/posts/2015-08-Understanding-LSTMs/

## Data Preprocessing for NLP

### Tokenization
- segmentation / sentence boundary determination 

### Normalization

#### Lowercasing 
- useful for small, sparse dataset
- depends on the task (approach + domain)
- for our task of topic classication, overall theme can be captured by the lowercase data, though if large dataset is available, not lowercasing can improve accuracy

#### Stemming
- useful in general
- Porters algorithm

#### Lemmatization
- similar but more liguistically robust than stemming, e.g. 'better' -> 'good'
- WordNet
- however, this paper titled 'On the Role of Text Preprocessing in Neural Network Architectures: An Evaluation Study on Text Categorization and Sentiment Analysis' (https://arxiv.org/pdf/1707.01780.pdf) finds that lemmatization has no significant impact on accuracy for text classication with neural architectures.

#### Stopword Removal
- a simple custom-built stopword library or a word-count-percentage-threshold removal
- more research to determine its effectiveness in topic classification
- USPTO (United States Patent and Trademark Office), which contains about 100 functional words in English

#### Other normalizations
- useful for social media content swamped by noise such as 'tomorrow' being represented by '2moro'/'2morrow'/'tmr'/etc.
- news content are usually clean, so not a necessary step (looking at Scope as a whole, the specific step to pull relevant Tweets for a particular news headline may require normalization, but for news topic classification alone it is not be necessary)

### Noise Removal
- domain specific, comprises of punctuation/special character/number/html formatting/domain specific keyword removal

### Text Enrichment / Augmentation
- part-of-speech tagging / grammatical tagging

### Rule of Thumb
- for a general/well written texts (which is true for news data), coupled with lots of data (also true for news data), we need light pre-processing, while text augmentation could be useful but not critical

### - - - - - - - - - - - - - - - - - - - - - - - - -
## Upstream dependency
- data preprocessing very much depends on the data mining step. Ideally we want plain text file (.txt) with no HTML code or formatting contaminating the texts. If contamination present, additional pre-processing which involves cleaning HTML/formatting codes will be needed, can be done via regex (is there a more efficient way ... ?)
## Downstream dependency
- depending on the model we work with, chances are the papers will have already suggested the necessary pre-processing steps optimal for that particular model
## Others
- provided that we work with plain text, the next thing to look out for would be the domain, in our case, we're working with different data such as news content (so expecting for clean, well formatted texts, greatly reduces the amount of pre-processing required), tweets (dirtier data), video (may be dirty), podcast (may be dirty).

## Tutorial

## Meeting 1 (21st of November 2019)
### Type of data
- videos (description, title, transcript)
- news (title, body)
- tweets (author, body, hashtags)
- (no source yet) podcast (transcript)

## Feedback
- from Data Mining Team 
- from Network Design Team
- using these feedback, finetune our preprocessing requirements

### What we can do now
- look for good/popular packages for the type of pre-processing we're sure to need (provide links, package name, tutorials, sample script to demonstrate, etc.)
### stemming and lemmatization (Areeg)
- https://www.datacamp.com/community/tutorials/stemming-lemmatization-python
- Stemming is the process of reducing inflection in words to their root forms such as mapping a group of words to the same stem even if the stem itself is not a valid word in the Language
- Pyhton package: NLTK (Natural Language Tool Kit)
- pip install nltk
- nltk.download() - choose the punkt model
- English stemmers are PorterStemmer and LancasterStemmer (newer)
- PorterStemmer doesn't usually produce stems that are actual English words. Follows a set of rules to remove suffixes rather than actually having a list of possible root words. BUT this is good because it is simple and fast. Used in Information Retrieval Environments.
-LancasterStemmer is iterative and may be heavy-stemming and hence may reduce the meaning behind the words.
### normalization (for tweets) (Areeg)
- https://medium.com/lingvo-masino/do-you-know-about-text-normalization-a19fe3090694

### stop-word removal (CJ)
- common tool : NL toolkit has a defined stopword library. May need to add/remove stopwords depending on our datasets (e.g. for news should be okay to just use standard library),  for Tweets or dirtier texts data, may require some newly defined stopwords
- https://github.com/gaganmanku96/nlppreprocess/blob/master/nlppreprocess/nlppreprocess.py
- allows customized stopwords addition/removal + words replace (e.g. doesn't/doesnt/dosnt -> does not, etc.)

### text-enrichment (CJ)
- multiple augmentation techniques
- word-level augmentation such as insertion and substitution augmentation using contextualized word embeddings, i.e. BERT augmentation
- WordNet from NL Toolkit helps to find synonym words, allows for substitution augmentation
- again, different strategy for different datasets : (news data) contextualized word embeddings augmentation + synonym replacement would be good, (tweets/social media) 
- relevant paper (studying the effect of different augmentation strategies on social media + formal text such as news data, exactly our case) introduces augmentation on text source as well as text representation

### lowercasing (because capital may indicate special nouns or other useful information)
-

## Data Mining

### Sources
- newsdata, tweets

### Requirements
- preferrably labelled
- sources (for checking against bias)
- newsdata (BBC has labelled data)
- labels (depends on what the source contains, but preferrably we want to create our own hierarchical label, scraping news from hierarchical source)

### Datasets
#### BBC Dataset
- 2225 documents from BBC news website with 5 class labels (business, entertainment, politics, sport, technology)
- another 737 documents from BBCSport news website 5 class labels (athletics, cricket, football, rugby, tennis)
- pre-processed datasets are available (stemming, stop-word removal, low term frequency filtering (count<3))
- DATASET AVAILABL AT: http://mlg.ucd.ie/datasets/bbc.html
- Github implementation of Dataset: https://github.com/suraj-deshmukh/BBC-Dataset-News-Classification

#### News Category Dataset
- 202,372 news headline from year 2012 to 2018 from HuffPost with 40+ class labels
- format is : category, headline, authors, link, short description, date
- as the short description is very short, we can scrape the original news article from the provided link and then pre-process 
https://www.kaggle.com/rmisra/news-category-dataset

#### 20 Newsgroups Dataset
- around 18,000 newsgroups post on 20 topics
- available on scikit-learn and can be fetched directly with flexible custimisation such as loading only a sub-selection of categories, filtering newsgroup-related metadata (e.g. headers, footers, quotes) for a more realistic representation of the news dataset, etc.
https://scikit-learn.org/0.19/datasets/twenty_newsgroups.html
- UPDATE: Its news labels are weird; waiting on a decision on how to deal with them. (Areeg)

#### 4 Processed Datasets (Reuters-21578 R8, Reuters-21578 R52, 20 Newsgroup, Cade12 and WebKB
- Preprocessed with short word (count<3) removal, stop-word removal and stemming
http://ana.cachopo.org/datasets-for-single-label-text-categorization

#### UCI Machine Learning Repository (NOT SURE ABOUT THIS)
- 422937 news pages divided into business, science and tech, health and entertainment
- Used news aggregator in 2014
- Provides links to news sources but no the actual data so may need scraping as above
- https://archive.ics.uci.edu/ml/datasets/News+Aggregator

#### Twitter Streaming API for Tweets
- Tutorial to obtain these
- http://adilmoujahid.com/posts/2014/07/twitter-analytics/
- https://towardsdatascience.com/tweet-analytics-using-nlp-f83b9f7f7349

#### Dataset :
summary of dataset : 
1. 20 Newsgroup in 4 preprocessed forms
- lowercased, tabs/punctuation replaced by space
- the above + remove short words < length 3
- the above + remove stop-word
- the above + stemmed

2. BBC 
- full text
- remove stop-word, remove low count term <3, stemmed

#### Dataset Category Filtering : 
R8
acq/crude/grain/interest/money-fx/trade -> economic/business
ship -> politics
earn -> removed

R52
acq/alum/bop/cocoa/coffee/copper/cotton/cpi/cpu/crude/dlr/fuel/gas/gnp/gold/grain/heat/housing/income/instal-debt/interest/ipi/iron-steel/jet/jobs/lead/lumber/money-fx/money-supply/nat-gas/nickel/orange/pet-chem/platinum/potato/reserves/retail/rubber/trade/wpi/ -> economic/business
ship -> politics
carcass/earn/lei/livestock/meal-feed/strategic-metal/veg-oil/zinc/sugar/tea/tin -> remove

20NG
alt.atheism/soc.religion.christian/talk.religion.misc -> religion
comp.graphics/comp.os.ms-windows.misc/comp.sys.ibm.pc.hardware/comp.sys.mac.hardware/comp.windows.x/sci.crypt/sci.electronics/sci.med/sci.space/ -> technology
talk.politics.guns/talk.politics.mideast/talk.politics.misc/ -> politics
rec.sport.baseball/rec.sport.hockey -> sports
misc.forsale/rec.autos/rec.motorcycles -> remove

Kaggle News-Category-Dataset
'ARTS','ARTS & CULTURE' -> arts
'COMEDY','ENTERTAINMENT','MEDIA' -> entertainment
'BLACK VOICES' -> black voices
'COLLEGE','EDUCATION' -> education
'BUSINESS', -> business
'CRIME' -> crime
'GOOD NEWS' -> good news
'GREEN' -> green
'HEALTHY LIVING -> health
'IMPACT' -> impact
'LATINO VOICES -> latino voices
'PARENTS' -> parents
'POLITICS' -> politics
'QUEER VOICES' -> LGBTQ
'RELIGION' -> religion
'SCIENCE','TECH' -> tech
'SPORTS' -> sport
'TASTE' -> food
'THE WORLDPOST','WORLDPOST','WORLD NEWS' -> world
'TRAVEL' -> travel
'WOMEN' -> women
'STYLE', 'WEIRD NEWS','FIFTY' -> remove (too few descriptions/too random)
preprocess :  remove date, link, authors, category -> type
              short_description > 10 words,
              headline + short_description -> news



