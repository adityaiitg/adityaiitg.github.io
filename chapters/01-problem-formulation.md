# 1. Problem Formulation

Before diving into complex models and architectures, the most critical step in designing any Machine Learning system is **Problem Formulation**. A perfectly engineered system that solves the wrong problem is useless.

This step involves translating a vague business objective into a well-defined ML problem.

## 1. Clarifying Questions

When presented with a problem (e.g., "Build a recommendation system for our platform"), never assume you know exactly what the stakeholder wants. Ask clarifying questions to narrow the scope:
* **What is the primary business objective?** (e.g., increase user engagement, boost revenue, reduce churn)
* **Who is the end user?** 
* **What does the current system look like?**
* **Are there any hard constraints?** (e.g., must run on mobile device, must return predictions in < 50ms)

## 2. Defining Use Cases and Business Goals

Establish exactly what success looks like from a product perspective.
* **Scope:** Are we targeting all users or a specific demographic initially? What features are absolutely necessary for the MVP (Minimum Viable Product)?
* **Scale:** How many active users do we have? DAA (Daily Active Users) vs MAA (Monthly Active Users).
* **Personalization:** Does the model need to be highly personalized per user, or is a generic model sufficient?

## 3. Establish System Requirements & Constraints

* **Performance Requirements:**
  * **Latency:** Does the prediction need to happen in real-time (e.g., fraud detection during checkout < 100ms) or can it be batch processed overnight (e.g., generating weekly "Discover Weekly" playlists)?
  * **Throughput:** How many predictions per second (QPS - Queries Per Second) must the system handle during peak traffic?
* **Data Constraints:**
  * **Availability:** Do we currently have the data required to train this model? 
  * **Privacy/Compliance:** Are there GDPR, CCPA, or HIPAA regulations we need to adhere to? Can we store user data indefinitely?

## 4. Translating Abstract to ML Problem

Once the business context is clear, translate the objective into mathematical ML terms:
* **ML Objective:** What are we optimizing for? (e.g., maximize click-through rate, minimize root mean square error)
* **ML Input (Features):** What signals will the system use? (e.g., user search history, item metadata, time of day)
* **ML Output (Labels):** What is the exact output of the model? (e.g., a probability score between 0 and 1, a class label, a bounding box)
* **ML Category:** 
  * *Binary Classification* (e.g., Spam vs Not Spam)
  * *Multi-class Classification* (e.g., Categorizing a news article into Sports, Tech, Politics)
  * *Regression* (e.g., Predicting the exact price of a house)
  * *Ranking* (e.g., Ordering search results by relevance)
  * *Unsupervised Learning* (e.g., Grouping similar users into clusters)

## 5. Do we actually need ML?

This is often the ultimate trick question. Machine Learning introduces complexity, technical debt, and maintenance overhead. 
* Can this problem be solved with simple heuristics or rules?
* If a simple set of IF/ELSE statements yields 80% accuracy, is the extra 10% accuracy from an ML model worth the compute, data annotation, and maintenance costs?

> **Rule of Thumb:** Always start with a heuristic or rule-based MVP. Only introduce ML when the heuristic fails to scale or meet business requirements.

---
**Next Step:** Once the problem is formulated, we need to decide exactly how we will measure success using offline and online metrics. Proceed to Chapter 2.
