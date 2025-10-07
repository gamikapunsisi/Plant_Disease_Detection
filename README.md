## Plant Disease Detection Project — Results & Observations

### **Dataset & Preprocessing**
- **Dataset:** PlantVillage (15 classes, 20,638 images)
- **Preprocessing:** Images resized to 224×224, pixel normalization, extensive augmentation (rotation, shift, zoom, horizontal flip, brightness adjustment)
- **Splits:** 80% for training (16,516), 20% for validation (4,122)

---

### **Models Trained**
1. **Custom CNN**  
   - Deep convolutional architecture with batch normalization and dropout.
   - Trained from scratch, regularized to prevent overfitting.

2. **Transfer Learning (ResNet50)**
   - Pre-trained on ImageNet, fine-tuned for 15 plant disease classes.
   - Top layers trained, ResNet base frozen.

---

### **Performance Metrics**

| Model                | Accuracy | Precision | Recall | F1-Score |
|----------------------|---------:|----------:|-------:|---------:|
| **Custom CNN**       |  0.9791  |  0.9764   | 0.9781 |  0.9770  |
| **ResNet50 Transfer**|  0.6114  |  0.6112   | 0.5645 |  0.5467  |

---

### **Key Observations & Justification**

- **Custom CNN outperformed ResNet50 transfer learning** on all metrics, achieving nearly 98% validation accuracy and macro F1-score.
- **ResNet50 transfer learning** performed significantly worse (~61% accuracy). Possible reasons:
    - Custom CNN may be more tailored for the specific image features and dataset size.
    - ResNet50's frozen layers may not capture unique plant disease patterns without further fine-tuning.
- **Overfitting was successfully prevented** in Custom CNN:
    - Validation accuracy closely tracked training accuracy.
    - Data augmentation, dropout, and batch normalization contributed to high generalization.
    - Regularization and learning rate scheduling further improved stability.
- **Class-wise analysis** (from confusion matrix and metrics): Most classes were classified correctly; minor confusion in visually similar diseases.
- **Augmentation and careful preprocessing** were critical to achieving high accuracy.

---

### **Conclusion**

- **Best model:** Custom CNN (from scratch)
- **Why:** Better fit for the dataset, effective regularization, and augmentation.
- **Project Requirements:** All fulfilled—dataset, preprocessing, two models, metrics, and reasoned observations.

---

