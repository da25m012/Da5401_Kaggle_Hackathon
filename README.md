# **DA5401 Kaggle Hackathon**

**DA5401 Kaggle Challenge : Metric Learning**  
**G C V Sairam**  
**Roll No : DA25M012**  
**November 19, 2025**  

---

## **1 Introduction**

### **1.1 Problem Statement**

There is an AI evaluation system that tests a target conversational AI agent with a variety of prompts covering different business domains. The system takes the prompts and the model’s responses and scores them across several evaluation metrics.

We are given a dataset containing:

* Metric definitions
* Prompt–response pairs
* Target fitness scores assigned by an LLM judge

The task is to train a model that takes the metric definitions and prompt–response embeddings as input and predicts a fitness score between **0–10**. Essentially, the model needs to learn the semantic relationship between two pieces of text:
(1) the metric and
(2) the conversational pair.

---

### **1.2 Dataset**

The dataset contains:

* **5000** training examples
* **3638** test examples
* Two embeddings per datapoint:

  * One for the metric definition
  * One for the combined system prompt, user message, and response

The target scores ranged from 0 to 10 but were **heavily skewed toward 9 and 10**, making the dataset highly imbalanced.

---

## **2 Methodology**

### **2.1 Data Pre-processing**

To keep the inputs consistent, the system prompt, user message, and AI reply were merged into a single block of text. Both this combined text and the metric name were converted into embeddings using the **l3cube-pune/indic-sentence-bert-nli** model.

Due to the severe imbalance in score distribution, sample weights were computed based on the frequency of each score. This ensured that rare scores would not be ignored by the model.

---

### **2.2 Model Architecture**

A **dual-tower neural network** was used:

* **Tower 1:** Processes the metric embedding
* **Tower 2:** Processes the prompt–response embedding

Each tower consists of:

* A Dense layer with ReLU activation
* Dropout
* L2 regularization

After processing, the outputs from both towers are concatenated and passed through a final Dense layer with linear activation to produce the predicted fitness score.

This design allows the model to first learn independent representations before combining them.

---

### **2.3 Model Complexity**

Each tower contains nearly **400,000 parameters**. After dropout, their outputs form a combined 1024-dimensional vector.
The final prediction layer adds only a small number of additional parameters.

Overall, the model has **just under 800,000 trainable parameters**, making it relatively lightweight while still large enough to capture meaningful semantic patterns.

---

### **2.4 Training the Model**

The model was trained using:

* **Adam optimizer** with a small learning rate
* **Mean Squared Error** (MSE) as the loss function
* **RMSE** as the evaluation metric

Training setup:

* Maximum epochs: **40**
* Validation split: **10%**
* Early stopping with **patience = 5**

Training and validation errors consistently decreased, and the final validation RMSE was about **3.9**.

---

## **3 Results**

### **3.1 Training Error**

Both training and validation RMSE steadily decreased throughout training. The best validation RMSE reached **around 3.9**.

---

### **3.2 Observations on Test Predictions**

On the test set:

* Most predictions fell in the **5–6 range**
* Very few predictions were outside this range
* This behaviour differs sharply from the training distribution (mostly 9–10)

Possible reasons:

* Heavy class imbalance prevented the model from learning differences between high- and low-scoring examples
* The model defaulted to predicting mid-range values
* Because of this collapse, the output had a final RMSE of **3.68**

---

## **4 Pros and Cons of the Model**

### **Pros**

* Uses embeddings that already encode semantic information
* Small and easy to train
* Dual-input architecture is appropriate for comparing two textual sources

### **Cons**

* Predictions collapse to mid-range (5–6) instead of covering 0–10
* Model fails to distinguish extreme quality differences
* Performance limited by dataset imbalance

---

## **5 Conclusion**

A simple neural model was implemented to estimate evaluation scores using metric embeddings and combined conversation embeddings. The model trained stably and learned some useful patterns, but the predictions were mostly centered around the mid-range.

Further improvements would likely require better feature engineering, more expressive architectures, or handling the strong score imbalance more effectively.

Just tell me!
