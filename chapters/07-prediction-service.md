# 7. Prediction Service

Once a model is trained and achieves satisfying offline metrics, it must be deployed into a production environment. The Prediction Service is responsible for taking realtime inputs, transforming them, running the model inference, and returning a prediction—usually in well under 100 milliseconds.

## 1. Batch vs. Online Prediction

The first major architectural decision is choosing between batch and online inference based on your latency and compute constraints.

*   **Batch Prediction (Offline Inference):** 
    *   *How it works:* The model runs on a schedule (e.g., once every 24 hours), computes predictions for all possible users, and stores the results in a fast key-value database (like Redis).
    *   *Pros:* Extremely high throughput. When the user opens the app, the prediction is fetched from the database in < 5ms. No complex scaling required during traffic spikes.
    *   *Cons:* Stale recommendations. It cannot react to a user's real-time actions during their session.
*   **Online Prediction (Real-time Inference):**
    *   *How it works:* When a request hits the server, the model loads dynamic features and calculates the specific prediction on the fly.
    *   *Pros:* Highly responsive. If a user clicks three sci-fi movies, the very next recommendation immediately shifts to sci-fi.
    *   *Cons:* Computationally expensive, risking high latency. Requires complex microservices and auto-scaling GPU clusters.
*   **The Hybrid Approach:** Most modern architectures merge the two. (e.g., Netflix calculates a massive batch of movie candidates overnight, but uses a lightweight online model to dynamically sort them based on your clicks from the last 5 minutes).

## 2. Nearest Neighbor Service (Approximate NN)

In search and recommendation systems, finding the most relevant item is a mathematically daunting constraint. If you have 100 million videos, calculating the exact mathematical similarity between a User's vector and every single Video vector in real-time is impossible.

We solve this using **Approximate Nearest Neighbor (ANN)** algorithms:
*   Instead of doing an exact search ($O(N)$), ANN algorithms (like FAISS, ScaNN, or HNSW) partition the vector space using trees, hashing, or graph structures.
*   It allows the system to find the "closest" items in milliseconds at the cost of a tiny reduction in absolute accuracy.

## 3. ML on the Edge (On-Device AI)

Not all models run on massive server clusters. Many applications mandate that the model runs directly on the user's mobile device (e.g., FaceID, voice assistants, or smart camera filters). 

*   **Why run on Edge?** 
    *   *Zero Network Latency:* The app works without internet access.
    *   *Privacy:* Sensitive data (like biometric scans) never leaves the user's phone.
    *   *Compute Costs:* Server costs are offloaded directly to the consumer's hardware.
*   **The Challenge:** Mobile phones have strict constraints regarding battery life, RAM, and compute power. We cannot deploy massive Deep Learning clusters to an iPhone.

### Model Compression Techniques
To fit ML on the Edge, the models must be aggressively shrunk:
*   **Quantization:** Reducing the precision of the numbers that make up the model's weights. (e.g., converting highly precise 32-bit floating point numbers to simpler 8-bit integers). This drastically reduces the model's memory footprint with minimal accuracy loss.
*   **Pruning:** Neural networks often have redundant connections. Pruning identifies and deletes the "synapses" in the network that contribute the least to the final prediction.
*   **Knowledge Distillation:** Training a massive, complex "Teacher" model on the server, and then training a tiny, fast "Student" model to mimic the outputs of the Teacher. We then deploy the Student model to the edge device.

---
**Next Step:** The model is deployed. Now we must test if it actually improves the product. Proceed to Chapter 8 for Online Testing.
