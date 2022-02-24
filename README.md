# What is the Market Thinking About? 

## Background 
Every US public company, at the end of each quarter, holds an earnings call to share the results with the market and investors. These calls cover the results of the previous quarter, the logic and reasons, as well as plans and hopes for the future. This is a collosal volume of information, and together, these transcripts can provide insight into market concerns, as well as specifics for a particular company. For those interested in finance and the stock market, these are a fantastic resource. 


Earnings calls are also boring. Since this is a presentation to the company investors and wall street, it is a delicate dance of diplomacy - every company tries to portray their best side. This results in a difficult to read transcript peppered with financial terms. The volume of these also provides a challenge - consuming these in large numbers is a time consuming task. 

The topics discussed across these transcripts shed light on potential concerns and market factors, such as inflation. But for the amateur, such as myself, a simplification is needed. 


## Design 

I sought to build a tool which analyzes topics discussed across the industry, shedding light on global topics potentially concerning the markets, rather than details of the individual company quarter. Topics such as the pandemic, war, privacy law changes, and the sentiment across companies with regards to each. 

There are two pieces to this: locating topics which are mentioned in earnings calls, but do not refer to the workings of the company itself, or financial presentations - rather we're looking for outside factors metioned. This requires careful filtering to remove the obfuscating words, and topic modeling on the remaining terms. 

The second is sentiment. Using the original sentence, as well as sentiment models trained on financial discussions, I intend to identify sentiment per sentence. Combining sentiments per sentence and topics per sentence, I was looking to calculate sentiment per topic. 


## Data 
To create my corpus, I scraped the Montley Fool transcripts of 1000 earnings calls over the last 2 quarters(Dec 2021, Sep 2021), and used the prepared statements. Additionally, I researched using the Q&A section, which contains both analyst questions, and company responses, rather than the prepared section; however, I found this data significantly harder to parse, due to the unplanned form of the responses, which resulted in fewer complete sentences, and harder to understand(and parse) phrasings. 
Ex: "And so that -- to the question that was asked before around are we like -- that was asked before around, is there anything that's going to make it so that we -- it takes us longer to kind of get to where we want on this, is that even though we're compounding extremely quickly, that's -- you know, we also have a competitor that is compounding at a pretty quick rate, too. " - Meta, Q4 earnings Q&A
The final corpus consisted of 152,000 sentences in 1000 documents, and approximately 3 million words. 

## Algorithms 
There were two main components to the project, Sentiment analysis, and Topic Selection. 
Both were performed on the sentence level. 

### Sentiment Analysis 
* VADER
* Loughran McDonald
* FinBert 

I originally used Vader for sentiment analysis, however I found that due to the positive phrasings of the sentences, most of the sentiment returned as positive. After some research, I found the Loughran McDonald dictionary of financial terms, which specifically seeks to address this, and its implementation in the LM package. 
After manually inspecting the results, I found that while the results were improved, but resulted in many neutral statements due to positive and negative words being used in the same sentence. 
I located the FinBert package, which trains the Bert algorithm on Financial data, and provides better sentiment for this specific purpose. After manually inspecting the labels, I defaulted to using the Finbert results. 

### Topic Selection

#### General Cleaning 
I used Gensim and Spacy create a custom pipeline for cleaning the financial data. 
My interest was in persisting terms and groups of words, which also removing stop words. 

![Pipeline](https://github.com/mpolyakova/earnings-nlp-metis/blob/master/pipeline.png)

I start with a spacy pipeline in order to persist the parts of speech from the original sentence as I clean.  
I start with combining noun chunks (nouns and their descriptors), and Named entities into single token. This is to persist combinations such as "supply chain" and "inflationary pressures", as I filter for noun. 

Then I filter for nouns. I made the decision that topics which concern the market are likely to show up as nouns, rather than verbs or other parts of speech, and they will be easier to locate this way. 

Since noun chunks and Named entities frequently include words such as "the" as part of the token, which would otherwise be considered stop words, I break apart each token into parts, and remove the stopwords. 

At this point, the sentences are broken into a string of nouns (sometime with adjectives). Identify common phrases, I use gensim Phrases to find common bigrams and trigram, resulting in phrases such as "supply chain" and "supply chain shortages". 

I then lemmatize the word list to narrow the list of words. 

#### Further Filtering

At this point, the list contains a large number of business terms and terms relatively specific to wall street, such as "revenue", "cost", "business", "market", "growth", "opportunity". These, while important in the documents, and present in the majority of them, are not the topics I'm looking for - they are more specific to the format and audience. Since they dominate the documents, I attempt to remove this - first by hand and word counts, then by number of documents and avg times per document. This is a manual process during which I manually inspect the resulting word set. At the end, however, this removes over half of the sentences in the corpus, and results in 1-2 words on average per remaining sentence. 


#### Topic Selection 
I ran the algorith using both LSA and LDA for multiple number of topics, and both the count based vectorizer and the tfidf vectorizers. Consistenly, I saw more reasonable groups of words in the LDA, so I focused on optimising there.  Additionally, the frequency rather than raw count based input provided consistently higher coherence scores. 

I used various filtered corpuses(approx 10 different filterings), and ran LDA using various numbers of topics(range 5-40 topics), monitoring the outputted coherence scores, and topic print outs. I searched for higher coherence. For the more promising combinations, I ran the model and used PyLDavis to inspect the topics and word prevalences in each topic.  

I was not able to conclusively identify topics. With less filtered input, the topics became dominated by the financial words. With more filtered input, I could not interpret the relevant terms into a topic. 


## Tools:
### Preprocessing
Numpy, Pandas, Spacy, NLTK, Scikit-Learn, Gensim
### Topic Modeling
LDA, LDA, Gensim 
### Visualation
WordCount, Seaborn, Matplotlib 



## Communication 
I completed a 5 minute presentation if insights from the topic modeling process. Slides and visuals are included in this repository. Future work may include specific company selection by industry, limiting problem scope. Additionally I would incorporate filtering via financial terms dictionary, and adjusting the cleaning pipeline to limit the noun chunking. 






