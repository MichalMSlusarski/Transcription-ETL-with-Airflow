# YouTube transcription topic modeling for Polish language (Airflow ETL)

Apache Airflow-based data processing workflow designed for daily ETL operations on text data.

The project consists of a Directed Acyclic Graph (DAG) that orchestrates the execution of four main tasks: 
- monitoring selected channel for uploads,
- downloading transcription of new content,
- processing transcription,
- loading extracted data into a BigQuery database.

The text processing is desinged to extract the following information:
- topics 
- sentiment
- readability

## DAG Overview

The DAG ensures that the tasks are executed in the correct order. This guarantees that the data is fetched, transformed, and loaded sequentially.

The process consists of the following tasks:

![DAG](https://github.com/MichalMSlusarski/Transcription-LDA-with-Airflow/blob/main/DAG.png)

In the event that no new video was uploaded during the scheduled time, the first task returns None, leading to the termination of the entire process. This avoids unnecessary computation. An appropriate email notification is sent using the send_email_func function.

The project allows for customization, such as modifying the data retrieval, transformation, and loading functions to adapt to specific data sources and destination systems.

### Data extraction

The whole process of data extraction is built around the YouTube Data API v3. Every day the process begins by checking the specified channel for new updates. The code fetches the latest video ID from the channel, making sure that it is not already in the database reference list before adding it. If all is good to go, we ask for the following information: the video ID, (and based on that...) the title, and the transcript.

### Topic modeling

Upon receiving the downloaded transcription, the processing module begins rudimentary data cleanup. As we are working with a quite cumbersome language - Polish, a thorough lemmatization must be performed. For any modern text sourced from the internet, a pre-trained spaCy model -```pl_core_news_sm``` seems to be doing remarkably well.

The doc2bow method converts the list of tokens into a bag-of-words representation, which is a list of tuples where each tuple represents a word's ID and its frequency in the document. The resulting corpus is a list of such bag-of-words representations.

The LdaModel function trains an LDA model on the corpus. It takes the corpus, the number of topics to be extracted, the dictionary mapping, and the number of passes as input parameters. The model learns the underlying topic distribution based on the given corpus.

### Sentiment analysis

For sentiment analysis I used one of rare pre-trained sentiment analysis models, specifically designed for the Polish language. The appropriate function splits the text into individual sentences, applies the sentiment analysis model to each sentence, and collects the sentiment scores. It then calculates the average sentiment score by summing the scores of all sentences and dividing it by the total number of sentences. The resulting sentiment score is rounded to five decimal places and returned by the function.

### Readability index

The Gunning-Fog formula is used to calculate readability index. It's one of the very few methods easily and reliably applicable to the Polish language. The index provides a numerical value that represents the number of years of formal education required to understand the text. A higher index value indicates a more complex and difficult text, while a lower index value suggests a simpler and easier-to-understand text. I used the implementation from the ```textstat``` library.

## Credits

Apart from the large libraries, this project benefits from a very small package called SentimentPL: https://github.com/philvec/sentimentPL (~10★)
