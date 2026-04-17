# 5. Feature Engineering

Data in its raw form—like text strings, timestamps, or high-resolution images—cannot be mathematically processed by machine learning algorithms. **Feature Engineering** is the art and science of extracting the most important signals from raw data and transforming them into numerical representations.

In many real-world systems, a simple Logistic Regression model with exceptionally well-engineered features will drastically outperform a complex Deep Neural Network with poor features.

## 1. Choosing the Right Features

Features are generally categorized by the "Big Actors" in your system (e.g., in a recommender system, these are Users and Items).

*   **User Features:** Who is interacting with the system?
    *   *Static/Demographic:* Age, location, device type, language.
    *   *Dynamic/Historical:* Watch history, average time spent on platform, past search queries.
*   **Item/Document Features:** What are they interacting with?
    *   *Static:* Price, category, overall rating, text length.
    *   *Contextual/Dynamic:* Trending status, time since publication.
*   **Context Features:** What are the circumstances surrounding the interaction?
    *   Time of day, day of the week, current user location, weather.
*   **Cross Features (Interaction Features):** Combining actors.
    *   If user location = "New York" AND item category = "Winter Coat", set value to 1. 

## 2. Feature Representation Techniques

Once we know *what* features to use, we must convert them into numbers.

*   **One-Hot Encoding:** Used for categorical variables with a small, fixed number of classes (e.g., `Device = [Mobile, Tablet, Desktop] -> [1, 0, 0]`). It scales terribly for high-cardinality features like "User ID".
*   **Numerical Scaling/Normalization:** Continuous features like "Age" or "Price" shouldn't be fed directly into a model. A neural network will overly weight a price of `$50,000` compared to an age of `30`. We apply standard scaling (Z-score normalization) or Min-Max scaling to keep values between `-1 and 1`.
*   **Embeddings:** The most critical representation for modern ML. Embeddings convert complex, high-cardinality entities (words, images, or even entire user profiles) into dense, low-dimensional mathematical vectors. 
    *   *Example:* Word2Vec creates text embeddings where words with similar semantic meanings are grouped closer together in vector space.

## 3. Preprocessing Unstructured Data

Unstructured data requires significant engineering before it reaches the model.

*   **Text Processing:** Lowercasing, removing stop words, tokenization (breaking text into sub-words or characters), and applying models like TF-IDF or BERT to generate dense embeddings.
*   **Image/Video Processing:** Resizing to a uniform dimension (e.g., 224x224), normalizing pixel RGB values from `0-255` down to `0-1`, and extracting specific frames from video streams.

## 4. Static vs. Dynamic Features

In a production system, features exist in two distinct pipelines:
1.  **Static Features (Batch Computed):** Features that rarely change (like total lifetime purchases) are pre-computed overnight by a batch pipeline (like Spark) and stored in a Feature Store.
2.  **Dynamic Features (Online Computed):** Features that change by the millisecond (e.g., items viewed in the current session). These must be calculated on-the-fly by the prediction service at inference time.

---
**Next Step:** With clean, highly representative features, we are finally ready to train the algorithm. Proceed to Chapter 6 to learn about Model Development.
