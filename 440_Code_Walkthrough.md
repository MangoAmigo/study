# Comprehensive Line-by-Line Code Walkthrough (Theory, Decisions, & Variables)

This document provides a highly detailed walkthrough of the codebase. It explains the theoretical reasoning behind the code, the meanings of variables, and the design decisions made along the way.

---

## 1. Production Environment Setup

Before running this code in a production or local environment, you need to set up the correct dependencies.

### Environment Requirements
To build and test this code, it is highly recommended to use a Python Virtual Environment (`venv` or `conda`) to avoid dependency conflicts. 

**Terminal Setup Commands:**
```bash
# 1. Create a virtual environment named 'nlp_env'
python3 -m venv nlp_env

# 2. Activate the environment (Linux/Mac)
source nlp_env/bin/activate
# (For Windows: nlp_env\Scripts\activate)

# 3. Install the required libraries
pip install numpy pandas matplotlib seaborn scikit-learn tensorflow transformers datasets gensim nltk wordcloud torch
```

**Why these specific libraries?**
* `tensorflow` provides the Keras API needed to build the LSTMs and RNNs.
* `transformers` and `datasets` (by HuggingFace) are required to download and train BERT.
* `scikit-learn` is used for TF-IDF, traditional ML models, and evaluation metrics (Macro F1).

---

## 2. Shared Code: Data Loading & Splitting

```python
1: import pandas as pd
2: from sklearn.model_selection import train_test_split
3:
4: # Load the datasets
5: train_df = pd.read_csv("medical_tc_train.csv")
6: test_df = pd.read_csv("medical_tc_test.csv")
7: 
8: # Split the training data into Train and Validation
9: X_train_text, X_val_text, y_train_id, y_val_id = train_test_split(
10:    train_df["medical_abstract"], 
11:    train_df["condition_label"] - 1, 
12:    test_size=0.2, 
13:    random_state=42, 
14:    stratify=train_df["condition_label"]
15: )
```

### Variable Meanings:
* `train_df`, `test_df`: Pandas DataFrames containing the raw CSV data (rows and columns).
* `X_train_text`: The raw English medical abstracts used for training.
* `y_train_id`: The mathematical labels (0 to 4) corresponding to the disease classes.

### Code Decisions & Theory:
* **Line 11 (`condition_label - 1`):** The original CSV file labels the diseases as `1, 2, 3, 4, 5`. However, Neural Networks strictly require classes to start at `0` (i.e., `0, 1, 2, 3, 4`) for the `sparse_categorical_crossentropy` math to work. Subtracting 1 fixes this instantly.
* **Line 12 (`test_size=0.2`):** We dedicate 20% of the training data purely for "Validation". Validation data is used to test the model at the end of every epoch during training to ensure it isn't memorizing (overfitting).
* **Line 14 (`stratify`):** *Crucial Decision.* Because the dataset is heavily imbalanced (mostly "General Pathological Conditions"), if we split randomly, the validation set might contain zero rare diseases. `stratify` mathematically guarantees that the 80/20 split maintains the exact percentage distribution of classes.

---

## 3. Shared Code: Keras Preprocessing (For RNNs)

```python
16: from tensorflow.keras.preprocessing.text import Tokenizer
17: from tensorflow.keras.preprocessing.sequence import pad_sequences
18:
19: num_words = 10000
20: max_len = 50
21:
22: tokenizer = Tokenizer(num_words=num_words, oov_token="<OOV>")
23: tokenizer.fit_on_texts(X_train_text)
24:
25: X_train_seq = pad_sequences(tokenizer.texts_to_sequences(X_train_text), maxlen=max_len, padding="post")
```

### Variable Meanings:
* `num_words`: The maximum vocabulary size. The model will only learn the top 10,000 most frequent words.
* `max_len`: The strict sequence limit (50 words) enforced on every abstract.
* `X_train_seq`: The final 2D matrix of numbers ready to be fed into the neural network.

### Code Decisions & Theory:
* **Line 22 (`oov_token="<OOV>"`):** If the model encounters a word in the Validation set that wasn't in the top 10,000 training words, it would normally crash or ignore it. The `OOV` (Out Of Vocabulary) token safely replaces unknown words with a designated integer.
* **Line 23 (`fit_on_texts`):** This maps every English word to an integer (e.g., "patient" $\rightarrow 1$). *Decision:* Notice we only fit on `X_train_text`. We never fit on the validation or test sets, because in the real world, the AI cannot "peek" at data it hasn't seen yet.
* **Line 25 (`padding="post"`):** *Theory:* RNNs use matrix multiplication, which requires fixed-size dimensions. If an abstract is 30 words, `pad_sequences` appends 20 zeros to the *end* (`post`) to reach 50. If it's 600 words, it truncates (chops off) the end down to 50. 

---

## 4. Member 4: Bidirectional LSTM

```python
26: from tensorflow.keras.models import Sequential
27: from tensorflow.keras.layers import Embedding, LSTM, Dense, Dropout, Bidirectional
28:
29: model_bilstm_1 = Sequential([
30:     Embedding(input_dim=num_words, output_dim=32, input_length=max_len),
31:     Bidirectional(LSTM(32)),
32:     Dropout(0.5),
33:     Dense(5, activation="softmax"),
34: ])
35:
36: model_bilstm_1.compile(optimizer="adam", loss="sparse_categorical_crossentropy", metrics=["accuracy"])
37: model_bilstm_1.fit(X_train_seq, y_train_id, validation_data=(X_val_seq, y_val_id), batch_size=64, epochs=5)
```

### Variable Meanings:
* `output_dim=32`: The size of the word vector. Every word is represented by an array of 32 float numbers.
* `LSTM(32)`: The LSTM layer contains 32 internal memory units (neurons).
* `batch_size=64`: The model processes 64 abstracts simultaneously before updating its weights.

### Code Decisions & Theory:
* **Line 29 (`Sequential`):** Keras API allowing us to stack neural network layers like a sandwich.
* **Line 30 (`Embedding`):** *Theory:* Maps the integer IDs to dense semantic vectors. The network updates these 32-dimensional vectors during training to understand relationships (e.g., pulling "tumor" and "cancer" closer together in math space).
* **Line 31 (`Bidirectional`):** *Theory:* A standard LSTM reads word 1 to word 50. A Bidirectional wrapper creates a *second* LSTM that reads word 50 backward to word 1. This prevents the model from forgetting the beginning of the sentence due to vanishing gradients, giving it full context.
* **Line 32 (`Dropout(0.5)`):** *Decision:* Deep models overfit easily. This mathematically forces 50% of the neurons to output `0` randomly during training. It prevents the network from memorizing specific training phrases.
* **Line 33 (`activation="softmax"`):** *Theory:* The `Dense(5)` layer outputs 5 raw numbers. `softmax` converts these raw numbers into a probability distribution that adds up to 100% (e.g., 80% Neoplasm, 10% Cardiovascular, etc.).
* **Line 36 (`sparse_categorical_crossentropy`):** The loss function. It calculates the error between the predicted probability (e.g., 80%) and the true label (100%). It's `sparse` because our true labels are single integers (`0, 1, 2`) rather than one-hot encoded vectors (`[1,0,0,0,0]`).

---

## 5. Member 4: BERT Preprocessing & HuggingFace Trainer

```python
38: from transformers import AutoTokenizer, AutoModelForSequenceClassification, Trainer, TrainingArguments
39: from datasets import Dataset
40:
41: bert_tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
42:
43: def tokenize(batch):
44:     return bert_tokenizer(batch["text"], truncation=True, max_length=128)
45:
46: train_hf = Dataset.from_pandas(pd.DataFrame({"text": list(X_train_text), "label": list(y_train_id)}))
47: train_hf = train_hf.map(tokenize, batched=True).rename_column("label", "labels")
48:
49: bert_model = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=5)
50: 
51: training_args = TrainingArguments(output_dir="./bert", num_train_epochs=1, per_device_train_batch_size=16)
52: trainer = Trainer(model=bert_model, args=training_args, train_dataset=train_hf, eval_dataset=val_hf)
53: trainer.train()
```

### Variable Meanings:
* `bert-base-uncased`: The specific Google-trained model weight ID. "uncased" means it treats "Patient" and "patient" the same.
* `train_hf`: A highly optimized HuggingFace dictionary format replacing Pandas.

### Code Decisions & Theory:
* **Line 41 (`AutoTokenizer`):** *Theory:* BERT uses **WordPiece** tokenization, not the Keras integer tokenizer. It breaks words into subwords (`Cardiomyopathy` $\rightarrow$ `Cardio`, `myo`, `pathy`). We download the exact tokenizer used by Google so our inputs match BERT's pre-trained brain perfectly.
* **Line 44 (`max_length=128`):** *Decision:* Because Transformers process text simultaneously via the Attention Mechanism (unlike RNNs which read sequentially and lose memory), we can drastically increase the input length from 50 to 128 tokens, giving BERT vastly more medical context to work with.
* **Line 47 (`rename_column`):** HuggingFace's internal C++ code strictly looks for a column named `"labels"`. If it's named `"label"`, the model will crash during loss calculation.
* **Line 51 (`num_train_epochs=1`):** *Decision:* We only fine-tune for 1 epoch. BERT already fundamentally understands English from reading Wikipedia. Training it for 5 epochs (like the RNNs) would cause massive overfitting on our small 9,000-abstract dataset.
* **Line 52 (`Trainer`):** Instead of manually writing PyTorch gradient descent loops, HuggingFace's `Trainer` handles the complex math (backpropagation, dynamic padding, learning rate decay) under the hood.


---

## 6. The Hyperparameter Configurations (What They Mean)

Throughout the notebook, you will notice that every member tested 3 different "Configurations" for their models. This is called **Hyperparameter Tuning**. We do this to find the "sweet spot" where the model learns the data perfectly without memorizing it (overfitting) or failing to understand it (underfitting).

### Recurrent Model Configurations (LSTMs, GRUs, RNNs)
All recurrent models shared these three test settings:

* **Configuration 1: `Dropout(0.5)` and `epochs=5`**
  * **What it means:** The model trains for 5 complete passes over the dataset. During training, a massive **50%** of the neurons are randomly turned off. 
  * **The Goal:** Heavy regularization. We force the network to learn robust, core patterns because half its brain is missing during every step. This config generally performed the best because medical data is noisy and prone to overfitting.
* **Configuration 2: `Dropout(0.3)` and `epochs=5`**
  * **What it means:** The model still trains for 5 passes, but only **30%** of the neurons are turned off. 
  * **The Goal:** Testing if Configuration 1 was "too aggressive." Sometimes turning off 50% of the brain stops the model from learning complex details. We tested 30% to see if the model could extract deeper patterns without overfitting.
* **Configuration 3: `Dropout(0.2)` and `epochs=20`**
  * **What it means:** Only **20%** of the neurons are turned off, and the model is forced to read the dataset **20 times**.
  * **The Goal:** Pushing the model to its absolute limits. By dropping less neurons and training for 4x longer, we wanted to see how much it could learn. **Result:** This configuration almost universally failed because it *overfit*. By epoch 8 or 9, it memorized the training data and its validation score plummeted.

### BERT Configurations
Because BERT is a massive pre-trained Transformer, we didn't need to tweak its architecture. We only tuned its training duration:

* **Configuration 1: `epochs=1`**
  * **What it means:** We showed BERT our medical dataset exactly one time.
  * **The Goal:** Since BERT already understands the English language, we just wanted to gently "nudge" it toward medical terminology. This performed the best because 1 pass was enough for BERT to understand the 5 diseases without memorizing the noise.
* **Configuration 2 & 3: `epochs=2` and `epochs=3`**
  * **What it means:** Forcing BERT to read our dataset 2 or 3 times.
  * **The Goal:** Testing if extra exposure would yield better accuracy. **Result:** Because BERT is so incredibly smart, reading our small dataset 3 times caused it to immediately overfit. Its performance actually dropped, proving that with pre-trained Transformers, less is often more.

### Essential Definitions: Epochs & Dropout

If you are asked to define these terms fundamentally, here is the plain-English breakdown:

* **What is an Epoch?**
  * In machine learning, one **Epoch** means the neural network has read the *entire* training dataset from start to finish exactly one time. 
  * If `epochs=5`, the AI reads the same 9,240 medical abstracts five times over. Just like a student studying for an exam, reading the textbook multiple times helps the AI learn the patterns better. However, if it reads the book too many times (e.g., `epochs=20`), it stops understanding the core concepts and just starts *memorizing* the exact wording. This is called **Overfitting**.

* **What is Dropout?**
  * **Dropout** is a mathematical trick used to prevent Overfitting. 
  * Imagine you are a student learning to identify diseases, but your teacher randomly covers up 50% of your notes while you study. You are forced to look at the "big picture" rather than memorizing tiny, specific details. 
  * In code, `Dropout(0.5)` literally means that during every single step of training, the computer randomly turns off 50% of the neurons (brain cells) in the neural network. This forces the remaining neurons to work harder to understand the abstract, resulting in a model that generalizes much better to new, unseen data in the Test Set.
