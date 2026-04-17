# 3. Architectural Components (MVP Logic)

After framing the ML problem and defining success metrics, it is time to map out the high-level system architecture. In a production environment, the Machine Learning logic is often the smallest part of the total system. A robust design clearly separates ML components from Non-ML components.

## 1. High-Level Architecture Overview

A typical real-world system consists of user-facing components, backend servers, databases, and the ML prediction logic working in tandem. 

### Non-ML Components
* **Client / UI:** The user interface (Mobile app, Web app) making requests.
* **API Gateway / Load Balancer:** Routes incoming requests, handles rate-limiting, and distributes traffic across available servers.
* **App Server (Business Logic):** Validates requests, aggregates required data from various databases, and manages the non-ML business rules.
* **Databases:**
  * **Relational DB / NoSQL DB:** Stores user profiles and application metadata.
  * **Feature Store:** A centralized data system crucial for ML. It serves pre-computed, low-latency features (like user history) directly to the prediction service.

### ML Components
The ML prediction pipeline is usually abstracted into an isolated microservice.
* **Candidate Generator:** (Used heavily in Search/Recommendation architectures) Quickly narrows down millions of possible items into a few hundred relevant candidates using lightweight models or heuristics.
* **Ranker / Scorer:** Takes the hundreds of candidates from the generator, enriches them with heavy features, and applies a deep learning model to score and order them precisely.
* **Filter:** Applies final business constraints (e.g., removing out-of-stock items, suppressing NSFW content).
* **Training Pipeline:** A separate, offline batch process that recalculates model weights and stores them in a model registry.

## 2. Modular Architecture Design Principle

Do not attempt to solve the entire problem in a single, monolithic model. Production models are built using **modular pipelines**. 

### Example: The Multi-Stage Recommender
If you are designing YouTube's homepage recommendation system, feeding all videos through a deep neural network would be computationally impossible. 

Instead, the architecture is broken down as follows:
1. **Stage 1 (Retrieval):** Simple, fast models (like Matrix Factorization or Two-Tower networks) retrieve 500 candidate videos out of 100 million based on user history.
2. **Stage 2 (Ranking):** A heavy, compute-intensive Deep Neural Network (DNN) scores and ranks those 500 candidates.
3. **Stage 3 (Re-ranking/Calibration):** A swift heuristic layer ensures diversity (e.g., not showing 5 videos from the same creator in a row) before sending the final top 10 items to the App Server.

## 3. The Lifecycle of an Inference Request

Consider what happens when a user opens an app:
1. The App Server receives the request and extracts the User ID.
2. It makes a gRPC call to the **Prediction Service**.
3. The Prediction Service queries the **Feature Store** to fetch the user's historical context and recent behavior.
4. The requested features and item metadata are passed into the scoring engine containing the loaded model binaries.
5. The Prediction Service returns probability scores to the App Server.
6. The App Server logs the interaction to a message broker (like Apache Kafka) for future training, and returns the final UI to the client.

---
**Next Step:** To power these architectural components, the system requires a steady stream of high-quality data. Proceed to Chapter 4 to learn about Data Collection and Preparation.
