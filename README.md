README.md

# Kipo AI Machine Learning Internship: Take-Home Question 

## Task description

We have a dataset of ~8000 PDF documents called "datasheets," which describe the technical characteristics of a physical product. Our goal is to build a model that predicts one characteristic called an "IP Rating" of a product from the product's datasheet. We'll approach this task in the following steps, outlined in more detail below: 
1. Do exploratory data analysis to describe the characteristics of the data. 
2. Select an approach to featurizing the documents.
3. Train a classifier. 
4. Evaluate the performance of your classifier. 

You are free to use any existing libraries and tools you want. Please email your interviewer with any questions if you get stuck. Do not spend more than 4 hours on this task! Feel free to simplify the problem if needed. 

## Dataset description and background

A datasheet is a PDF document that describes the technical characteristics of a hardware product. One of those characteristics is the [IP ("Ingress Protection") rating](https://en.wikipedia.org/wiki/IP_code), which describes the extent to which a part prevents the entry of water and dust. Our task is to predict the IP rating of a part from the datasheet for that part. 

There are two ZIP files: one contains the original datasheet PDFs, and the other contains text files corresponding to the PDFs that contain the PDFs’ text. There are roughly 8,000 datasheets in each. 

The file `master.csv` lists metadata for these datasheets, including the part number, manufacturer, and the part's IP rating, among other things. The titles of the files in the ZIP folder correspond to the column `UID` in `master.csv`. The UID is a unique identifier for each datasheet file.  

The column `IP Rating` lists the IP ratings for each part and is the target variable for the task. A part can have one, multiple, or no IP rating. 

For example, the first row in `master.csv` describes the metadata for the datasheet `1.pdf`, and the text data from this datasheet is available in `1.txt`. The part describes in this datasheet has IP rating "IP67." 

## Step 1: Exploratory data analysis

First, we'll try to understand the datasheets and their features. Note that the datasheets and metadata are all collected from [this page on a distributor's website](https://www.mouser.com/c/connectors/circular-connectors). What do you observe about the data? Consider the labels, PDF and text data, etc.

This is an open-ended task. There is no right answer; our goal is just to see how you approach understanding new datasets. Explain your observations below:

### My Observations
**Missing values:**

* `Pricing` has significantly small number of missing values (17)
* `Lifecycle` has significantly large number of missing values (6372)
* Handle missing values for `Lifecycle`, `RoHS`, `IP Rating`, `Product`
* Other features can be filled in simply 
* 
* `Lifecycle`: **Nominal**, Mode = New Product (92%)
* `RoHS`: **Nominal/Ordinal**, Mode = RoHS Compliant (78%), RoHS Compliant By Exemption (18%)
* `IP Rating`: **Ordinal**, Mode = IP67(37%), IP68(15%), IP50(10%)
* `Product`: **Nominal**, Mode = 'Connectors'(57%), Receptacles(14%), Accessories(8%), Plugs(6%)

**Categorical Features:**

Some columns, such as `Manufacturer`, `Product`, `Contact Gender`, and `Termination Style`, contain categorical data.
These categorical features will need to be encoded before being used in the model.
* `Lifecycle`: **Nominal**
    * EOL Products reaching end of lifecycle may have different characteristics compared ot new
    * New vs EOL: change in manufacturing processes/materials/design revisions
    * New vs Mature: Undergo different levels of testing/level of reliability
    * Updates and Revisions: **Preliminary Release** subject to revisions *(might be insignificant %)*
* `RoHS`: **Nominal/Ordinal** Restriciton of Hazardous Substances(lead, mercury)
    * Presence/absence of **hazardous** subsatnces -> prone to corrosion can affect durability/reliability -> IP Rating
    * Comply with RoHS regulations -> stricter QC -> consistent performance and better IP
* `Product`: **Nominal**
    * Physical Characteristics: Varying physical characteristics that affect IP
    * **Connectors** and **cable clamp/backshells** typically have openings that may require additional protection against environmental elements compared to **plugs**/**housings**
    * Design Usage/Application: **Gaskets**/**Seals** inherently prioritize IP to seal against water/dust
* `Contact Gender`: **Nominal/Ordinal**, Plug vs Socket: Female = receptacle receiving male connectors
    * Design of female connectors may incorporate additional sealing features
* `Termination Style`: **Nominal**
    * Design: **Crimp**/**Cramp**/**Solder** terminations -> more secure/stable/better protection
    
**Duplicates:**
* Some entries have have same `UIDs`(same PDF datasheet file)
* But have different `Pricing`, `Availability`, similar `Part Number`
* Conclusion: Subsets of same UID/Component with slightly different variations
* Investigate if variations of component have same IP rating

**Unique values:**
* `RoHS`: 5
* `Lifecycle`: 5
* `IP Rating`: 35
* `Product`: 31
* `Contact Gender`: 18
* `Termination Style`: 27

## Step 2: Data preparation

In light of your observations above, pre-process, clean, and prepare the data as needed. Keep in mind the goal of building a classification model. 

Feel free to make simplifying assumptions - no need to utilize every document in the dataset. Ask your interviewer if you need guidance. 

## Step 3: Select a vectorizer

For simplicity, we'll just consider with text data, so we'll deal with the TXT files rather than the original PDFs. Consider these three choices for producing numerical vectors/features from your text data: 

(A) Use [word embeddings](https://spacy.pythonhumanities.com/01_03_word_vectors.html) (e.g. from SpaCy's Word2Vec model) and compute the average embedding of all or some of the words in the document.
(B) Use sentence embeddings from a Large Language Model (e.g. OpenAI's text-embedding-ada-002) to embed the text in the document.
(C) A [word count vectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html), which lists the number of times each word or n-gram appears in a document.
(D) [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf), which lists the frequency of words in a document while adjusting for words that are generally more frequent. 

Consider the differences between these approaches and how we might use in our context. Select the approach you think is best. Explain why you chose this approach, and its pros and cons relative to other approaches, below: 

### My choice and explanation:
**Word Embeddings** *(Chosen)*
* Captures semantic meaning of words and their context(good for categorical data like `Product`)
* Captures relationship between words(applicable for variations of component like Strain Reliefs vs Strain Relief Backshell)
* Can handle out-of-vocabulary words
* May not fully capture document-level context

**Sentence Embedding**
* Captures context and meaning at the sentence level and document-level context
* Not a particularly important need to understand overall content and context of datasheet

**Word Count Vectorizer**
* Captures word frequency information & can handle large vocabularies
* Ignores semantic meaning and context, hence not applicable

**TF-IDF (Term Frequency-Inverse Document Frequency)**
* Considers word frequency while adjusting for word importance. Helps to identify words that are unique to specific documents.
* May not fully capture document-level context and semantic meaning



## Step 4: Build and train your classification model

Now, build and train a model that classifies the IP rating from the document's text. Start with any additional pre-processing steps you may need, then vectorize the documents with the approach that you selected above. Feel free to add tweaks and customizations to that approach if you'd like, but no need to. There will be a handful of important details of how your model is constructed to pay attention to. 

Feel free to use any libraries (e.g. TensorFlow, PyTorch, or anything else). 

Because the goal is to take as input just a single document, you won't have access to any of the part "metadata" (e.g. the Mfr Part Number, Termination Style, etc) during inference - just the text. You should be able to train your model locally, or with Google Colab, for free, but let your interviewer know if you run into problems. 

Be sure to construct a train/validate/test split and evaluate your model on the test set.

### Additional tips
a. This is a broad problem, so you can simplify the problem and reduce scope as needed. This is a hard problem - any solution will be far from perfect. 
b. Focus on this dataset, even if your model overfits to this dataset (as opposed to the other products on the Mouser website). 

## Step 5. Submit your work

When you're done, share your code with your interviewer - email, private GitHub repo, or anything else are great. 
- Please include a few sentences describing your approach, what worked well, and what was challenging.
- Please include a requirements.txt file with the libraries you used, or any special instructions for running your code.
- Report your model's performance on a held-out test set! 

### Logistics and evaluation

- Please reach out to your interviewer with any questions you might have. We'd be surprised if you had no questions - so don't hesitate to ask! 
- You are welcome to use any tools (including Copilot) and libraries you want. How well you can build on existing libraries is part of our evaluation.
- Regex is the baseline that we will compare your solution against. 

Evaluation: We're evaluating how you analyzed the data and corrected issues in it, and how you thought about tradeoffs when building your model. The performance of your model matters, but isn't the only thing we are considering. 

We will discuss the model you built in the final interview round!
### My preliminary conclusions:
* For preprocessing of data: it was relatively straightforward to standardize the string values of `IP Rating`, `Pricing`, `Contact Gender` etc with simple strip functions and conversions to float etc
* For `IP Rating`, I took some liberties in engineering it to be numerical data, losing out on the nuance in `K` ratings such as `IP69K` by treating it as `IP69` as seen in the dictionary
* Missing values: I struggled to think of ways to circumvent the missing categorical values and my straightforward but liberal approach of dropping the missing values caused the dataset to shrink by half. It would help  a lot to have a more complete set of data especially for categorical data as compared to numerical.
* I suspect that more investigation/exploration could be done to explore the relationship between variation of components and their IP Ratings or whether they had missing values that could be inferred
* I would have liked to explore the relative importance of certain features, such as comparing `Contact Gender` vs `Product`. This could be done preliminarily by research, hypothesising and SKlearn's tools to observe impurity reduction
* I would have also liked to explore and compare other models like XGBoost/CatBoost given their impressive performance and efficient handling of missing data which would be very applicable because of our missing categorical data
* Additionally, since we have categorical variables with many labels(highly cardinal), CatBoost's ability to automatically deal with cateogrical variables and not require extensive data preprocessing would be cool to compare its performance with
* As this was my first time working with word embedding feature engineering, my research online was not very helpful/I did not exactly know what research was applicable. Hence, I went with my intuition and simply tried to categorical encode `Contact Gender`, `Product` etc using word embedding. 
* I tried to research on Ingress Protection Rating and my decision to split them may not be practical
* The decision to split the IP Rating might cause more effort in training the model and having to systematically combine the outputs from different models to get the cohesive IP Rating
* Nevertheless, muy decision to use Random Forest was because of its bagging method as well as its advantages of being able to be used for both classification and regression and reduction in overfitting
* It has an accuracy score of 0.860 and 0.801 for IP_1(dust) and IP_2(water) respecitvely.
