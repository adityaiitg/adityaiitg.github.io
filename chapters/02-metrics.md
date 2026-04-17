# 2. Metrics (Offline and Online)

Once we understand the business logic, we must define how to evaluate the success of the ML system. A well-designed ML pipeline relies on two layers of metrics: **Offline Evaluation Metrics** (used during training) and **Online Evaluation Metrics** (used in production).

## 1. Offline Evaluation Metrics

Offline metrics are computed on a static, historical dataset (the validation/test set). These are mathematically sound metrics used by engineers to tune model weights before they ever see real users.

### Classification Tasks
* **Precision:** Out of all positive predictions made, how many were actually correct? Useful when the cost of a false positive is high (e.g., spam detection).
* **Recall:** Out of all actual positive cases, how many did the model find? Useful when the cost of a false negative is high (e.g., cancer diagnosis).
* **F1-Score:** The harmonic mean of Precision and Recall. Great for imbalanced datasets.
* **ROC-AUC & PR-AUC:** Measures the model's ability to distinguish between classes across different thresholds. PR-AUC is preferred when dealing with highly imbalanced data.

### Ranking and Retrieval Tasks
* **Precision@k & Recall@k:** Measures relevance in the top *k* results, but does not care about the order of items within those top *k*.
* **Mean Average Precision (mAP):** Considers the ranking quality of retrieved results.
* **Normalized Discounted Cumulative Gain (nDCG):** The gold standard for ranking. It assigns higher scores when highly relevant items appear at the very top of the list.

### Regression Tasks
* **Mean Squared Error (MSE) / Root Mean Squared Error (RMSE):** Heavily penalizes large errors.
* **Mean Absolute Error (MAE):** More robust to outliers.

## 2. Online Evaluation Metrics

Once the model passes offline tests, it is deployed to interact with real users via shadowing, A/B testing, or canary releases. Online metrics measure true business impact. An ML model can have an amazing F1-score offline but still hurt user engagement online due to unforeseen dynamics.

### User Engagement & Conversion
* **Click-Through Rate (CTR):** The ratio of users who click on a recommended item to the number of total users who viewed it.
* **Conversion Rate (CVR):** The percentage of users who take a desired action (e.g., purchasing a product) after clicking.
* **Task Success/Failure Rate:** Did the ML recommendation actually solve the user's implicit goal?
* **Session/Watch Time:** Total duration a user spends engaging with content (crucial for platforms like YouTube or Netflix).

### System Performance & Counter Metrics
* **Latency (p50, p90, p99):** System response time. Real-time inference usually requires p99 latency under 100-200ms.
* **Negative Feedback Rate:** How often users hide, report, or leave negative feedback on recommendations.

## 3. Trade-offs Between Metrics

System design is fundamentally about trade-offs. You rarely get to maximize all metrics simultaneously. 
* **Precision vs. Recall:** Increasing strictness to boost precision will inherently lower recall.
* **Latency vs. Accuracy:** A deeper, more complex neural network yields higher offline accuracy but increases inference latency, which may harm online engagement due to slow load times.
* **Short-term vs. Long-term:** Clickbait recommendations might drastically boost short-term CTR (online metric) but cause a drop in long-term Daily Active Users (DAU) as user trust degrades over time.

---
**Next Step:** With our problem defined and metrics established, we must map out the high-level architecture of the system. Proceed to Chapter 3.
