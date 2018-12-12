# Domain Specific Word Embeddings For Query Expansion

Using the continuous space word embedding model word2vec we study the use of term relatedness in the context of query expansion. Moreover, with our models trained on domain specific corpora we examine the performance of these embeddings against models which have been trained over arbitrary corpora. Our query expansion methods are used for standard ad\-hoc information retrieval on standard TREC data (Disks 4,5 with query sets 301\-450, 601\-700). We demonstrate that when classifying queries under a specific domain and using the appropriately trained word2vec embeddings, our retrieval methods achieve results which are negligibly better than using embeddings trained over arbitrary corpora. However, we notice the less ambiguous queries, which can be easily categorized into one specific domain, allow our embeddings to show significant improvement. Our results hint that if trained over larger domain\-specific corpora, using these embeddings may yield more accurate results

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites
Clone this repo
` git clone http://thisrepo.git`

Clone the Word2Vec repo
` git clone https://github.com/dav/word2vec.git `

Download the latest version of trec_eval at
` https://trec.nist.gov/trec_eval/ `

Download a Wikipedia dump of your choosing. Please note that final results can vary greatly based on which Wiki dump you use (as this will be used as the training corpora). For my experiments, I used the ` enwiki-20140707-corpus.xml ` dump which is available in a zipped format from Linguatools [here](https://linguatools.org/tools/corpora/wikipedia-monolingual-corpora/). Be aware that the zipped file alone is ~4.25gb and ~20gb unzipped.

## Running the Tests

For my tests I broke it down into the following steps.

### Build domain specific corpora
Use the provided ` xml2text.pl ` script in order to run through the Wikipedia dump and extract only plain text. Given that we want to build corpora of a specific domain we want to set the ` -only-categories ` option when running. This option takes in a plain text file containing a list of topics related to whatever domain we want. Two topic lists have been provided in `/resources`. The following example command will build a corpora with only technolgy related articles.
` perl xml2txt.pl -only-categories tech_topics.txt enwiki-20140707-corpus.xml output_corpora.txt`
Once the corpora is built, I would recommend removing punctuation, lower-casing, and removing stopwords so that the vectors are trained on similar syntax as the indexed docs.

### Index the Documents
Using Lucene we index the documents provided by TREC (disks 4 and 5). 
Note: Legalities prevent me from distributing these files. They must be obtained in your own way.
Provided a path for where the index will go as well as the path of where the documents are located as done is this example command
` java org.apache.lucene.demo -index /Users/Fern/Documents/Index  -docs /Users/Fern/Documents/TREC_Documents`
At the time of indexing Porter Stemming is applied and stopwords are removed by referencing a provided list of stopwords. I used the SMART stopword list provided [here](http://www.lextek.com/manuals/onix/stopwords1.html) and also available in `/resources`. If a different stopword list is provided then you will need to edit the source code; line 101 in `IndexFiles.java`. All the documents are originally lower-cased and void of any punctuation.

### Run Word2Vec to Obtain Vectors
Using the corpora built earlier, we will run Word2Vec to build a set of vectors that we will ultimately use in our query searches. Word2Vec provides a number of options before running. My trials were run with a size of 200, and ignored words that showed up less than three times.
` ./word2vec -train output_corpora.txt -output output_vectors.vec -size 200 -min-count 3`

### Seperate Queries By Domain
All queries to be used originate from the TREC Robust Query Track. For our experiments we want to seperate the tracks and group them together by similar domain. Two query lists have been provided, one technology related and one medicine related. The idea is to see how results compare when using certain vectors with certain search topics. The original Robust Track data is provided free of charge from TREC [here](https://trec.nist.gov/data/t13_robust.html).

### Seperate the qrels 
As with the initial queries, we want to seperate the qrels files so that they match with the seperated query files we made in the step above. The qrels are essentially the 'answers'. They contain a ranking for each of the indexed documents that are considered relevant for each query. Provided are two qrel files that correspond with each of the seperated query files (i.e. there is qrels file which contains the answers for just the queries in the technology related query file; same for the medical query file).

### Perform Search
With the TREC documents indexed and word vectors built, you should now be ready to perform searches to get a results file. The program takes in a `.properties` file to obtain all needed input. A template has been provided in `/resources`.  Varying fields will be 'indexPath', 'queryPath', 'stopFilePath', 'resPath', and 'vectorPath.' Once the `.properties` has been configured to your liking you can run the following command
`java PreRetrievalQE myPropFile.properties`
A results file will be created in the location specified by 'resPath'.

### Use trec_eval On Results File
You may notice that the results file has a very unique (and ugly) format. It is formmated as so in order to be compatible with `./trec_eval`. Why are we using `./trec_eval`? Because it is the standard scoring system put in place by the National Institue of Standards and Technology for those who participate in TREC (Text REtrieval Conference). In depth information can be found [here](https://zipfslaw.org/2016/02/19/trec_eval-calculating-scores-for-evaluation-of-information-retrieval/) and [here](http://www.rafaelglater.com/en/post/learn-how-to-use-trec_eval-to-evaluate-your-information-retrieval-system). Running `./trec_eval -h` will also provide you with more in depth information.

