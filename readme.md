# 🚦 Traffic Sign Recognition in Vietnam using YOLOv8

## 📖 Introduction

This project studies and applies **YOLOv8 (You Only Look Once)** to recognize traffic signs in Vietnam.
As intelligent transportation systems and autonomous vehicles continue to grow, automatic traffic sign recognition becomes a crucial component to ensure **safety, accuracy, and real-time responsiveness**.

---

## 🧩 Motivation

* **Practicality**: Vietnam has a diverse system of traffic signs with various shapes, colors, and categories.
* **Challenges**: Lighting conditions, weather, viewing angles, and noise affect the accuracy of recognition.
* **Solution**: Applying YOLOv8 – a modern, high-speed model suitable for real-time detection.

👉 *Illustration of YOLO detecting traffic signs*
<img width="477" height="479" alt="image" src="https://github.com/user-attachments/assets/eba2d7bd-ac9d-4c42-acf4-9962a0426ecd" />


---

## 📚 Theoretical Background

### Object Detection

* **Localization**: Determine object positions using bounding boxes.
* **Classification**: Identify the object type.
* **Optimization**: Data augmentation and fine-tuning to improve accuracy.

### Representative Models

* **R-CNN, Fast R-CNN, Faster R-CNN**: high accuracy but slow.
* **SSD**: fast but less accurate.
* **YOLO**: a good balance of speed and accuracy, suitable for real-time applications.

👉 *Comparison of YOLOv8 with previous versions*

<img width="1135" height="443" alt="image" src="https://github.com/user-attachments/assets/478637fe-f400-422c-99ba-4065cb29d9d9" />


---

## 🗂️ Dataset

* The dataset includes various categories of traffic signs: **prohibitory, mandatory, guidance, warning**.
* Collected under real-world conditions: different lighting, weather, and viewing angles.
* Preprocessing steps: labeling, normalization, and creating the `mydataset.yaml` file.

👉 *Dataset examples and sample labels*

<img width="864" height="183" alt="image" src="https://github.com/user-attachments/assets/988519b1-a468-4691-a642-ff328558f9cf" />

<img width="430" height="115" alt="image" src="https://github.com/user-attachments/assets/9dce2ebe-d88c-4340-849d-97d43ccf4eb1" />


---

## ⚙️ Model Development

### YOLOv8 Architecture

* CNN backbone for feature extraction.
* Detection head for predicting bounding boxes and classes.

👉 *YOLOv8 Architecture*

<img width="597" height="212" alt="image" src="https://github.com/user-attachments/assets/cdeae831-4017-4471-b9c6-d116bd165a5c" />


### Data Configuration

* The `mydataset.yaml` file defines the train/test paths, number of classes, and class names.

👉 *Data configuration*
<img width="379" height="637" alt="image" src="https://github.com/user-attachments/assets/b79a08d4-afe7-45ee-8b13-1af6cc8c9043" />

<img width="543" height="412" alt="image" src="https://github.com/user-attachments/assets/ad44e428-8c87-4baa-a175-75ee52e44470" />


### Model Training

* Training performed on Google Colab.
* Hyperparameters include batch size, learning rate, and epochs.
* Utilizes a pretrained YOLOv8 model for fine-tuning.

👉 *Training process*

<img width="756" height="157" alt="image" src="https://github.com/user-attachments/assets/35e70237-1a8e-4a3c-98f2-3c9454af726e" />

<img width="756" height="100" alt="image" src="https://github.com/user-attachments/assets/b20bde16-e7ff-479e-87a9-ead3403822b7" />

<img width="674" height="323" alt="image" src="https://github.com/user-attachments/assets/47690523-36e8-43d3-8815-42fc4467629c" />

---

## 📊 Results

* The model accurately recognizes many types of traffic signs.
* High accuracy under good lighting conditions, lower in noisy environments.
* Near real-time processing speed.

👉 *Prediction results*

<img width="576" height="389" alt="image" src="https://github.com/user-attachments/assets/10874063-c4ea-4e84-8467-20bbd47f2080" />

<img width="561" height="379" alt="image" src="https://github.com/user-attachments/assets/ce31e919-2e9f-4e2f-a6c2-a748c230c399" />

<img width="459" height="310" alt="image" src="https://github.com/user-attachments/assets/5664b781-0223-42f6-a97c-192ea29181ef" />

<img width="459" height="310" alt="image" src="https://github.com/user-attachments/assets/cb31ab38-6704-45ab-a483-e8b89010312f" />


👉 *Final results after training*
<img width="675" height="346" alt="image" src="https://github.com/user-attachments/assets/0a302ae0-088c-4ce1-8d15-5209ab4dd282" />

<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/0efdff9e-2cb1-467c-a1db-1d120b3044b1" />

---

## 📝 Conclusion & Future Work

* ✅ YOLOv8 demonstrates strong performance in recognizing Vietnamese traffic signs.
* ⚠️ Limitations: dataset size is still small, and training resources are limited.
* 🚀 Future improvements:

  * Expand the dataset with more diverse real-world conditions.
  * Optimize the model for deployment on IoT devices or autonomous vehicles.
  * Integrate with advanced driver-assistance systems (ADAS).

---

## 📂 Project Structure

```
├── images/                # Contains all illustrative images
│   ├── hinh1.1.png
│   ├── hinh1.2.png
│   ├── hinh2.1.png
│   ├── hinh2.2.png
│   ├── hinh3.1.png
│   ├── hinh3.2.png
│   ├── hinh3.3.png
│   ├── hinh3.5.png
│   ├── hinh4.3.png
│   ├── hinh4.4.png
│   └── hinh4.6.png
├── mydataset.yaml         # Dataset configuration file
├── train_yolov8.ipynb     # Model training notebook
└── README.md              # Project documentation
```

---

## 👨‍💻 Author

* **Phạm Nguyễn Nhật Trường** – Faculty of Information Technology, Nguyen Tat Thanh University
* Supervisor: **Phạm Đình Tài**
