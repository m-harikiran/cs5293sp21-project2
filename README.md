# Uredact Sensitive Information

#### Author: Harikiran Madishetti

---

## About

When confidential data is exchanged with the media, it must go through a redaction procedure. That is to say, all sensitive names, locations, and other sensitive data must be concealed. Documents containing confidential details, such as police reports, court documents, and medical records, are often costly and time-consuming to redact.

This project will learn how to use Sci-Kit Learn, NLTK, and other packages to build a training dataset, train an ML model, and predict redacted data. The dataset created with NLTK can train a Machine Learning model to predict the most likely unredacted words, especially names that the redactor redacted.

Predicting the titles is a difficult challenge. To predict the most reliable and best name, we must practice our ML model with a broad dataset. To create a training dataset for this project, I used the [Large Movie Review Dataset](https://ai.stanford.edu/amaas/data/sentiment/), which includes feedback on most famous movies.

## Packages Required

The below is the list of packages used in this project.

- re
- os
- regex
- argparse
- nltk
- glob
- pytest
- pipenv
- scikit-learn

Libraries such as argparse, re, os, and glob are standard libraries in python3. To install other libraries please use Command `pipenv install -r requirements.txt`

## Directions to Install and Use Pagkage

The below are the insturctions to be followed to download, install and run the package/project.

1. Create a directory and then cd into the directory
   **`mkdir Text_Project2 && cd Test_Project2`**
2. Download the project files from GitHub
   **`git clone https://github.com/Harikiran-Madishetti/cs5293sp21-project2.git`**
3. Cd into project directory **cs5293sp21-project1**
   **`cd cs5293sp21-project2`**
4. Install python package pipenv to create a virtual enviromnent
   **`pip install pipenv`**
5. After successfully installing pipenv create a python3 virtual environment
   **`pipenv install --three`**
6. Install the dependencies listed in **requirements.txt** to start using the package
   **`pipenv install -r requirements.txt`**
7. After installing the dependencies successfully run unit tests
   **`pipenv run pytest`**
8. After running the unit tests successfully start using package (relpace URL with the URL of incidents list) using below command to fetch summary of the incidents by its nature
   **`pipenv run python project2/main.py --tdata "project_docs/train/\*\***/**\*.txt" --input "project_docs/redact/review.txt"`**

## Assumptions

In this project, I use the NLTK package to define people's names and build features from them. I am guessing that the names have a limit of four words for function building. If titles contain more than four words, I am extracting the characteristics of just four words. I also assume the data to be redacted and unredacted is in text file format. If the original text is in a different format, it may produce an error. I am not taking the first and last character indexes of each name into account when we estimate the redacted names. All project-related files are stored in **project docs**, training data that can be used to train the ML model is present in **project docs/train**, and the text file is expected to be redacted, and unredacted must be placed in **project docs/redact**. The text files are censored and saved with the extension **.redacted**. The redacted files are then read, and the top four possible names for redacted names are predicted, with the output saved in the **.predicted** log.

## Description

In this project, I am reading the redacted data generated by the redactor and then estimating or predicting the top four most probable unredacted names using a KNN classifier and saving the output to a file with extension **.predicted**. The project kit includes three key files to do this:

1. **main.py**
2. **unredactor.py**
3. **redactor.py**
   In order to test the package we also have **test_unredactor.py**.

### 1. main.py

This file is called in order to process the data and return the most likely unredacted names. This file accepts the input parameters **--tdata** as file names or paths where training data files are stored, and **--input** as file names or paths where the redacted text document and predict data files are stored. The library **agrparse** handles all of these command-line arguments.

These input paramentrs are passed to the methods imported from **redactor.py**, **unredactor.py** for reading, training ML model, redacting and writing the predicted data. The below are different methods called by main function:

- Method **`redactor.redactNames(input_parameters.input)`** is used to read a text document, redact the names and write the redacted data to new file with extension **.redacted**
- Method **`unredactor.extractTrain(input_parameters.tdata)`** is used to construct the training data set for ML model
- Method **`unredactor.extractRedacted(redacted_doc)`** is used to construct the features for redacted names present in redacted files generated by **redactor**

Once the training and prediction data features are constructed the ML model is trained and used to predict the top 4 most likely unredacted names.

```python
    v = DictVectorizer(sparse=False)
    X_train = v.fit_transform([x for (x, y) in training_data])
    y_train = [y for (x, y) in training_data]
    X_redacted = v.fit_transform([x for (x, y) in redacted_data])
    y_redacted = [y for (x, y) in redacted_data]
```

The features constructed and returned by unredactor and redactor are present in a list of dictionaries, so I am using **DictVectorizer()** to fit and transform the input features.

```python
    knnModel = KNeighborsClassifier(
        n_neighbors=5, weights='uniform', algorithm='auto')  # Creating an object for KNN Classifier
    knnModel.fit(X_train, y_train)  # Fitting training data into KNN Model
    # Finding the K Nearest Neighbors for redacted names
    indx_KNN = knnModel.kneighbors(
        X_redacted, n_neighbors=4, return_distance=False)
```

I am using the K-Nearest Neighbor model to predict the top 4 neighbors of the redacted name. The model finally returns the indices of the four neighbors from the training dataset, which are close to the redacted name.

```python
    for x, y in zip(y_redacted, indx_KNN):
        doc = open(redacted_doc.replace('.redacted', '.predicted'),
                   'a')  # Opening the file to append data
        message = '\n{}. The top 4 likely names for {} :: {}, {}, {}, {}\n'.format(
            count, x, y_train[y[0]], y_train[y[1]], y_train[y[2]], y_train[y[3]])
```

The above code is used to extract the most likely names predicted by the KNN Classifier, which can replace redacted names, and then the predicted names are written to a file with extension **.predicted**

### 2. unredactor.py

This package is used by **main.py** to read redacted data and construct the features required for predicting the redacted names present in a document.

#### i. extractTrain(path)

This method takes a path or pattern as input to read the files used to retrieve names, as well as the corresponding features for training the ML model, and calls a method **getTrainFeatures(data)** to obtain training features. Finally, this method returns a list of tuples, each of which contains a person's name and features.

```python
    for file_name in glob.glob(path, recursive=True):
        data = open(file_name).read()
        training_data.extend(getTrainFeatures(data))
```

#### ii. getTrainFeatures(data)

This method takes the data read by the previous method, tokenizes it, and then uses the NLTK called object named entity chunk to define the individual names. I am extracting features such as length of name with and without spaces, number of terms, number of spaces, length of words 1, 2, three, and word four from the listed names. All extracted features are stored in a dictionary, and the result is a list of tuples containing a dictionary of features and their corresponding names.

```python
    tokenized_data = nltk.word_tokenize(data)       # Splitting data into words
    pos_tokenized_data = nltk.pos_tag(tokenized_data)
    chk_tagged_tokens = nltk.chunk.ne_chunk(pos_tokenized_data)
    for chk in chk_tagged_tokens.subtrees():
        features = {}  # To hold features of each name
        if chk.label().upper() == 'PERSON':  # Extracting the words with tag PERSON
            name = ' '.join([i[0] for i in chk])
            features['name_len_s'] = len(name)  # Length of name with spaces
            features['name_len'] = len(name.replace(' ', '')) # Length of name without spaces
            features['word_cnt'] = len(name.split(' ')) # No. of words in a name
            features['white_space'] = len(name) - len(name.replace(' ', '')) # No. of white spaces in a name
            features['w1_len'] = 0  # Length of 1st word
            features['w2_len'] = 0  # Length of 2nd word
            features['w3_len'] = 0  # Length of 3rd word
            features['w4_len'] = 0  # Length of 4th word
            words = name.split(' ')
            for i in range(len(words)):  # Finding the length of words
                if i == 0:
                    features['w1_len'] = len(words[i])
                elif i == 1:
                    features['w2_len'] = len(words[i])
                elif i == 2:
                    features['w3_len'] = len(words[i])
                elif i == 3:
                    features['w4_len'] = len(words[i])
```

#### iii. extractRedacted(path)

This method takes a path or pattern to read the redacted files whose names are unredacted or predicted using the ML model. This method calls **getRedactedFeatures(data)** to construct the features for the redacted names and finally returns a list of tuples, each containing a redacted name and features.

```python
    data = open(path).read()        # Reading the redacted data
    # Extracting the features from the redacted data
    redacted_data.extend(getRedactedFeatures(data))
```

#### iv. getRedactedFeatures(data)

This method takes the data read by the previous method, identifies the redacted names using the regex pattern.

```python
    pattern = re.compile('\u2588+\s*\u2588*\s*\u2588*\s*\u2588+')  #Regex pattern
    matched_names = re.findall(pattern, data)  # Finding all the redacted names with matching pattern
```

The matched names are then processed to extract features such as length of name with and without spaces, number of terms, number of spaces, length of words 1, 2, three, and word four from the listed names. All extracted features are stored in a dictionary, and the result is a list of tuples containing a dictionary of features and their corresponding redacted names.

```python
        name = re.sub('\s+', ' ',  names)  # Removing extra spaces present in between words
        features['name_len_s'] = len(name)  # Length of name with spaces
        features['name_len'] = len(name.replace(' ', ''))  # Length of name without spaces
        features['word_cnt'] = len(name.split(' '))  # No. of words in a name
        features['white_space'] = len(name) - len(name.replace(' ', ''))  # No. of white spaces in a name
        features['w1_len'] = 0  # Length of 1st word
        features['w2_len'] = 0  # Length of 2nd word
        features['w3_len'] = 0  # Length of 3rd word
        features['w4_len'] = 0  # Length of 4th word
        words = name.split(' ')
for i in range(len(words)):  # Finding the length of words
            if i == 0:
                features['w1_len'] = len(words[i])
            elif i == 1:
                features['w2_len'] = len(words[i])
            elif i == 2:
                features['w3_len'] = len(words[i])
            elif i == 3:
                features['w4_len'] = len(words[i])

```

### 3. redactor.py

This package is used by **main.py** to read, redact the names present and write the redacted data to a file with extension **.redacted**

#### i. redactNames(path)

This method reads a text file using a path or pattern as input, then tokenizes the data to label the person using NLTK's named entity chunk. The block letters '█' are then used to replace the person's name. After that, the redated data is written to a file with the extension **.redacted**. Finally, this procedure returns the redacted document's route and file name.

```python
    data = open(path).read()  # Reading the file to be redacted
    tokenized_data = nltk.word_tokenize(data)       # Splitting data into words
    pos_tokenized_data = nltk.pos_tag(tokenized_data) # Generationg the parts of speech of each word
    chk_tagged_tokens = nltk.chunk.ne_chunk(pos_tokenized_data) # Chunking the tagged words using named entity chunker
    for chk in chk_tagged_tokens.subtrees():
        if chk.label().upper() == 'PERSON':  # Extracting the words with tag PERSON
            for name in chk:  # Extracting first and last name
                data = re.sub('\\b{}\\b'.format(name[0]),
                              '\u2588'*len(name[0]), data)  # Replacing the names with block character
```

### 4. test_unredactor.py

The package **test_unredactor.py** has test cases defined as methods, that can be used for unit testing of methods defined in the package **redactor.py** and **unredactor.py**. In order to test each method in **redactor.py** and **unredactor.py**, first we need to import **redactor.py** and **unredactor.py**. I am using **assert** in python to verify if the test condition is true or not. If the condition returns FALSE then assert statement will fail, which inturns fails the test case.

#### i. testRedactNames()

This method is used to test method **redactNames()** in **redactor.py**. In this, I am verifying if redacted data written to the file is same as expected or not.

```python
    expected = "I couldn't image ██████ ███████ in a serious role, but his performance truly "
    file_loc = 'project_docs/package_test/test.txt'  # Name and path of the test data
    redacted_doc_loc = redactor.redactNames(file_loc)  # Redacting the data of the test file
    redacted_data = open(redacted_doc_loc).read().splitlines()  # Reading the data from the redacted document
    assert redacted_data[1] == expected  # Verifying if the redacted data and expected data is same or not
```

#### ii. testExtractTrain()

This method is used to test method **extractTrain()** in **unredactor.py**. In this, I am verifying if return data is in a list and also verifying if returned data contains list of tuples.

```python
    file_loc = 'project_docs/package_test/test.txt' # Location and name of text data file
    train_xy = unredactor.extractTrain(file_loc) # Calling the method extractTrain
    assert type(train_xy) == list # Verifying if the return type is list or not
    assert type(train_xy[0]) == tuple # Verifying if the list contains tuples or not
```

#### iii. testGetTrainFeatures()

This method is used to test method **getTrainFeatures()** in **unredactor.py**. In this, I am verifying if return data is in a list, verifying if returned data contains list of tuples and also checking if the expected output and is same as resulted output.

```python
    assert type(extracted_features) == list # Verifying if the return type is list
    assert type(extracted_features[0]) == tuple # Verifying if the returned type contains tuple
    assert type(extracted_features[0][0]) == dict # Verifying if the tuple returned has dictionary of features
    assert extracted_features == expected # Verifying it the resulted output and expected output are same or not
```
