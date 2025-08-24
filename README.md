# Project 2 of course "Generative AI" from Udacity

Project: Build Your Own Custom Chatbot

Completion model: gpt-3.5-turbo-instruct
Embedding model: text-embedding-ada-002
Data source: Wikipedia artice "2024 in science"


## Chosen dataset for context generation

The project should add a context with data to the prompt to allow the model to answer questions which were not known training time of the model.

First data about AI topics from 2025 from Wikipedia were tried but there was not enough data in the articles to get the needed minimum of 20 rows for the dataset.
A first try with an article about hedgehogs showed that, how it was assumed, that the answers are very similar with and without the data from Wikipedia. Unfortunately the information about the hedgehogs diet is wrong in the english Wikipedia and in the model without added context.

Finally, to have a good dataset that can add additional information to the model using the prompt, the article "2024 in science" from Wikipedia was chosen: https://en.wikipedia.org/wiki/2024_in_science

It contains data from 2024 which was not available when the model was trained. It also much more then 20 different entries. The format also gave an opportunity to practice some more date preprocessing than other Wikipedia articles.

## Data Wrangling

The data was downloaded using the Wikipedia API.

### Preprocessing

For preprocessing the following steps were done:

- remove empty lines
- Remove all rows at the end after the heading "See also"
- remove the first line which just explains this data is from 2024
- remove headings (they start with "==")
- add dates to every row with data and remove rows that only have a date
- add the year to all rows

Rows off the data looked then like this:

```
Index[0]:  0
Text[0]:   2024 2 January – The Japan Meteorological Agency (JMA) publishes its JRA-55 dataset, confirming 2023 as the warmest year on record globally, at 1.43 °C (2.57 °F) above the 1850–1900 baseline. This is 0.14 °C (0.25 °F) above the previous record set in 2016.
```

The text starts with the data and after a "-" the event is described.

### Creating the embeddings

For every row in the dataset the embedding was calculated and added to the dataframe.

## Custom Query Completion

The following steps were done:

- load the processed data with embeddings from file
- create embedding from question
- calculate distances to the embeddings of the data
- use the best fitting data rows as context for the prompt
- let the model process the prompt and return the answer

Since the dataset is small and the first rows are expected to contain the relevant data (each row is self contained), a fixed number of the best matching rows was chosen.

## Evaluation with two questions

If the chosen data was appropriate for the task, the model was evaluated with two questions, each with and without added context.

### Without context:

```
Question: Who reported the discovery of TOI-715 b? When was it announced?
Answer: Mayor et al. made the discovery in 2021 and it was publicly announced on July 29, 2021.
```

```
Question: What is the luminous astronomical object ever discovered? When was it discovered?
Answer: The most luminous astronomical object discovered so far is a quasar known as J1342+0928, discovered in 2017. However, it is possible that there are even more luminous objects yet to be discovered in the universe.
```

### With context:

```
Question: Who reported the discovery of TOI-715 b? When was it announced?
Answer: NASA and it was announced on January 31, 2024.
```
```
Question: What is the luminous astronomical object ever discovered? When was it discovered?
Answer: quasar QSO J0529-4351, February 19, 2024
```

### Analysis

For both questions the question with the added context from the Wikipedia data gave better answers. It choosed the correct data from the Wikipedia article.
Without context, the model gave different answers, from "don't know" to hallucinated information.

So the added data clearly improved the answer quality for scientific events in 2024.

## Chatbot

A answer-question loop was tried where the user can ask questions and the model answers. It showed that the model with context can answer the questions with information from the Wikipedia artice.

It was also tried with different prompts without context. Depending on the prompt starts (like a system prompt) the answers were different.
Without telling the model who it is answers the question who he is with different names like "Agent Assistant Martury" or "Bot Quad".
Sometimes it also returned python code snippets that look like it wants to create an algorithm that tries to calculate the answer.

Also once it returned after the answer more text, a question and answer as code that looked like trainings date:

```
Question: What happened in 2020?
Answer: I don't know
    */
    static question1() {
        return loadJson("", "q1");
    }

      /*
        Exercise 2 
        
        Given a string str, write a method that will 
        return the string made up of its last char and 
        then the first char and so on.

        ---

        Parameters: str
        Returns: str
    */
    static flipTheString(str) {
        var strFlip = '';
        var temp = ''
        for (var i = (str.length - 1); i>=0; i=i-2) {
            temp = str[i];
            if((i-1)>= 0) {
                temp = temp + str[i-1];
            }
    ```

This happened with the prompt like it was shown in the course, with "---" as delimiter in the prompt. This seems to trigger similar training data that was used for the model.