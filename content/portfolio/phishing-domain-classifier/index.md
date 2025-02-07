---
title: "Phishing Domain Classifier"
date: 2024-09-26
description: "A machine learning project to detect phishing URLs"
tags: ["Machine Learning", "Cybersecurity", "Flask"]
showDate: true
showWordCount: true
draft: false
---


![alt text](thumb-hacker-phishing-1.jpg)

# **Introduction**  
Phishing attacks are one of the most common cybersecurity threats today, tricking users into providing sensitive information through malicious websites. With phishing techniques evolving, automated detection systems are crucial to stay ahead. That’s why I built a **phishing URL classifier**, a machine learning-powered web app that predicts whether a given URL is legitimate or fraudulent.

In this blog, I’ll walk through how I developed this project, from **feature extraction** to **model training** and **building the web app** using Flask.

---

# **How the App Works**  
The application allows users to input a URL. The system then performs the following steps:

1. **Extract Features**: The app analyzes various characteristics of the URL, such as length, presence of special characters, domain age, and HTTPS usage.
2. **Model Prediction**: A **Random Forest classifier** predicts if the URL is **phishing (-1) or legitimate (1)** based on these features.
3. **Display Results**: The app returns the classification result along with the probability score.

This automated process enables quick phishing detection without requiring manual intervention.

---

# **Feature Extraction: What Makes a URL Suspicious?**  

A key part of phishing detection is **feature engineering**, which means defining measurable characteristics of URLs that help determine whether they might be fraudulent. Attackers often use deceptive techniques to trick users, and by analyzing different aspects of a URL, we can detect suspicious patterns. Here are the features our app considers:  

### **Basic URL Characteristics**  
- **URL Length**: Longer URLs tend to hide malicious content.  
  - **Legitimate** (< 54 characters)  
  - **Suspicious** (54-75 characters)  
  - **Phishing** (> 75 characters)  
- **Presence of "@" Symbol**: If a URL contains an **@**, it’s often a phishing attempt to mislead users.  
- **Redirects ("//" Usage)**: If a URL contains multiple "//" outside its normal position, it may be trying to disguise its real destination.  
- **Use of Hyphens in Domain**: A domain with hyphens (e.g., "secure-bank-login.com") is often a sign of phishing.  
- **Subdomain Count**:  
  - **Legitimate** (0-1 subdomains)  
  - **Suspicious** (2 subdomains)  
  - **Phishing** (3+ subdomains)  
- **Shortened URLs**: Attackers often use services like bit.ly or tinyurl.com to mask phishing links.  

### **Domain & Security Indicators**  
- **HTTPS Usage & SSL Certificate Validity**: A missing HTTPS or an invalid SSL certificate increases phishing risk.  
- **Domain Age**: New domains (less than 6 months old) are often created for phishing before being flagged.  
- **Domain Expiry**: If a domain expires in **≤ 1 year**, it’s a red flag, as legitimate businesses usually register domains for longer periods.  
- **WHOIS Record Availability**: If WHOIS data is missing, it may indicate a fraudulent site.  

### **Website Structure & Content**  
- **Favicon Source**: If the website’s favicon (icon in browser tab) is loaded from an external domain, it might be phishing.  
- **Non-Standard Ports**: Phishing sites may use uncommon ports instead of standard ones like 80 (HTTP) or 443 (HTTPS).  
- **"HTTPS" in Domain Name**: If the domain itself contains "https" (e.g., "https-secure-login.com"), it’s likely deceptive.  
- **External Requests & Links**:  
  - A **high percentage** of external requests (images, scripts) can indicate a phishing attempt.  
  - Too many external **anchor links** (clickable links) also suggest the page is redirecting users elsewhere.  
  - If external links are placed inside `<meta>`, `<script>`, or `<link>` tags, it’s another suspicious sign.  

### **User Interaction & Behavior**  
- **Server Form Handler (SFH)**: If form data is sent to a different domain or an empty handler, the site may be stealing credentials.  
- **Submitting Info to Email**: If the page sends data via `mailto:` or uses PHP’s `mail()` function, it might be phishing.  
- **Abnormal URL Format**: A legitimate site’s domain should match the actual hostname. If it doesn’t, something’s off.  
- **Website Forwarding**:  
  - **Legitimate**: 0-1 redirects  
  - **Suspicious**: 2-3 redirects  
  - **Phishing**: 4+ redirects  
- **Status Bar Manipulation**: If the site modifies the status bar (e.g., using JavaScript `onMouseOver` tricks), it’s likely phishing.  
- **Disabling Right-Click**: Phishing sites often disable right-click to prevent users from inspecting elements or copying text.  
- **Popup Windows with Input Fields**: If a popup contains form fields, it might be trying to capture login credentials.  
- **Iframe Usage**: **Phishing sites** frequently use `<iframe>` tags to embed malicious content from another source.  

### **Reputation & Popularity**  
- **Website Traffic**: Sites ranked **below 100,000** (by Tranco) are usually legitimate, while low-traffic sites are more suspicious.  
- **PageRank Score**: If the site has a **low PageRank (< 0.2)**, it’s a potential phishing risk.  
- **Google Indexing**: If a website isn’t indexed by Google, it might be unsafe.  
- **Backlinks (External Links Pointing to Site)**:  
  - **Legitimate**: More than 2 backlinks  
  - **Suspicious**: 1-2 backlinks  
  - **Phishing**: 0 backlinks  
- **Statistical Reports-Based Analysis**: If the domain appears in phishing databases (like PhishTank), it’s almost certainly malicious.  

By extracting all these features, the app builds a dataset that helps identify phishing attempts with greater accuracy. These signals work together to detect patterns commonly found in fraudulent sites, improving our ability to flag suspicious URLs before they cause harm.  

---

# **Training the Machine Learning Model**  

Detecting phishing websites requires a strong classification model capable of identifying deceptive patterns in URLs. I chose a **Random Forest classifier**, a powerful ensemble learning algorithm that effectively handles complex data structures while offering interpretability. Below is a breakdown of the process I followed:  

---

## **1. Exploratory Data Analysis (EDA)**  

Before diving into model training, I conducted an **Exploratory Data Analysis (EDA)** to better understand the dataset and its characteristics.  

### **Dataset Overview**  
The dataset has been downloaded from [`UCI Machine Learning Repository`](https://archive.ics.uci.edu/dataset/327/phishing+websites) donated by R. Mohammad and L. McCluskey in 2015. It's not defined how they collected the data but the features are well documented. The dataset contains **11,055 URLs**, with **6,157 labeled as phishing (1) and 4,898 as legitimate (-1)**. This slight class imbalance means phishing websites are slightly more prevalent, but still close enough that it doesn’t require drastic resampling techniques.  

To get a clearer picture, I analyzed the dataset’s **features and correlations** to identify the strongest indicators of phishing behavior.  

<div style="overflow-x: auto;">
<table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>having_IP_Address</th><th>URL_Length</th><th>Shortining_Service</th><th>having_At_Symbol</th><th>double_slash_redirecting</th><th>Prefix_Suffix</th><th>having_Sub_Domain</th><th>SSLfinal_State</th><th>Domain_registeration_length</th><th>Favicon</th><th>port</th><th>HTTPS_token</th><th>Request_URL</th><th>URL_of_Anchor</th><th>Links_in_tags</th><th>SFH</th><th>Submitting_to_email</th><th>Abnormal_URL</th><th>Redirect</th><th>on_mouseover</th><th>RightClick</th><th>popUpWidnow</th><th>Iframe</th><th>age_of_domain</th><th>DNSRecord</th><th>web_traffic</th><th>Page_Rank</th><th>Google_Index</th><th>Links_pointing_to_page</th><th>Statistical_report</th><th>Result</th></tr></thead><tbody><tr><th>0</th><td>-1</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>1</td><td>1</td><td>-1</td><td>1</td><td>-1</td><td>1</td><td>-1</td><td>-1</td><td>-1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>1</td><td>1</td><td>-1</td><td>-1</td></tr><tr><th>1</th><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>0</td><td>1</td><td>-1</td><td>1</td><td>1</td><td>-1</td><td>1</td><td>0</td><td>-1</td><td>-1</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>-1</td><td>0</td><td>-1</td><td>1</td><td>1</td><td>1</td><td>-1</td></tr><tr><th>2</th><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>1</td><td>1</td><td>-1</td><td>1</td><td>0</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>1</td><td>-1</td><td>1</td><td>0</td><td>-1</td><td>-1</td></tr><tr><th>3</th><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>-1</td><td>-1</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>-1</td><td>0</td><td>0</td><td>-1</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>-1</td><td>-1</td><td>1</td><td>-1</td><td>1</td><td>-1</td><td>1</td><td>-1</td></tr><tr><th>4</th><td>1</td><td>0</td><td>-1</td><td>1</td><td>1</td><td>-1</td><td>1</td><td>1</td><td>-1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>0</td><td>0</td><td>-1</td><td>1</td><td>1</td><td>0</td><td>-1</td><td>1</td><td>-1</td><td>1</td><td>-1</td><td>-1</td><td>0</td><td>-1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr></tbody></table></div>

### **Feature Correlations**  

![alt text](image.png)

The **heatmap analysis** shows the correlation between each feature and the target variable (phishing or legitimate).  Key observations include:

*   **SSL Certificate Validity:**  A strong positive correlation (0.71) indicates that websites with a valid SSL certificate are much more likely to be legitimate. Phishing sites often lack proper certificates.
*   **Anchor Tags Linking to External Domains:** A high positive correlation (0.69) suggests that phishing sites tend to have a high percentage of outbound anchor links redirecting users to different domains.
*   **Presence of Subdomains:** A moderate positive correlation (0.30) indicates that the presence of multiple subdomains can be a sign of phishing activity.
*   **Prefix/Suffix in Domain:** A moderate positive correlation (0.35) suggests that the presence of hyphens or other prefixes/suffixes in the domain name can be indicative of phishing.
*   **Request URLs from External Sources:**  A positive correlation (0.25) suggests that a higher proportion of externally loaded resources (images, scripts) can be a red flag.
*   **Domain Registration Length:** A negative correlation (-0.23) suggests that phishing sites are more likely to have shorter domain registration periods.

This heatmap effectively visualizes the relationships between features and the likelihood of a website being phishing, highlighting the most influential factors.


---

### **Handling Imbalanced Data**  
Although the dataset isn’t severely imbalanced, phishing URLs slightly outnumber legitimate ones. To ensure the model learned effectively from both classes, I applied **class weighting** instead of oversampling or undersampling. This prevents the model from being biased toward the majority class.  

---

## **2. Model Selection & Training**  

To find the best model for phishing detection, I tested several algorithms using **LazyPredict**, an automated benchmarking tool. The top-performing models included:  

| Model                     | Accuracy | Balanced Accuracy | ROC AUC | F1 Score |  
|---------------------------|----------|-------------------|---------|----------|  
| **Extra Trees Classifier** | 97.6%    | 97.6%             | 97.6%   | 97.6%    |  
| **Random Forest**         | 97.4%    | 97.3%             | 97.3%   | 97.4%    |  
| **XGBoost**               | 97.3%    | 97.2%             | 97.2%   | 97.3%    |  

🔹 **Why Random Forest?**  
While Extra Trees performed slightly better, **Random Forest** provided comparable accuracy while being easier to interpret. It also handles overfitting well by averaging multiple decision trees, ensuring robust performance on new data.  

### **Cross-Validation**  
To validate the model’s stability, I performed **5-fold cross-validation**, which confirmed a **mean accuracy of 97.1%**. This consistency across different splits of the dataset indicated that the model generalizes well.  

---

## **3. Model Evaluation**  

Once trained, I evaluated the **Random Forest classifier** on the test set, which contained **2,211 URLs**. The results were impressive:  

✔️ **Accuracy**: **98%** – The model correctly classified 98% of phishing and legitimate URLs.  
✔️ **Precision**: **98%** – Out of all URLs classified as phishing, 98% were truly phishing sites.  
✔️ **Recall**: **98%** – The model successfully detected 98% of actual phishing URLs.  
✔️ **F1 Score**: **98%** – A high balance between precision and recall.  
✔️ **ROC-AUC Score**: **98%** – Indicates strong performance in distinguishing between phishing and legitimate sites.  

### **Confusion Matrix Analysis**  
A confusion matrix helps visualize the model’s performance:  

|               | Predicted Legitimate (-1) | Predicted Phishing (1) |  
|--------------|--------------------------|------------------------|  
| **Actual Legitimate (-1)** | **951** (True Negatives) | **29** (False Positives) |  
| **Actual Phishing (1)**    | **21** (False Negatives)  | **1210** (True Positives) |  

🔹 **False Positives (29 cases)**: These are legitimate URLs incorrectly flagged as phishing. A lower false positive rate reduces unnecessary user frustration.  
🔹 **False Negatives (21 cases)**: These are phishing URLs incorrectly classified as legitimate. Minimizing false negatives is crucial since missing a phishing attempt can lead to security breaches.  

### **Precision vs. Recall Tradeoff**  
- A **high precision** means the model makes fewer false accusations (legitimate sites misclassified as phishing).  
- A **high recall** means the model catches more phishing sites but might flag some legitimate ones by mistake.  
- With both at **98%**, the model achieves an excellent balance.  

---

# **Building the Web App with Flask**  
Once the model was trained, I built a **Flask web application** to allow users to interact with it. The app consists of:

- **Frontend (HTML, CSS)**: A simple UI where users enter a URL.
- **Backend (Flask API)**:
  - The `/predict` endpoint receives the URL input.
  - The **FeatureExtractor** class extracts relevant features.
  - The **Random Forest model** predicts whether the URL is phishing.
  - Results are returned as a JSON response.

Flask predict route:
```python
@app.route("/predict", methods=["POST"])
def predict():
    try:
        data = request.get_json()
        if not data or "url" not in data:
            return jsonify({"success": False, "message": "No URL provided."}), 400
        url = data["url"]
        
        # Preprocess the input data
        extractor = FeatureExtractor(url)
        X_processed = extractor.extract_all_features()
        features = parse_features(X_processed)

        # Make prediction
        prediction = model.predict(X_processed)
        probability = model.predict_proba(X_processed)  
        probability = np.max(probability)
        return jsonify({
            "success": True,
            "prediction": int(prediction[0]),
            "probability": probability,
            "features": features
        })

    except Exception as e:
        logging.error(f"Error: {e}")
        status_code = extract_status_code(str(e))
        if status_code: 
            return jsonify({"success": False, "message": status_code}), 500
        else:
            return jsonify({"success": False, "message": "Invalid URL"}), 500
```
This API enables real-time URL classification, making phishing detection accessible to users.

**Key Directories and Files:**

- **`app/`**: Contains Flask application files.
  - **`static/`**: Static assets like CSS and JavaScript.
  - **`templates/`**: HTML templates.
  - **`__init__.py`**: Initializes the Flask app and caching.
  - **`routes.py`**: Defines Flask routes and prediction logic.
- **`data/`**: Data storage.
  - **`raw/`**: Original, unprocessed data.
  - **`processed/`**: Cleaned and processed data.
  - **`external/`**: External datasets or resources.
- **`notebooks/`**: Jupyter notebooks for exploration and modeling.
- **`src/`**: Source code for ML pipelines.
  - **`feature_pipeline.py`**: Feature engineering and selection.
  - **`model_pipeline.py`**: Model training and evaluation.
  - **`inference_pipeline.py`**: Data inference for direct predict in console.
  - **`config.py`**: Configuration parameters.
  - **`utils.py`**: Utility functions.
- **`models/`**: Serialized models and pipelines.
  - **`phishing_model.pkl`**: Trained machine learning model.
- **`reports/`**: Documentation and reports.
- **`requirements.txt`**: Python dependencies.
- **`setup.py`**: Package setup script.
- **`run_pipeline.py`**: Script to execute ML pipelines.
- **`run_app.py`**: Script to start the Flask application.
- **`Dockerfile`**: Docker configuration for containerization.
- **`.gitignore`**: Specifies files and directories to ignore in Git.
- **`README.md`**: Project documentation.

---

## **App Interface**  

Here’s how the application looks in action:  

### **1. Inputting a URL**  
The main interface provides a simple input field where users can enter a URL to check for phishing threats.  
![alt text](<Captura de pantalla 2025-02-06 233315.png>)

### **2. Scanning the URL**  
Once the URL is submitted, the app processes the request and returns a prediction. Below, the URL "randolphrogers.me" has been classified as safe with **95.00% probability**.  
![alt text](<Captura de pantalla 2025-02-06 233330.png>)

### **3. Debug View**  
For deeper insights, a debug version shows a breakdown of all extracted features and their individual scores, giving transparency to the classification process.  
![alt text](<Captura de pantalla 2025-02-06 233411.png>)

---

# **Results and Analysis**  

After successfully building the app and feature extraction pipeline, I tested the model on completely new data, including confirmed phishing sites. However, the results were disappointing. The model, which had performed **almost perfectly during evaluation**, struggled to correctly classify phishing sites in real-world scenarios.  

### **Identifying the Issue: Overfitting or Dataset Limitations?**  
At first, I suspected **overfitting**. I revisited my training and testing procedures, but all performance metrics suggested a well-trained model. To further investigate, I created a **new holdout dataset**, simulating real-world conditions, and evaluated the model again. The results? **Excellent performance, just like in training.**  

This raised a critical question: **Why did the model fail on actual phishing sites but perform well on test data?**  

### **Debugging with Feature Inspection**  
Using the app’s **debug mode**, I manually examined the results of every incorrectly classified phishing site, comparing their feature values with what the model had learned. This led to a key discovery:  

> **Every new phishing website followed almost all of the most important phishing detection features from my dataset.**  

The real issue became evident. **The dataset I used for training was obsolete.**  

### **The Cybersecurity Arms Race: Why Fresh Data Matters**  
In cybersecurity, there is an ongoing **race between attackers (red team) and defenders (blue team)**. New phishing techniques emerge as security measures evolve, and old detection patterns become **ineffective**. My dataset was outdated, meaning the model had **learned to detect past phishing trends** rather than the latest threats.  

### **Seeking Updated Data: A New Dataset, New Challenges**  
After realizing this, I searched for a more **recent dataset**. The best I found was **collected two years later** than my original dataset. However, it had **only 17 features compared to my 30**. I retrained and tested the model using this dataset, and while the results were **slightly weaker**, they were still comparable.  

This confirmed that while **data freshness is critical**, **feature richness also plays a huge role** in maintaining strong model performance.  

### **Limitations of Modern Data Collection**  
One of the biggest challenges in cybersecurity-related machine learning is **access to up-to-date data**. Many sources that previously provided useful insights are no longer available.  

For example:  
- **Alexa Internet**, which provided web traffic rankings for millions of websites, was **shut down in 2022**  
- **Several key threat intelligence databases** now restrict access behind costly APIs or enterprise-level services  
- Many features from my original dataset are now **harder to extract due to increased security measures** on websites  

As a side project, **these costs are prohibitively high**, making it difficult to continuously update and improve the model.  

### **Reflections: What This Project Taught Me**  
While the results were **not what I expected**, this project turned out to be a **valuable learning experience**. It forced me to  
- **Reevaluate my training process** and test my model under more realistic conditions  
- **Develop alternative evaluation methods** to simulate real-world data  
- **Think critically about data validity**, rather than just model accuracy  

This experience reinforced a key lesson. **In cybersecurity, models are only as good as the data they are trained on.**  

Additionally, I realized that the **probability score** displayed by my model might **not be calibrated properly**. Users might interpret it differently than what the model actually represents. A **probability calibration step** could improve interpretability.  

# **Challenges and Lessons Learned**  
Every project presents obstacles, and this one was no different. Here are some key challenges I faced  

- **Outdated Training Data**. The dataset I used was no longer effective in identifying modern phishing attacks  
- **Limited WHOIS Data**. WHOIS records were often incomplete, limiting domain age analysis  
- **Balancing Model Performance**. Reducing false positives was crucial. Incorrectly flagging legitimate sites could create user frustration  
- **Access to Fresh Data**. Many useful data sources are now restricted behind paid services, limiting feature extraction capabilities  

Despite these challenges, I gained **invaluable insights** into both **machine learning in cybersecurity** and the **importance of continuously evolving datasets**  

# **Future Improvements**  
There is always room for improvement. Here are a few areas I would like to explore next  

✅ **Use Deep Learning**. Experiment with neural networks for improved classification accuracy  

✅ **Enhance Feature Engineering**. Explore new feature extraction techniques, especially from **webpage content analysis**  

✅ **Integrate Threat Intelligence**. Cross-check URLs against real-time phishing databases for better validation  

✅ **Deploy as a Browser Extension**. Allow users to check URLs directly from their browsers, making the tool more accessible  

✅ **Calibrate Model Probability Scores**. Ensure displayed probabilities reflect actual confidence levels rather than misleading users  

# **Conclusion**  
This project was an exciting blend of **cybersecurity and machine learning**, allowing me to build a practical tool that can help users stay safe online. By extracting key features from URLs and using a trained model for classification, the app provides an automated phishing detection system  

However, the biggest takeaway was not about model accuracy. It was about **data relevance**. No matter how advanced a machine learning model is, if it is trained on outdated information, its predictions will become unreliable over time  

Moving forward, I aim to explore **more dynamic methods** for continuously updating and adapting phishing detection models. 

---

Thank you for reading. If you are interested in similar projects or have suggestions for enhancements, feel free to reach out.

---

**Keywords:** Phishing Detection, Machine Learning, Flask, Cybersecurity, URL Classification

---