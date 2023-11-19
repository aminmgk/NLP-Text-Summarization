### 1. Import Necessary Libraries:

```python
from nltk.tokenize import sent_tokenize, word_tokenize
from nltk.corpus import stopwords
from gensim.models.phrases import Phrases, Phraser
from collections import Counter
```

Explanation:
- `nltk`: Natural Language Toolkit, used for natural language processing tasks.
- `gensim`: Library for topic modeling and document similarity analysis.
- `Counter`: Helps count occurrences of elements in a list.

### 2. Define Functions:

#### `split_sentences(sents)`

```python
def split_sentences(sents):
    stop = set(stopwords.words('english'))
    sentence_stream = [[word.lower() for word in word_tokenize(sent) if word.isalnum() and word.lower() not in stop] for sent in sents]
    bigram = Phrases(sentence_stream, min_count=2, threshold=2, delimiter=b'_')
    bigram_phraser = Phraser(bigram)
    bigram_tokens = bigram_phraser[sentence_stream]
    
    all_words = [word for tokens in bigram_tokens for word in tokens]
    frequent_words = [word for word, count in Counter(all_words).items() if count > 1]
    sentences = [' '.join(tokens) for tokens in bigram_tokens]

    return frequent_words, sentences
```

Explanation:
- Tokenizes sentences and words using NLTK.
- Removes stopwords and non-alphanumeric characters.
- Applies Phrases model from Gensim to find bigrams.
- Extracts frequent words and constructs sentences.

#### `get_summary(text, limit=3)`

```python
def get_summary(text, limit=3):
    sents = sent_tokenize(text)
    frequent_words, sentences = split_sentences(sents)
    summary = [sent for sent in sentences if any(word in sent for word in frequent_words)][:limit]
    return summary
```

Explanation:
- Tokenizes the input text into sentences.
- Calls `split_sentences` to get frequent words and sentences.
- Creates a summary by selecting sentences containing frequent words.

#### `summarize(text, limit=3)`

```python
def summarize(text, limit=3):
    summary = get_summary(text, limit)
    print('\n'.join(summary))
```

Explanation:
- Calls `get_summary` to obtain the summary.
- Prints the summary, separating each sentence with a newline.

### 3. Example Usage:

```python
text = "Natural language processing is a subfield of artificial intelligence that focuses on the interaction between computers and humans using natural language."
summarize(text, 2)
```

Explanation:
- Initializes a text variable with a sample content.
- Calls `summarize` function with the text and specifies a limit of 2 sentences for the summary.
