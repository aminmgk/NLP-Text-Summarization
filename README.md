```python
# Import necessary libraries
from nltk import FreqDist
from nltk.tokenize import word_tokenize, sent_tokenize
from nltk.stem.wordnet import WordNetLemmatizer
from nltk.corpus import stopwords
from bs4 import BeautifulSoup
from urllib.request import urlopen
from gensim.models import Phrases
from gensim.models.phrases import Phraser
from collections import Counter
import string

# Define functions for text summarization
def intersection(sent1, sent2):
    # As sentences are lists of tokens, there is no need to split them.
    intersection = [i for i in sent1 if i in sent2]
    return len(intersection) / ((len(sent1) + len(sent2)) / 2)

def split_sentences(sents):
    sentence_stream = [[i for i in word_tokenize(sent) if i not in stop] for sent in sents]
    bigram = Phrases(sentence_stream, min_count=2, threshold=2, delimiter=b'_')
    bigram_phraser = Phraser(bigram)
    bigram_tokens = bigram_phraser[sentence_stream]
    trigram = Phrases(bigram_tokens, min_count=2, threshold=2, delimiter=b'_')
    trigram_phraser = Phraser(trigram)
    trigram_tokens = trigram_phraser[bigram_tokens]
    all_words = [i for j in trigram_tokens for i in j]
    frequent_words = [i for i in Counter(all_words).most_common() if i[1] > 1]
    sentences = [i for i in trigram_tokens]
    return frequent_words, sentences

def score_sentences(words, sentences):
    # Return scores for sentences.
    scores = Counter()
    # Words - list of words and their scores, first element is the word, second - its score.
    for word in words:
        for i in range(0, len(sentences)):
            # If word is also in title, then add double score to the sentence.
            if word[0] in sentences[i] and word[0] in title:
                scores[i] += 2 * word[1]
            elif word[0] in sentences[i]:
                scores[i] += word[1]
    sentence_scores = sorted(scores.items(), key=scores.__getitem__, reverse=True)
    return sentence_scores

def get_summary(text, limit=3):
    sents = sent_tokenize(text)
    frequent_words, sentences = split_sentences(sents)
    sentence_scores = score_sentences(frequent_words, sentences)
    limited_sents = [sents[num] for num, count in sentence_scores[:limit]]
    best_sents = [i[0] for i in sorted([(i, text.find(i)) for i in limited_sents], key=lambda x: x[0])]
    return best_sents

def summarize(text, limit=3):
    summary = get_summary(text, limit)
    print(title)
    print()
    print(' '.join(summary))

# Example of using the functions
url = urlopen('http://news.sky.com/story/snap-election-to-be-held-in-march-after-northern-ireland-government-collapses-10731488')
soup = BeautifulSoup(url.read().decode('utf8'), "lxml")
text = '\n\n'.join(map(lambda p: p.text, soup.find_all('p')))
text = text[text.find('An early election'):]

title = soup.find('h1').text.strip()

# Execute the summarization
summarize(text, 5)
```
