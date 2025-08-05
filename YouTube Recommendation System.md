# 🧠 1. Problem Understanding & Goal Framing

## 1. Objective:
Design a recommendation system for YouTube that effectively handles the cold-start problem for newly uploaded videos, ensuring they reach the right users with minimal delay.

## 2. Business Goal:
1. Maximize user engagement: Increase click-throughs, watch time, and session duration.
   
2. Improve content discovery: Ensure new content is discoverable, especially from small or emerging creators.
   
3. Support creator growth: Drive early traction to newly uploaded content, keeping creators motivated and active.

## 3. Key Metrics (KPIs):

<img width="619" height="301" alt="image" src="https://github.com/user-attachments/assets/9cf6938e-8b47-4bdd-b504-2b8726fd50b8" />


## 2. Data Strategy

### Data Collection: 

1. Content metadata: title, tags, category, description, upload time

2. Channel info: creator history, number of subscribers, past video performance

3. User context: watch history, demographic profile, device, time of day

### Multimodal content:

1. Text: Title, description → BERT embeddings

2. Visuals: Thumbnail → CNN embeddings (ResNet, EfficientNet)

3. Audio/Transcript: Auto-generated captions → NLP-based content features


 ### Preprocessing:

1. Normalize timestamps and extract freshness features

2. Clean and embed textual metadata

3. Generate content embeddings (offline) for new videos

### Labeling Strategy:

1. Use past video recommendations as positive/negative pairs: >70% watch time-> Positive label, Else -> Negative Label

2. For cold-start: Bootstrap using engagement patterns of similar videos or channels


## 🤖 3. Modeling Strategy


## Phase 1: Baseline

#### Content-based filtering: 

1. Cosine similarity of video embeddings (title/tags/thumbnail) with user history
   
2. Fast, zero-dependency on user interaction data
   

## Phase 2: Hybrid System (Recommended)

### Candidate Generation:

1. Two-Tower Model:

 - > User tower: Learns embeddings from watch history, context

- > Item tower: Learns embeddings from metadata, uploader features

- > Use dot product or learned similarity function

Fast retrieval using FAISS or ScaNN

2. Ranking Layer:

- > Gradient Boosted Trees (XGBoost/LightGBM) or Deep MLP

- > Inputs: Click probability, video freshness, content quality scores, recency features

- > Output: Final score for re-ranking candidates

3. Cold-Start Specific Enhancements:

- > Uploader priors: Boost score if the uploader has strong historical engagement

- > Meta-learning: Predict early video performance from metadata using few-shot modeling

- > Exploration mechanism: Inject promising new videos into a small % of user feeds and learn quickly via early feedback loops



## 4. Evaluation Metric :

### Offline Evaluation:

<img width="631" height="255" alt="image" src="https://github.com/user-attachments/assets/61e64453-3eb9-440a-8be8-632eeec27cb7" />

### Online Evaluation (A/B Testing): 

1. CTR uplift for new video slots

2. Watch time per session when new content is shown

3. Bounce rate reduction

4. Cold-start latency improvements


## 5. ⚙️ Scalability Layer Explained

When designing a large-scale recommendation system like YouTube’s, scalability is non-negotiable because you’re dealing with:

1. Billions of videos

2. Billions of users

3. Real-time expectations (sub-100ms latency)

To handle this, the system is architected in two main stages:


### 🧱 1. Retrieval Layer (Candidate Generation) — Scale-first
Purpose: Narrow down millions of videos to a few hundred high-potential candidates in real-time.

--> Techniques Used:

1. Two-Tower Model:

      User tower: Learns user embeddings from watch history, location, time of day, etc.

      Item tower: Learns embeddings for videos from metadata (title, tags, thumbnail, etc.)

      Dot product or cosine similarity is used to rank video relevance.

2. Approximate Nearest Neighbor (ANN) Search:

Tools like FAISS, ScaNN, or Annoy are used to perform fast similarity search across millions of video embeddings.

ANN enables sub-100ms retrieval time even with a billion candidates.


--> Scalability Focus:

Embeddings for new videos are precomputed periodically (e.g., hourly or via streaming). Embeddings are stored in high-throughput vector stores. User embeddings are computed on the fly or from a cache for active sessions.


### 🧠 2. Ranking Layer — Precision-first
Purpose: Take top ~500 candidates from the retrieval step and rerank them using deeper, more computationally expensive models.

--> Techniques Used:

1. Gradient Boosted Trees (like LightGBM or XGBoost) Or lightweight deep MLP classifiers

--> Features used:

      Predicted CTR
      Video freshness
      User-video interaction likelihood
      Uploader reputation
      Early engagement signals (if any)

--> Scalability Focus:

1. Models are small enough to run within tight latency budgets (10–50ms per request).

2. Deployed as microservices with autoscaling.

3. May run in parallel across regional data centers for geo-distributed load.


## 6. Monitoring & FeedBack Loops

1. Track cold-start KPIs in real time

2. Monitor model drift (drop in CTR or watch time)

3. Alert on underperforming new video categories

--> Iteration Plan

1. Add user feedback loops (e.g., short skips, watch completion)

2. Train personalized cold-start models with meta-learning

3. Improve creator tools to optimize thumbnails, titles based on predicted CTR


## 7. ✅ Strong Summary

This system tackles the cold-start challenge through a hybrid ML architecture that leverages content-based embeddings, uploader priors, and personalized retrieval models. It ensures scalability to billions of videos and users, while aligning tightly with business goals like engagement, discoverability, and creator retention.



