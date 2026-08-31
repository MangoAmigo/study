# The Ultimate NLP Exam Study Guide

Welcome to the definitive study guide for your Natural Language Processing exam! This document comprehensively covers the theoretical concepts, mechanisms, pros/cons, and differences across Lectures 1 through 6. 

It is written specifically for a beginner. Read it top-to-bottom, and you will be fully prepared for your exam.

---

## Lecture 1: Introduction to NLP

### What is NLP?
Natural Language Processing (NLP) is the intersection of Computer Science, Artificial Intelligence, and Linguistics. It focuses on programming computers to process, analyze, and "understand" large amounts of natural human language data.

### The Core Challenge: Ambiguity
Unlike programming languages (like Python or C++), human language is inherently ambiguous. 
* **Lexical Ambiguity:** A word can have multiple meanings. (e.g., "Bank" can mean a financial institution or a river edge).
* **Syntactic Ambiguity:** A sentence can be parsed in multiple ways. (e.g., "I saw the man with the telescope." Who has the telescope?)
* **Semantic Ambiguity:** The literal meaning differs from the intended meaning (e.g., sarcasm or idioms).

### Applications of NLP
* Text Classification (Spam detection, sentiment analysis)
* Machine Translation (Google Translate)
* Question Answering (Siri, ChatGPT)
* Information Extraction (Named Entity Recognition - finding names/dates in text)

---

## Lecture 2: Linguistics Essentials

Before doing any Machine Learning, we must prepare the text. This is called the NLP Pipeline.

### 1. Tokenization
Computers can't read paragraphs. **Tokenization** is the process of breaking a stream of text up into words, phrases, or symbols called "tokens". 
* *Example:* "I love NLP!" $\rightarrow$ `["I", "love", "NLP", "!"]`

### 2. Text Normalization
We must standardize text so the computer doesn't treat variations of the same word as entirely different entities.
* **Lowercasing:** "Apple" and "apple" become identical.
* **Stop-Words Removal:** Removing highly frequent words that carry little semantic meaning ("the", "is", "at"). 
  * *Pros:* Reduces data size and speeds up processing.
  * *Cons:* Destroys syntactic structure (e.g., "To be or not to be" becomes entirely stop-words).

### 3. Stemming vs. Lemmatization
Both techniques reduce words to their base form.
* **Stemming:** A crude, rule-based approach that simply chops off the ends of words. 
  * *Example:* "running" $\rightarrow$ "run", "better" $\rightarrow$ "bett".
  * *Pros:* Very fast.
  * *Cons:* Can create non-existent words (e.g., "bett").
* **Lemmatization:** A smart, dictionary-based approach that looks at vocabulary and morphological analysis to return the base dictionary form (the "lemma").
  * *Example:* "running" $\rightarrow$ "run", "better" $\rightarrow$ "good".
  * *Pros:* Highly accurate and linguistically correct.
  * *Cons:* Slower and computationally expensive.

### 4. Bag of Words (BoW) & N-Grams
* **Bag of Words:** Represents text by counting the frequency of words, completely ignoring grammar and word order. 
* **N-Grams:** Instead of single words, we look at sequences of *N* words.
  * **Unigram (1-gram):** "I", "love", "dogs"
  * **Bigram (2-gram):** "I love", "love dogs"
  * *Pros:* Captures local context (e.g., "not good" is captured as a bigram, whereas unigrams would separate "not" and "good").

---

## Lecture 3: Machine Learning Essentials

How do we teach computers to make predictions?

### Supervised vs. Unsupervised Learning
* **Supervised Learning:** The model is trained on data that comes with answers (labels). *Example:* Predicting if an email is spam (Label = 1) or not spam (Label = 0).
* **Unsupervised Learning:** The model is given data without answers and must find hidden structures (like clustering similar news articles together).

### Classification Models
Classification is a supervised learning task where the output is a discrete category.

1. **Multinomial Naive Bayes (Generative Model)**
   * **Theory:** Based on Bayes' Theorem. It calculates the probability of a document belonging to a class based on the frequencies of words inside it.
   * **The "Naive" Assumption:** It assumes every word's appearance is completely independent of the others (which is false in real English, but works great for math).
   * **Pros:** Extremely fast, requires very little training data, handles high dimensions well.
   * **Cons:** Cannot learn interactions between features (words).

2. **Logistic Regression (Discriminative Model)**
   * **Theory:** Fits a linear boundary through the data space and uses the Logistic (Sigmoid/Softmax) function to output a probability between 0 and 1.
   * **Pros:** Highly interpretable (you can look at weights to see which words matter most), doesn't assume word independence.
   * **Cons:** Struggles if the data is highly non-linear.

### Evaluation Metrics
If a dataset has 99 normal emails and 1 spam email, a lazy model can guess "normal" 100% of the time and achieve 99% Accuracy. This is why Accuracy is dangerous for **imbalanced data**.
* **Precision:** Out of all the emails the model *claimed* were spam, how many actually were? (Quality of predictions).
* **Recall:** Out of all the *actual* spam emails in reality, how many did the model manage to find? (Quantity found).
* **F1-Score:** The harmonic mean of Precision and Recall. Use this when you have imbalanced classes!

---
## Lecture 4: Word Representation

Words must become vectors (arrays of numbers) before models can use them.

### 1. One-Hot Encoding
* **Mechanism:** A vocabulary of 10,000 words means every word is a vector of 10,000 zeros, with a single `1` at the word's index.
* **Pros:** Simple to understand.
* **Cons:** The vectors are massive, highly sparse (mostly zeros), and contain **zero semantic meaning**. The math distance between "cat" and "dog" is the exact same as "cat" and "car".

### 2. TF-IDF (Term Frequency - Inverse Document Frequency)
* **Mechanism:** 
  * `TF`: How many times a word appears in this specific document.
  * `IDF`: A penalty applied to words that appear in *every* document (like "the").
* **Pros:** Gives high weight to important, rare keywords.
* **Cons:** Still results in massive, sparse vectors. Still doesn't understand meaning or context.

### 3. Word Embeddings (Word2Vec)
* **Mechanism:** Created by Google (Mikolov et al., 2013). Instead of 10,000-dimension sparse vectors, words are mapped into a small, dense 300-dimension space. Words with similar meanings are mapped mathematically close together.
* **Two Training Architectures:**
  1. **CBOW (Continuous Bag of Words):** The model looks at the surrounding context words to predict the missing center word. (Fast, better for frequent words).
  2. **Skip-gram:** The model takes a single center word and tries to predict the surrounding context words. (Slower, but much better for rare words).
* **Pros:** Captures deep semantic meaning (e.g., Vector(King) - Vector(Man) + Vector(Woman) = Vector(Queen)).
* **Cons:** Cannot handle polysemy (words with multiple meanings, like "bank", get a single average vector).

---

## Lecture 5: Sequence Learning

Traditional models (like Naive Bayes or Logistic Regression) treat text as a "Bag of Words" where order doesn't matter. But in reality, **order is everything**. "The dog bit the man" is very different from "The man bit the dog".

### Language Models (LMs)
A Language Model assigns a probability to a sequence of words. It tries to answer: "What is the probability of the next word being X given the previous words?"

### Markov Assumption & N-gram LMs
Because it's impossible to calculate the probability of a word based on *all* previous words in a book, we use the **Markov Assumption**: the probability of a word only depends on a fixed window of previous words (e.g., the last 2 words in a Trigram model).

### Evaluation: Perplexity
* **Perplexity** is how confused the model is by a sequence of text. 
* A lower perplexity means the model is less confused and is better at predicting the next word.

---

## Lecture 6: Recurrent Neural Networks (RNN)

To truly handle sequences and long-term memory, we introduced Recurrent Neural Networks (RNNs).

### Simple RNNs
* **Mechanism:** Unlike standard Feed-Forward networks which only go forward, an RNN has a loop. At time step `t`, it takes the current word `x_t` AND the hidden state (memory) from the previous step `h_{t-1}` to make a prediction.
* **Pros:** Can process sequences of any length. Considers word order.
* **The Fatal Flaw: Vanishing Gradient Problem**
  * When training an RNN, we use an algorithm called **Backpropagation Through Time (BPTT)**. 
  * Because the network multiplies gradients together over and over as it goes back through time, if the gradient is a fraction (e.g., 0.5), multiplying it by itself 50 times causes it to shrink to nearly zero ($0.5^{50} \approx 0$).
  * **Result:** The network completely forgets the beginning of the sentence by the time it reaches the end.

### LSTMs (Long Short-Term Memory)
Invented in 1997 by Hochreiter & Schmidhuber to explicitly solve the Vanishing Gradient problem.
* **Mechanism:** Introduces an internal "Cell State" (a conveyor belt of long-term memory) and three mathematical **Gates**:
  1. **Forget Gate:** Decides what useless information to throw away.
  2. **Input Gate:** Decides what new information to add to the cell state.
  3. **Output Gate:** Decides what the next hidden state should be.
* **Pros:** Can remember context over very long sequences. Solves vanishing gradients.
* **Cons:** Computationally expensive and slow to train (too many parameters).

### GRUs (Gated Recurrent Units)
Invented in 2014 by Cho et al. as a streamlined, faster version of the LSTM.
* **Mechanism:** Combines the Forget and Input gates into a single **Update Gate**. Does not have a separate Cell State.
* **Pros:** Trains much faster than LSTM while achieving very similar performance.

### Bidirectional RNNs
* **The Problem:** In standard RNNs, understanding a word only relies on the words *before* it. But sometimes, future words change the context (e.g., "He said the bark was rough" $\rightarrow$ tree bark vs dog bark).
* **The Solution:** A Bidirectional RNN uses two independent hidden layers. One reads the sentence forwards (left-to-right). The other reads it backwards (right-to-left). Their outputs are combined.
* **Pros:** Provides complete past and future context for every word.

---
**Good luck with your exam! You now know the theoretical architectures, flaws, and solutions that define modern NLP.**
