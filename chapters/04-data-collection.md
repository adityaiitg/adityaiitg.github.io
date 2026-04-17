# 4. Data Collection and Preparation

Machine Learning models reflect the data they are trained on. Without a robust data ingestion and preparation strategy, even the most sophisticated Neural Network will fail. This chapter covers how to source, label, and process data for scaling ML systems.

## 1. Understanding Data Needs

Before building data pipelines, map out what exact signals the model requires:
* **Target Variable:** What are you trying to predict? 
* **Big Actors:** Who are the core entities interacting? Usually, this is the **User** and the **Item/Document**.
* **Data Volume & Velocity:** Are we processing petabytes of images, or thousands of structured rows? How fast does the model need to react to new data?

## 2. Data Sources and Ingestion

Data is categorised by its origin and accessibility.

* **Implicit Feedback (Logging):** Behavior observed passively. This is cheap, abundant, but incredibly noisy. (e.g., clicks, scroll depth, dwell time, video completion rate).
* **Explicit Feedback:** Direct user input. Rare but highly accurate. (e.g., a 5-star rating, filling out a survey, hitting "Not Interested").

The pipeline usually involves sending clickstream data to a data broker (like Kafka or AWS Kinesis), executing real-time stream processing (via Flink or Spark Streaming), and dumping it into a Data Warehouse (Snowflake, BigQuery).

## 3. The Labeling Bottleneck (For Supervised ML)

For supervised learning models, obtaining ground-truth labels is the most expensive and time-consuming stage. 

### Labeling Strategies
* **Natural Labels:** The best case scenario. The system natively observes the outcome. For CTR prediction, if the user clicked, the label is 1; if they ignored it, the label is 0. 
* **Human Annotation:** When natural labels do not exist (e.g., identifying toxic chat messages or drawing bounding boxes around cars). Pro: High accuracy. Con: Extremely slow, costly, and restricted by strict data privacy guidelines.
* **Handling Lack of Labels (Programmatic Labeling):**
  * *Semi-Supervised Learning:* Train on a small set of human labels, use the model to predict pseudo-labels on the massive unlabelled dataset, and retrain.
  * *Weak Supervision:* Use noisy heuristics to generate bulk labels automatically (e.g., using regex keyword spotting to flag toxic comments).
  * *Transfer Learning:* Leverage massive foundation models pre-trained on internet-scale data. Fine-tune them on your smaller downstream task using few-shot learning.

### The Negative Sampling Problem
What constitutes a "negative" label? If a user scrolled past an ad without clicking, did they hate it? Or did they just not see it? Simply sampling all unclicked items as negatives creates massively imbalanced datasets. Strategies like **Hard Negative Mining** or random sampling are required to balance classes dynamically.

## 4. The Data Generation Pipeline Workflow

A standard ML data generation pipeline generally follows this chronological flow:
1. **Ingestion:** Raw streams from the application arrive at the lakebed.
2. **Feature Generation:** Raw text is tokenized, unstructured logs are converted to usable states.
3. **Feature Transform:** Categorical variables are mapped; normalization is applied.
4. **Label Generation:** The "Joiner" component maps the user's historical feature state precisely to the future action (the label) they took at that exact timestamp. *Time-travel leakage must be strictly avoided here.*
5. **Data Augmentation:** Common in Computer Vision and Audio. Rotating, blurring, or adding noise to samples to artificially increase dataset resilience.

---
**Next Step:** Raw data is rarely fed straight into algorithms. Proceed to Chapter 5 to learn how to mold this data into dense mathematical vectors through Feature Engineering.
