# 6. Model Development and Offline Evaluation

With a robust pipeline generating high-quality features, we can now select, train, and evaluate our ML models. The rule of thumb in model development is **KISS (Keep It Simple, Stupid)**. Always start with the simplest possible approach and incrementally increase complexity.

## 1. Model Selection (The MVP Approach)

Do not jump straight to Transformers. Follow a logical progression constraint-checked against your offline metrics.

1.  **Heuristics / Rule-Based System:** Use this as your baseline. (e.g., Recommend the most popular items ordered by recency).
2.  **Simple ML Models:** Logistic Regression, Naive Bayes.
    *   *Pros:* Highly interpretable, extremely fast to train and serve, cheap.
    *   *Cons:* Cannot capture complex, non-linear relationships.
3.  **Tree-Based Models:** Gradient Boosted Decision Trees (GBDT), XGBoost, Random Forest.
    *   *Pros:* Dominates on tabular/structured data. Handles missing data well without massive preprocessing.
    *   *Cons:* Harder to update incrementally; usually requires full retraining.
4.  **Deep Learning (Neural Networks):** FeedForward networks, CNNs (for vision), Transformers (for text).
    *   *Pros:* State-of-the-art performance for unstructured data (audio, text, image). Unlocks complex embedding architectures like the Two-Tower model.
    *   *Cons:* Computationally expensive, requires massive datasets to avoid overfitting, acts as a "black box" lacking interpretability.

## 2. Dataset Splitting & Preparation

How you split your data dictates whether your offline metrics translate to online success.

*   **Train / Dev / Test Splits:** Usually split roughly `80% / 10% / 10%`.
*   **Time-based Splitting:** Randomly shuffling records across time is a massive mistake in forecasting or recommendation systems. You will cause **Data Leakage** by training the model on the "future" to predict the "past." Always split strictly by time (e.g., Train on Jan-Oct, Test on Nov-Dec).
*   **Class Imbalance:** If 99% of transactions are legitimate and 1% are fraudulent, a model that simply outputs "Legitimate" will be 99% accurate but entirely useless.
    *   *Solutions:* Oversampling the minority class, undersampling the majority class, or applying a Weighted Loss Function that heavily penalizes the model for missing the 1% fraud cases.

## 3. Model Training Fundamentals

*   **Loss Functions:** The mathematical equation the model tries to minimize. 
    *   *Binary Cross-Entropy (BCE):* Used for binary classification (Spam/Not Spam).
    *   *Mean Squared Error (MSE):* Used for regression tasks.
*   **Optimizers:** The mechanism that updates the model's weights based on the loss function. (e.g., Stochastic Gradient Descent (SGD), Adam, RMSProp).
*   **Training Strategy:** 
    *   *From Scratch:* Randomly initializing weights. Requires huge datasets.
    *   *Fine-tuning:* Taking a pre-trained foundation model and updating only the last few layers on your specific, smaller dataset.

## 4. Hyperparameter Tuning & Offline Evaluation

Once the model is training stably, it is time to squeeze out maximum performance. Hyperparameters (like learning rate, batch size, or tree depth) govern the learning process itself.
*   **Grid Search / Random Search:** Exploring different combinations of hyperparameters to find the optimal setup.
*   **Offline Evaluation:** Run the model strictly against the unseen Test Set. If the offline metrics (nDCG, F1-Score) beat the MVP baseline, the model is a candidate for production.

---
**Next Step:** A trained model stored on a disk provides zero business value. We must deploy it. Proceed to Chapter 7 on serving predictions.
