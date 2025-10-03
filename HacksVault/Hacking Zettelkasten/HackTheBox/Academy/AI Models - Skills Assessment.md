# Description

The `IMDB dataset` introduced by Maas et al. (2011) provides a collection of movie reviews extracted from the Internet Movie Database, annotated for `sentiment analysis`. It includes 50,000 reviews split evenly into training and test sets, and its carefully curated mixture of positive and negative examples allows researchers to benchmark and improve various natural language processing techniques. The `IMDB dataset` has influenced subsequent work in developing vector-based word representations and remains a popular baseline resource for evaluating classification performance and model architectures in sentiment classification tasks ([Maas et al., 2011](http://www.aclweb.org/anthology/P11-1015)).

Your goal is to train a model that can predict whether a movie review is positive (`1`) or negative (`0`). You can download the dataset from the question, or [from here](https://academy.hackthebox.com/storage/modules/292/skills_assessment_data.zip).

Out of interest, these exact same techniques can be applied into things such as text moderation for instance.

# Datasets

```bash
-$ unzip skills_assessment_data.zip
```

## Loading

```python
import pandas as pd

# Load the training dataset
df = pd.read_json(
    "train.json",
)

# Display basic information about the datasets
print("-------------------- HEAD --------------------")
print(df.head())
print("-------------------- DESCRIBE --------------------")
print(df.describe())
print("-------------------- INFO --------------------")
print(df.info())

# Check for missing values
print("Missing values:\n", df.isnull().sum())

# Check for duplicates
print("Duplicate entries:", df.duplicated().sum())

# Remove duplicates if any
df = df.drop_duplicates()
```

**Results**

```bash
-------------------- HEAD --------------------
                                                text  label
0  Bromwell High is a cartoon comedy. It ran at t...      1
1  Homelessness (or Houselessness as George Carli...      1
2  Brilliant over-acting by Lesley Ann Warren. Be...      1
3  This is easily the most underrated film inn th...      1
4  This is not the typical Mel Brooks film. It wa...      1
-------------------- DESCRIBE --------------------
             label
count  25000.00000
mean       0.50000
std        0.50001
min        0.00000
25%        0.00000
50%        0.50000
75%        1.00000
max        1.00000
-------------------- INFO --------------------
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 25000 entries, 0 to 24999
Data columns (total 2 columns):
 #   Column  Non-Null Count  Dtype 
---  ------  --------------  ----- 
 0   text    25000 non-null  object
 1   label   25000 non-null  int64 
dtypes: int64(1), object(1)
memory usage: 390.8+ KB
None
Missing values:
 text     0
label    0
dtype: int64
Duplicate entries: 96
```
# Preprocessing


```python
import nltk
import re
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
from nltk.stem import PorterStemmer

# Download the necessary NLTK data files
nltk.download("punkt")
nltk.download("punkt_tab")
nltk.download("stopwords")

print("=== BEFORE ANY PREPROCESSING ===") 
print(df.head(5))

# Convert all message text to lowercase
df["text"] = df["text"].str.lower()
print("\n=== AFTER LOWERCASING ===")
print(df["text"].head(5))

# Remove non-essential punctuation and numbers, keep useful symbols like $ and !
df["text"] = df["text"].apply(lambda x: re.sub(r"[^a-z\s$!]", "", x))
print("\n=== AFTER REMOVING PUNCTUATION & NUMBERS (except $ and !) ===")
print(df["text"].head(5))

# Split each message into individual tokens
df["text"] = df["text"].apply(word_tokenize)
print("\n=== AFTER TOKENIZATION ===")
print(df["text"].head(5))

# Stem each token to reduce words to their base form
stemmer = PorterStemmer()
df["text"] = df["text"].apply(lambda x: [stemmer.stem(word) for word in x])
print("\n=== AFTER STEMMING ===")
print(df["text"].head(5))

# Rejoin tokens into a single string for feature extraction
df["text"] = df["text"].apply(lambda x: " ".join(x))
print("\n=== AFTER JOINING TOKENS BACK INTO STRINGS ===")
print(df["text"].head(5))

from sklearn.feature_extraction.text import CountVectorizer

# Initialize CountVectorizer with bigrams, min_df, and max_df to focus on relevant terms
vectorizer = CountVectorizer(min_df=1, max_df=0.9, ngram_range=(1, 2))

# Fit and transform the message column
X = vectorizer.fit_transform(df["text"])

# Labels (target variable)
y = df["label"]
```

**Results**

```bash
=== BEFORE ANY PREPROCESSING ===
                                                text  label
0  Bromwell High is a cartoon comedy. It ran at t...      1
1  Homelessness (or Houselessness as George Carli...      1
2  Brilliant over-acting by Lesley Ann Warren. Be...      1
3  This is easily the most underrated film inn th...      1
4  This is not the typical Mel Brooks film. It wa...      1

=== AFTER LOWERCASING ===
0    bromwell high is a cartoon comedy. it ran at t...
1    homelessness (or houselessness as george carli...
2    brilliant over-acting by lesley ann warren. be...
3    this is easily the most underrated film inn th...
4    this is not the typical mel brooks film. it wa...
Name: text, dtype: object

=== AFTER REMOVING PUNCTUATION & NUMBERS (except $ and !) ===
0    bromwell high is a cartoon comedy it ran at th...
1    homelessness or houselessness as george carlin...
2    brilliant overacting by lesley ann warren best...
3    this is easily the most underrated film inn th...
4    this is not the typical mel brooks film it was...
Name: text, dtype: object

=== AFTER TOKENIZATION ===
0    [bromwell, high, is, a, cartoon, comedy, it, r...
1    [homelessness, or, houselessness, as, george, ...
2    [brilliant, overacting, by, lesley, ann, warre...
3    [this, is, easily, the, most, underrated, film...
4    [this, is, not, the, typical, mel, brooks, fil...
Name: text, dtype: object

=== AFTER REMOVING STOP WORDS ===
0    [bromwell, high, cartoon, comedy, ran, time, p...
1    [homelessness, houselessness, george, carlin, ...
2    [brilliant, overacting, lesley, ann, warren, b...
3    [easily, underrated, film, inn, brooks, cannon...
4    [typical, mel, brooks, film, much, less, slaps...
Name: text, dtype: object

=== AFTER STEMMING ===
0    [bromwel, high, cartoon, comedi, ran, time, pr...
1    [homeless, houseless, georg, carlin, state, is...
2    [brilliant, overact, lesley, ann, warren, best...
3    [easili, underr, film, inn, brook, cannon, sur...
4    [typic, mel, brook, film, much, less, slapstic...
Name: text, dtype: object

=== AFTER JOINING TOKENS BACK INTO STRINGS ===
0    bromwel high cartoon comedi ran time program s...
1    homeless houseless georg carlin state issu yea...
2    brilliant overact lesley ann warren best drama...
3    easili underr film inn brook cannon sure flaw ...
4    typic mel brook film much less slapstick movi ...
Name: text, dtype: object
```

# Training & Evaluation

**Training**

```bash
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline

# Build the pipeline by combining vectorization and classification
pipeline = Pipeline([
    ("vectorizer", vectorizer),
    ("classifier", MultinomialNB())
])

# Define the parameter grid for hyperparameter tuning
param_grid = {
    "classifier__alpha": [0.01, 0.1, 0.15, 0.2, 0.25, 0.5, 0.75, 1.0]
}

# Perform the grid search with 10-fold cross-validation and the F1-score as metric
grid_search = GridSearchCV(
    pipeline,
    param_grid,
    cv=10,
    scoring="f1"
)

# Fit the grid search on the full dataset
grid_search.fit(df["text"], df["label"])

# Extract the best model identified by the grid search
best_model = grid_search.best_estimator_
print("Best model parameters:", grid_search.best_params_)
```

**Results**

```bash
Best model parameters: {'classifier__alpha': 1.0}
```


**Evaluation** and **saving**

```python
import numpy as np

# Load the test dataset
new_df = pd.read_json(
    "test.json",
)

# Preprocess function that mirrors the training-time preprocessing
def preprocess_rev(text):
    text = text.lower()
    text = re.sub(r"[^a-z\s$!]", "", text)
    tokens = word_tokenize(text)
    # tokens = [word for word in tokens if word not in stop_words]                        # with stop words accuracy drops to 0.88
    tokens = [stemmer.stem(word) for word in tokens]
    return " ".join(tokens)

# Preprocess and vectorize messages
new_df["processed_text"] = new_df["text"].apply(preprocess_rev)

print(new_df["processed_text"].head(10))

# Transform preprocessed messages into feature vectors
X_new = best_model.named_steps["vectorizer"].transform(new_df["processed_text"])

# Predict with the trained classifier
predictions = best_model.named_steps["classifier"].predict(X_new)
prediction_probabilities = best_model.named_steps["classifier"].predict_proba(X_new)

# Display predictions and probabilities for each evaluated message
for i, text in enumerate(new_df["text"]):
    prediction = "Good Review" if predictions[i] == 1 else "Not Good Review"
    good_probability = prediction_probabilities[i][1]  # Probability of being good review
    bad_probability = prediction_probabilities[i][0]   # Probability of being not good
    
    print(f"Message: {text}")
    print(f"Prediction: {prediction}")
    print(f"Good review Probability: {good_probability:.2f}")
    print(f"Not-good review Probability: {bad_probability:.2f}")
    print("-" * 10)


import joblib

# Save the trained model to a file for future use
model_filename = 'movie_reviews_model.joblib'
joblib.dump(best_model, model_filename)

print(f"Model saved to {model_filename}")
```

**Results**

```
0    went saw movi last night coax friend mine ill ...
1    actor turn director bill paxton follow promis ...
2    recreat golfer knowledg sport histori pleas di...
3    saw film sneak preview delight cinematographi ...
4    bill paxton taken true stori us golf open made...
5    saw film septemb st indianapoli one judg heart...
6    mayb im read much wonder much hand hongsheng d...
7    felt film mani good qualiti cinematographi cer...
8    movi amaz fact real peopl portray real life ex...
9    quit may much exit preordain ident drug withdr...

Name: processed_text, dtype: object
Message: I went and saw this movie last night after being coaxed to by a few friends of mine. I'll admit that I was reluctant to see it because from what I knew of Ashton Kutcher he was only able to do comedy. I was wrong. Kutcher played the character of Jake Fischer very well, and Kevin Costner played Ben Randall with such professionalism. The sign of a good movie is that it can toy with our emotions. This one did exactly that. The entire theater (which was sold out) was overcome by laughter during the first half of the movie, and were moved to tears during the second half. While exiting the theater I not only saw many women in tears, but many full grown men as well, trying desperately not to let anyone see them crying. This movie was great, and I suggest that you go see it before you judge.
Prediction: Good Review
Good review Probability: 1.00
Not-good review Probability: 0.00
--------------------------------------------------

Message: Actor turned director Bill Paxton follows up his promising debut, the Gothic-horror "Frailty", with this family friendly sports drama about the 1913 U.S. Open where a young American caddy rises from his humble background to play against his Bristish idol in what was dubbed as "The Greatest Game Ever Played." I'm no fan of golf, and these scrappy underdog sports flicks are a dime a dozen (most recently done to grand effect with "Miracle" and "Cinderella Man"), but some how this film was enthralling all the same.<br /><br />The film starts with some creative opening credits (imagine a Disneyfied version of the animated opening credits of HBO's "Carnivale" and "Rome"), but lumbers along slowly for its first by-the-numbers hour. Once the action moves to the U.S. Open things pick up very well. Paxton does a nice job and shows a knack for effective directorial flourishes (I loved the rain-soaked montage of the action on day two of the open) that propel the plot further or add some unexpected psychological depth to the proceedings. There's some compelling character development when the British Harry Vardon is haunted by images of the aristocrats in black suits and top hats who destroyed his family cottage as a child to make way for a golf course. He also does a good job of visually depicting what goes on in the players' heads under pressure. Golf, a painfully boring sport, is brought vividly alive here. Credit should also be given the set designers and costume department for creating an engaging period-piece atmosphere of London and Boston at the beginning of the twentieth century.<br /><br />You know how this is going to end not only because it's based on a true story but also because films in this genre follow the same template over and over, but Paxton puts on a better than average show and perhaps indicates more talent behind the camera than he ever had in front of it. Despite the formulaic nature, this is a nice and easy film to root for that deserves to find an audience.
Prediction: Good Review
Good review Probability: 1.00
Not-good review Probability: 0.00
--------------------------------------------------
```
