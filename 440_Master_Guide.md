# The Ultimate NLP Masterclass: Decoding 440.ipynb

Welcome! This guide is designed to bridge the gap between the theoretical concepts taught in **Lectures 1 through 7** and the actual Python code written in `440.ipynb`. 

By reading this, you will understand exactly how the mathematical theories from Dr. Farig Sadeque's slides were translated into working models for Medical Abstract Multi-Class Classification.

---

## 1. Introduction: What are we actually doing? *(Ref: Lecture 1 - Introduction)*

### What is NLP?
As discussed in Lecture 1, Natural Language Processing (NLP) focuses on computational models of language. Computers only understand numbers, so the hardest part of NLP is mapping discrete language (words) into continuous mathematical spaces.

### The Project Goal: Text Classification
Our specific task is **Multi-Class Text Classification**. We are classifying medical abstracts into 5 disease categories: Neoplasms, Digestive System Diseases, Nervous System Diseases, Cardiovascular Diseases, and General Pathological Conditions. 

---

## 2. Phase 1: Data Preparation & EDA *(Ref: Lecture 2 - Linguistics Essentials)*

Before training an AI, we must load and preprocess the text according to the linguistics essentials taught in class.

### EDA & Class Imbalance
We plot a bar chart of the categories and notice a huge **class imbalance**. There are over 3,800 abstracts for "General" but only 1,195 for "Digestive". If the AI simply guesses the majority class ("General"), it gets a deceptively high Accuracy. This is why we rely on **Macro F1** for evaluation, as it treats all classes equally.

### Preprocessing & Tokenization
As taught in Lecture 2, we can't feed raw text to models. 
1. **Lowercasing & Normalization:** "Tumor" and "tumor" are unified to the same token.
2. **Stop-words Removal:** Words with low semantic value ("the", "is") are removed for our classical bag-of-words models.
3. **Tokenization:** Using the Keras `Tokenizer`, we assign an integer ID to the top 10,000 words in our corpus. Rare words are mapped to an `<OOV>` (Out of Vocabulary) token.
4. **Padding/Truncating:** Because neural networks require fixed-length sequences, we set a `max_len` of 50. Longer sequences are chopped, and shorter ones are padded with zeros.

---

## 3. Phase 2: Word Representations *(Ref: Lecture 4 - Word Representation)*

How do we mathematically represent words? We used several techniques from Lecture 4.

### 1. TF-IDF (Term Frequency - Inverse Document Frequency)
* **Theory:** TF-IDF weighs a term based on how often it appears in a document (TF) multiplied by how rare it is across all documents (IDF). A word like "heart" in a cardiovascular abstract gets a high score, whereas "patient" gets a low score because it appears everywhere.
* **N-grams:** We used both Unigrams and Bigrams (Lecture 2 concept), allowing the model to capture two-word contexts like "heart attack".

### 2. Word2Vec (CBOW vs Skip-Gram)
* **Theory:** Introduced by Mikolov et al. in 2013, Word2Vec learns dense vectors (embeddings) by predicting word contexts. 
  * **CBOW (Continuous Bag of Words):** Predicts a target word given its surrounding context.
  * **Skip-gram:** Predicts the surrounding context given a target word.
In our shared section, we trained both versions using Gensim and averaged the vectors to represent each abstract.

### 3. Embeddings for RNNs
Instead of using Word2Vec, our RNNs used trainable Keras `Embedding` layers. They initialize randomly and learn the semantic vectors specifically optimized for our medical classification task during backpropagation.

---

## 4. Phase 3: Classical Machine Learning *(Ref: Lecture 3 - ML Essentials)*

Before deep learning, Members 2 and 3 tested classical, statistical algorithms.

### Logistic Regression
Despite its name, it is a linear classifier. It applies a sigmoid (or softmax for multi-class) function to a linear combination of TF-IDF features to output class probabilities.

### Multinomial Naive Bayes
* **Theory:** Based on Bayes' Theorem. It calculates the probability of a class given the words in the document. It is "naive" because it assumes the presence of one word is conditionally independent of another, which is linguistically false, but works surprisingly well for text classification.

### Random Forest
An ensemble method that builds hundreds of decision trees randomly and takes a majority vote. It handles sparse TF-IDF matrices well but can be computationally heavy.

---
## 5. Phase 4: Recurrent Neural Networks *(Ref: Lecture 6 - RNN)*

Members 1, 2, 3, and 4 built Neural Networks to capture sequential dependencies in text (which Bag-of-Words and TF-IDF completely ignore).

### Simple RNN (Member 1)
* **Theory:** At each time step `t`, the RNN combines the current word input `x_t` with the hidden state from the previous step `h_{t-1}`. 
* **The Flaw (Vanishing Gradient):** As taught in Lecture 6, SimpleRNNs suffer from the Vanishing Gradient problem during Backpropagation Through Time (BPTT). Over a sequence of 50 tokens, the gradient shrinks exponentially, meaning the network forgets the beginning of the abstract. This is why Member 1's SimpleRNN performed the worst (0.30 Macro F1).

### LSTM and GRU (Members 2 & 3)
* **Theory:** To fix the vanishing gradient, LSTMs (Hochreiter & Schmidhuber, 1997) and GRUs (Cho et al., 2014) introduce mathematical **gates**.
  * **LSTM:** Uses a Forget Gate, Input Gate, and Output Gate alongside a Cell State. It explicitly decides what information to remember and what to forget.
  * **GRU:** A streamlined version of LSTM combining the forget and input gates into a single Update Gate.
These gating mechanisms allowed Members 2 and 3's models to vastly outperform the SimpleRNN.

### Bidirectional Networks (Members 1, 3, 4)
* **Theory:** A word's context often depends on future words. A Bidirectional RNN uses two separate hidden layers—one processing the sequence left-to-right, and the other right-to-left. Their outputs are concatenated, providing full context for every word.

### Hyperparameters used in the Code
* **`batch_size=64`**: The model processes 64 abstracts at a time before updating weights.
* **`epochs=5` vs `20`**: Training for too many epochs leads to **Overfitting** (memorizing the training data). We noticed 20 epochs caused the validation loss to spike, so 5 epochs was optimal.
* **`Dropout(0.5)`**: A regularization technique that randomly zeroes out 50% of the neurons during training to prevent overfitting.
* **`optimizer="adam"`**: An adaptive learning rate optimization algorithm.

---

## 6. Phase 5: The State-of-the-Art BERT *(Ref: Lecture 7 - Translation Transformer)*

RNNs process sequentially, which is slow and inherently limits long-range memory. In Lecture 7, we learned about the **Transformer** (Vaswani et al., 2017), which changed NLP forever.

### Why BERT Dominates
1. **Self-Attention Mechanism:** Instead of reading left-to-right, transformers process all words simultaneously. The Attention mechanism calculates how much "focus" every word in a sentence should place on every other word.
2. **Pre-training:** Our RNNs had to learn English purely from 9,000 abstracts. **BERT** (Bidirectional Encoder Representations from Transformers) was pre-trained by Google on the entirety of English Wikipedia. It already possesses a deep structural understanding of language.
3. **Subword Tokenization (WordPiece):** BERT handles rare medical jargon perfectly by breaking unknown words into known subwords (e.g., "Cardio" + "myopathy").
4. **Sequence Length:** Because Transformers don't suffer from vanishing gradients, Member 4 could feed BERT a sequence length of `128` tokens (compared to the RNNs' 50 tokens), giving it vastly more context.

### HuggingFace Trainer
Because BERT is already pre-trained, Member 4 simply **fine-tuned** it on our dataset for just 1 epoch using HuggingFace's `Trainer` API. It instantly outperformed all other models with a 0.62 Macro F1.

---

## 7. Phase 6: Final Evaluation

At the end of the notebook, Member 4 evaluated all models on the unseen Test Set.

### The Confusion Matrix
A heatmap grid showing true labels vs predicted labels. 
* **SimpleRNN:** Misclassified almost everything into the majority class ("General Pathological Conditions") because its vanishing gradient prevented it from learning distinct features.
* **BERT:** Formed a strong diagonal line, meaning it correctly mapped features to their respective specific diseases (like Neoplasms and Cardiovascular).

### The Ultimate Conclusion
1. **BERT won easily (0.62 F1).** Transformer Attention + Pre-trained knowledge + 128 tokens = State-of-the-art results.
2. **SimpleRNN lost terribly (0.30 F1).** Sequential processing + Vanishing Gradients + 50 tokens = Total failure.
3. **TF-IDF + Classical ML was surprisingly robust.** Despite lacking sequence awareness, just counting word occurrences (TF-IDF) is remarkably effective for highly specific medical jargon.

---
**Congratulations! You now have a complete, expert-level understanding of `440.ipynb`, mapping every line of code directly back to the theoretical concepts taught in Dr. Farig Sadeque's lectures.**

## 8. What We Didn't Use (And Why) - The Defense Section

Professors love to ask *why* you didn't do something during a Viva. Here is the theoretical defense for techniques that were taught in the lectures but intentionally left out of the code:

### 1. Stemming and Lemmatization *(Lecture 2)*
* **What it is:** Reducing words to their root forms (e.g., "running" $\rightarrow$ "run"). 
* **Why we excluded it:** We are working with highly specialized clinical text. Aggressive algorithms like the Porter Stemmer often blindly chop off prefixes and suffixes. In the medical field, altering a suffix can completely change a word's meaning (e.g., "myopathy" vs "myocarditis") or destroy critical acronyms like "ECG". We chose to preserve the raw medical terminology so BERT and TF-IDF could learn the precise meanings of specific diseases.

### 2. Accuracy as the Primary Metric *(Lecture 3)*
* **What it is:** Simply dividing correct predictions by total predictions.
* **Why we excluded it:** As shown in our EDA, our dataset suffers from severe **class imbalance** (33% of the data is a single class). If the model lazily guesses the majority class every time, Accuracy will look artificially high, masking the fact that the model is failing on minority classes (like Digestive Diseases). We evaluated using **Macro F1** instead, because it calculates the score for each class independently and averages them, ensuring minority classes are treated as equally important.

### 3. Seq2Seq Models & Generation *(Lectures 6 & 7)*
* **What it is:** Sequence-to-Sequence models take an input sequence and generate a completely new output sequence (like Google Translate translating English to French).
* **Why we excluded it:** Our project is purely a **Multi-Class Classification** problem, not a generative translation problem. Our models only needed to output a single probability distribution (a math vector of 5 numbers representing the 5 disease classes), so adding a Decoder network to generate text would be completely irrelevant.

### 4. Long Sequence Lengths for RNNs (e.g., 500 tokens)
* **What it is:** We capped our RNNs (SimpleRNN, LSTM, GRU) at a `max_len` of 50 tokens, meaning anything after 50 words was truncated. 
* **Why we didn't use 500+ tokens:** Because of the **Vanishing Gradient** problem discussed in Lecture 6, recurrent models cannot retain memory over long sequences. If we forced the LSTM to read 500 words sequentially, the math would become highly unstable during Backpropagation, and the training time would take hours instead of minutes. We utilized BERT when we needed a longer token window (128 tokens) because Transformers process simultaneously, not sequentially.
