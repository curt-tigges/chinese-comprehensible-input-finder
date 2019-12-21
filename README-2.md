# Zhongwen i+1: Chinese Comprehensible Input Finder

This tool was created to assist students of Chinese in the task of finding native-language materials that are appropriate to their proficiency level. According to linguist Stephen Krashen's Input Hypothesis, language is most effectively acquired through the consumption of massive amounts of *comprehensible* input--that is to say, native language content--written, spoken, or otherwise--that contains a small percentage of *unknown* content, but enough known content that the student can figure out the unknown component through context and inference.

In the real world, this means that the best content for a user to consume would be books, movies, articles and so on that consist of about 90-95% known words, where the unknown words aren't terribly obscure--instead, they should be relatively common words that are of value to a language learner.

The functions in this notebook take a user-provided collection of Chinese text--with one document per line--and return a ranked list of the most comprehensible documents from the collection. It will also produce and write out a frequency list for the collection and a CSV containing metrics for the comprehensible documents it outputs.

## Requirements
This notebook requires the installation of the following packages:
* Jieba

## Basic Parameters & Variables
The script begins by initializing some global variables and lists to contain the data we produce. Here is also where parameters can be entered for comprehensibility ranges and input files. Most users will not need to edit the file anywhere else.

However, users will need to examine and edit the *input/punct.txt* and *input/known_vocab.txt* files. For *known_vocab*, simply make a list of all words known. Future versions of this tool may fill this out automatically via vocabulary quizzes.

## Punctuation Removal
The first two functions, build_punct_list and strip_chinese_punctuation are used for removing punctuation from the input. A user can specify which punctuation to remove from readlines via the input/punct.txt file. build_punct_list reads the punct.txt file and creates a list of punctuation to remove, and strip_chinese_punctuation is then called for each document to remove punctuation.

## Corpus Building
Next, we read through each line of input, strip the punctuation, and then segment the individual words in the file using the Jieba library. This is a toolkit specifically designed for parsing Chinese text. At the end, we will have a list of lists that contains the documents and the words within each. We also produce the wordcount, document count, and a list of the original (non-processed) documents with their punctuation.

Once this is complete, we can then initialize a matrix containing metrics for each document and build the combined vocabulary list.

In the process, we use:
* build_corpus, which reads lines of input, removes punctuation, segments words, and puts it all into a list of lists;
* initialize_doc_metric_matrix, which creates a list of metrics for each document
* build_vocabulary, which identifies unique words in the collection and creates a dictionary of them.

## Vocabularies
* collect_known_words collects a user-provided list of known words. This list should be placed in the input folder and should be named "known_vocab.txt." These words are then matched to the vocab_dict dictionary, where the value is marked "True" to signify the word is known.
* Additional functions get_doc_word_count and build_collection_freq_list also get the document word counts (and place these in the document metric matrix) and build a frequency list over the entire collection. The frequency list will be used to give documents within the defined comprehensibility range a score based on the "usefulness" of their unknown words.


## Document Rank Processing
Now we can begin the process of determining the rank for each document based on comprehensibility and the usefulness of their unknown words. This is done with:
* get_doc_known_count, which calculates the number of known words and the usefulness ranking for each doc,
* calculate_known_ratio, which divides known count by total document word count
* get_comprehensible_collection, which creates a smaller and more manageable list of documents that fall within the comprehensibility range


### Comprehensibility Ranking
First, the get_doc_known_count function calculates the number of words in each document that are known by the user. This count is divided by the total word count to get the ratio of known words to total words. The function calculate_known_ratio is then used to get the ratio.


### Usefulness Ranking
If the word is unknown, then get_doc_known_count uses a ranking formula (-1/log(word probability)) on each word,where *word probability* is defined as the count of that word in the collection divided by the total count of all words in the collection. For each document, this is summed over all words, and then normalized by the document word count and multiplied by 100 to get more easily-read values (and prevent underflow).

This formula is designed to ensure that documents with more common words that are unknown to the reader will be ranked higher, since they are probably the most useful ones to know (and will help the student to progress faster). The -1/log portion of the formula effectively transforms exponentially declining tiny fractions into a more gradually declining set of values (with the -1 serving to undo the negative valuation created by performing a log on fractional values).


### Fetching Comprehensible Documents
Once the above work has been performed, we then copy all documents within the comprehensibility range into a new, smaller array.

## Output

### Writing Frequency Lists
The first output we generate consists of the frequency lists. The function print_freq_lists_to_file creates two lists:
* freq_list_all.csv contains a list of all words and their frequencies. This can be divided by the total words to get the word probability.
* freq_list_repeated_words.csv contains a list of all words that occur more than once. This is useful for large corpii with exceptionally large numbers of words that only appear once, such as Chinese Wikipedia.

### Printing Comprehensible Documents
To output all comprehensible documents, the function print_comprehensible_collection_to_file simply looks at the ID of the document in the comprehensible document collection, and then print the corresponding raw, unprocessed document (with original punctuation, etc.) to a line in text document comprehensible_collection.txt. We also print the document metrics (usefulness and comprehensibility) to comprehensible_collection_metrics.csv.

## Execution
With all functions defined, all that remains is to execute. The attached notebook will carry out the entire process of taking in input, processing, and producing the output files.