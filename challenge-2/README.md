# Challenge 2: Address Elements Extraction
See the competition on [Kaggle](https://www.kaggle.com/c/scl-2021-ds/overview).

## Background
At Shopee, we strive to ensure our customers' highest satisfaction for their shopping and delivery experience - fast and accurate delivery of goods. This can be better achieved if we have key address elements for each user address which allows us to accurately geocode it to obtain geographic coordinates to ship the parcel to our customers. These key address elements include Point of Interest (POI) Names and Street Names. However, most addresses that Shopee receives are unstructured and in free text format, not following a certain pattern. Thus it is important for us to develop a model to precisely extract the key address elements from it.

### Task
In this competition, you’ll work on addresses collected by us to build a model to correctly extract Point of Interest (POI) Names and Street Names from unformatted Indonesia addresses. Participants are expected to build their own model for this competition, submissions by teams which directly call any third party APIs on the test set will not be taken into consideration.

The evaluation metric is **accuracy**. Submissions are scored:

- 1 if the POI and street name **both** match
- 0 otherwise

## Solution Concept
We identified this as a Named Entity Recognition (NER) problem. This seemed a daunting task for us because none of us had worked on an NER problem before. After some research, we chose to use [spaCy](https://spacy.io/) to develop an NER model.

### Preprocessing

#### Shortened Tokens
We noted that several tokens in the training examples were shortened forms of the actual POI or street name in the labels. To address this issue, we analysed the frequencies of `(shortened token, full token)` pairs. From our analysis, we figured that this was probably a deliberate edit by the organiser to introduce some noise into the data. For example, some unnatural shortened tokens were:

- `cate` vs. `catering`
- `fina` vs. `finance`
- `restau` vs. `restaurant`

Since NER is extractive and would not otherwise associate a shortened token with a full token, we replaced all shortened tokens by the most common full token corresponding to each shortened token. In some cases, it wasn't clear which full token should be used. However, in many cases, the preprocessing proved to be helpful:

- `neg` to `negeri`
- `mas` to `masjid`
- `kan` to `kantor`

#### spaCy Data Format
spaCy required that data for each training example be formatted as a tuple:

```
(
    ADDRESS_TEXT,
    {
        'entities': [
            (entity1_start_index, entity1_end_index, ENTITY_NAME),
            (entity2_start_index, entity2_end_index, ENTITY_NAME),
            ...
        ]
    }
)
```

Using pandas and some regex, we re-formatted the processed data into the required format.

### Training
We used the [spaCy 2.0](https://v2.spacy.io/) API (as opposed to the spaCy 3.0 one) because it had better documentation, and there was more community support for it at the time. We trained separate models to extract POIs and street names: each model identified **only** POIs or street names - no other entities were extracted.

Our methodology was best described as Andrew Ng's [panda approach](https://www.youtube.com/watch?v=wKkcBPp3F1Y): we adjusted pipeline hyperparameters like the preprocessing configuration, learning rate, dropout, and batch size to iteratively improve the two models. The full training logs are provided below.

#### Training Logs
Overall, we trained 13 POI models and 13 street name models. The submissions (Sub) in bold were those that achieved higher scores on the public leaderboard on [Kaggle](https://www.kaggle.com/c/scl-2021-ds/leaderboard).

| Sub    | POI    | STREET    | Score   | Changes and Hypotheses/Conclusions | Reference |
| :----: | :----: | :-------: | :-----: | :--------------------------------- | :-------- |
| 2      | POI-1  | -         | 0.13746 | -                                  | -         |
| 3      | POI-1  | STREET-1  | 0.58093 | -                                  | -         |
| **4**  | POI-2  | STREET-2  | 0.59873 | Baseline |                         | -         |
| **5**  | POI-3  | STREET-2  | 0.60186 | (POI Dropout 0.5 --> 0.3) OR (POI No ET --> ET-1) = +Score | Unfair test |
| 6      | POI-4  | STREET-2  | 0.59053 | POI Dropout 0.3 --> 0.1 = -Score | [5] POI-3 / STREET-2 |
| **9**  | POI-2  | STREET-A  | 0.61000 | STREET-2 --> STREET-A = +Score | [4] POI-2 / STREET-2 |
| 10     | POI-5  | STREET-2  | 0.60133 | POI Dropout 0.3 --> 0.4 ~ Equal Score  | [5] POI-3 / STREET-2 |
| **11** | POI-6  | STREET-A  | 0.63866 | POI No ET --> ET-2 = +Score | [9] POI-2 / STREET-A |
| 12     | POI-6  | STREET-5  | 0.59846 | STREET-A --> STREET-5 = -Score | [11] POI-6 / STREET-A |
| 13     | POI-6  | STREET-B  | 0.63420 | STREET Dropout 0.3 --> 0.5 = -Score | [11] POI-6 / STREET-A |
| **14** | POI-8  | STREET-A  | 0.64206 | POI batch size 2000-2500 --> 500-2000 = +Score | [11] POI-6 / STREET-A |
| 15     | POI-7  | STREET-A  | 0.63813 | POI batch size 500-2000 --> 100-2000 = -Score | [14] POI-8 / STREET-A |
| **16** | POI-8  | STREET-C  | 0.64253 | STREET No ET --> ET-1 = +Score | [14] POI-8 / STREET-A |
| **17** | POI-9  | STREET-C  | 0.64393 | POI ET-2 --> ET-3 = +Score | [16] POI-8 / STREET-C |
| **18** | POI-9  | STREET-D  | 0.64426 | STREET batch size 2000-2500 --> 500-2000 = +Score | [17] POI-9 / STREET-C |
| 19     | POI-9  | STREET-E  | 0.59686 | STREET ET-1 --> ET-1 with no empty labels = -Score | [18] POI-9 / STREET-D |
| 20     | POI-10 | STREET-D  | 0.64353 | POI batch size 500-2000 --> 1000-2000 = -Score | [18] POI-9 / STREET-D |
| 21     | POI-9  | STREET-DR | 0.60426 | Randomly blank out street = -Score |
| 22     | POI-9  | STREET-F  | 0.64166 | Use all 300,000 samples (ET-3) = increase blank prediction rate = -Score | [18] POI-9 / STREET-D |
| 23     | POI-11 | STREET-D  | 0.63773 | POI ET-3 --> ET-4 = +Score | [18] POI-9 / STREET-D |
| 24     | POI-9  | STREET-G  | 0.63440 | STREET ET-1 --> ET-4 = +Score | [18] POI-9 / STREET-D |
| 25     | POI-9O | STREET-DO | 0.63720 | Under-trained models w/ validation |  |
| 25B    | POI-9  | STREET-DU | 0.64386 | Undertrained STREET-D |  |
| 26     | POI-9F | STREET-DF | 0.64500 | Models w/ validation |  |
| 27     | POI-9  | STREET-DF | 0.64500 | Only STREET-D "optimised" |  |
| 28     | POI-9F | STREET-D  | 0.64500 | Only POI-9 "optimised" |  |
| 29     | POI-11 | STREET-G  |  | ET-4 for both |  |
 
## Result
We finished in **27th place out of over 1,000 teams (top 3%)**. Our private leaderboard score was an accuracy of 65.4%.

## Reflections
Overall, the competition gave me valuable exposure to NER. The tight competition timeline forced me to learn quickly - both the basics of NER and how to implement these in code in spaCy. It was also a great opportunity to practice my skills in building neural networks, less network architecture search (since spaCy didn't allow it).

That said, there were some areas for improvement:

### Proper Validation
Given the long training time, we skipped cross validation entirely. But, we noticed that some teams (who posted their solutions) actually did! Perhaps, we could have improved model performance by using cross validation, possibly on a smaller subset of the data. We also did not keep a holdout set to test our models on. This could have been useful to provide us with a more accurate picture of how our model would perform on the leaderboard.

### Pipelines
My "pipeline" to preprocess data and train models was extremely manual. I kept different notebooks for different versions of the preprocessing steps and models. This meant that I had to duplicate and amend two notebooks just to preprocess the data and train a new model. Multiply this by two (POI and street name), and that's **four** manual steps. This process could have been improved by using better tools for managing dependent tasks, like Airflow (which I have since picked up).

At the time, I wasn't familiar with [spaCy 3.0](https://spacy.io/). However, Zach was discovered that it was pretty efficient because it used config files and simple CLI commands to control training. Check out his [article on Towards Data Science](https://towardsdatascience.com/using-spacy-3-0-to-build-a-custom-ner-model-c9256bea098). I could have explored this option a little more to improve efficiency in hyperparameter tuning.

### Managing Experiments
We logged our experiments manually using the table above. We could have improved this process by using a ML lifecycle management tool like [MLflow](https://mlflow.org/). I have shortlisted this as a tool to learn in the coming months.