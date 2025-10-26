# 🌿 Plant Disease Detection Project

> 🌱 A Deep Learning–based system to identify plant leaf diseases using CNN and Transfer Learning.

[![Made with Python](https://img.shields.io/badge/Made%20with-Python-blue.svg)](https://www.python.org/)
[![Open Source Love](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/gamikapunsisi/Plant_Disease_Detection)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/gamikapunsisi/Plant_Disease_Detection.svg)](https://github.com/gamikapunsisi/Plant_Disease_Detection/stargazers)

---

## 📖 Overview
This project uses **Convolutional Neural Networks (CNN)** to detect plant leaf diseases with high accuracy.  
The system helps farmers and researchers identify diseases early and take preventive actions to improve crop health and yield.

---

## 🧩 Dataset & Preprocessing
- **Dataset:** PlantVillage (15 classes, 20,638 images)  
- **Preprocessing:**  
  - Resized to 224×224 pixels  
  - Pixel normalization  
  - Extensive augmentation: rotation, shift, zoom, horizontal flip, brightness adjustment  
- **Splits:**  
  - Training: 80% (16,516 images)  
  - Validation: 20% (4,122 images)

---

## 🧠 Models Trained

### 1️⃣ Custom CNN  
- Deep convolutional architecture with **Batch Normalization** and **Dropout**  
- Trained from scratch  
- Regularized to prevent overfitting  

### 2️⃣ Transfer Learning — ResNet50  
- Pre-trained on **ImageNet**  
- Fine-tuned for 15 plant disease classes  
- Top layers trained, ResNet base frozen  

---

## 📊 Performance Metrics

| Model                | Accuracy | Precision | Recall | F1-Score |
|----------------------|---------:|----------:|-------:|---------:|
| **Custom CNN**       |  0.9791  |  0.9764   | 0.9781 |  0.9770  |
| **ResNet50 Transfer**|  0.6114  |  0.6112   | 0.5645 |  0.5467  |

---

## 🔍 Key Observations & Insights

- **Custom CNN** outperformed **ResNet50**, achieving nearly **98% accuracy** and excellent generalization.  
- **ResNet50’s lower performance (61%)** is likely due to frozen layers not adapting well to plant-specific patterns.  
- **Data augmentation**, **dropout**, and **batch normalization** successfully reduced overfitting.  
- Class-wise results show minor confusion among visually similar diseases.  
- Proper preprocessing and regularization played a major role in achieving stable and high accuracy.

---

## 🏁 Conclusion
- ✅ **Best Model:** Custom CNN (trained from scratch)  
- 🧠 **Why:** Tailored for dataset, strong regularization, and effective augmentation  
- 📋 **Requirements:** Dataset, preprocessing, model comparison, and analysis — all completed successfully  

---

## 🖼️ Example Output
| Input Image | Predicted Disease |
|--------------|-------------------|
| ![Leaf](https://via.placeholder.com/120x120.png?text=Leaf+Image) | *Powdery Mildew* |

---

## 💡 Future Work
- Add more plant species  
- Integrate live camera prediction  
- Deploy on AWS or Streamlit  
- Improve dataset balance  

---

## 👩‍💻 Author
**Gamika Punsisi**  
📧 [Email Me](mailto:gamikapunsisi@gmail.com)  
🌐 [GitHub Profile](https://github.com/gamikapunsisi)

---

## ⭐ Support
If you find this project helpful, please consider **starring** ⭐ the repository.  
It encourages me to improve and share more open-source AI projects!

---

