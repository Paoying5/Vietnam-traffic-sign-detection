Problem Discovery
1. Project Context
Problem

Traffic signs in Vietnam have many different:

shapes
colors
visual appearances
meanings
contexts
sizes
viewing angles

A computer vision system can be used to automatically detect traffic signs from road images or video.

The expected system is:

Road Image / Video
        │
        ▼
Object Detection Model
        │
        ├── Traffic sign class
        ├── Bounding box
        ├── Confidence
        └── Location
        │
        ▼
Structured Traffic-Sign Information
        │
        ├── Analytics
        ├── API
        ├── Visualization
        └── Application


The current project focuses first on building a reliable traffic-sign detection pipeline from the existing labeled dataset.

2. Stakeholders

The project should be considered from four perspectives.

2.1 End User

A user provides an image:

road.jpg


The system should return structured detection results such as:

{
  "detections": [
    {
      "class": "No Entry",
      "confidence": 0.94,
      "bbox": [120, 80, 240, 210]
    }
  ]
}


The user cares about:

correct traffic-sign detection
correct sign classification
accurate bounding boxes
confidence score
response speed
2.2 Data Analyst

The Data Analyst needs to understand the dataset.

Important questions:

How many images are available?
How many annotations are available?
How many traffic-sign classes exist?
Which classes are most common?
Which classes are rare?
Is the dataset imbalanced?
How many images contain each class?
How many images contain multiple objects?
Are there empty annotation files?
Are there suspicious or abnormal images?
How are bounding-box sizes distributed?
2.3 Data Scientist

The Data Scientist needs to understand how the dataset affects model performance.

Important questions:

Does class imbalance affect model performance?
Which classes are difficult to detect?
How does the number of samples affect AP?
What is the baseline model performance?
What are the Precision and Recall values?
What are mAP@50 and mAP@50:95?
Which classes produce the most false positives?
Which classes produce the most false negatives?
Does data augmentation improve performance?
Is the model overfitting?
2.4 Data Engineer

The Data Engineer is responsible for making the dataset reliable and reproducible.

Important questions:

Is the raw dataset structured correctly?
Does every image have a corresponding label file?
Does every label file correspond to an image?
Are annotation files valid YOLO format?
Are class IDs valid?
Are normalized coordinates valid?
Are corrupted images present?
Are empty labels intentional?
Can the dataset be versioned?
Can the dataset be reproduced later?
How can a new dataset version be added safely?
3. Current Dataset

The project currently contains a dataset under:

data/
├── images/
├── labels/
└── classes.txt


The repository also contains project documentation under:

docs/


The dataset itself is currently ignored by Git through:

.gitignore


Specifically:

data/


is ignored.

This is intentional for the current stage because the dataset is approximately hundreds of megabytes in size.

4. Dataset Inventory

Current verified dataset statistics:

Metric	Value
Images	3,216
Label files	3,216
Classes	51
Total annotations	8,341
Image storage	~764 MB
Label storage	~13 MB
Empty label files	18
Images without labels	0
Labels without images	0
Class ID minimum	0
Class ID maximum	50

The dataset therefore contains:

3,216 images
3,216 label files
8,341 bounding-box annotations
51 classes


Average number of annotations per image:

8,341 / 3,216 ≈ 2.59


This is an overall average only and does not describe the distribution of objects across images.

5. Annotation Format

The label files use YOLO-style annotation format.

Example:

10 0.368750 0.509259 0.018750 0.033333


The five fields represent:

class_id
x_center
y_center
width
height


The coordinates are normalized to the range:

0 → 1


For example:

10


is the class ID.

The remaining values describe the normalized bounding box.

6. Dataset Integrity Checks

Before training a model, the dataset needs to be validated.

The following checks have already been performed.

6.1 Image / Label Matching

Result:

Images without labels: 0
Labels without images: 0


Therefore every currently detected image has a corresponding label file, and every label file has a corresponding image.

This is a positive result.

6.2 Class ID Validation

The dataset contains:

51 classes


Therefore valid class IDs are:

0–50


The actual label files use:

0–50


Class ID 50 was initially flagged because an earlier validation assumed there were only 50 classes.

After verifying:

grep -c . data/classes.txt


the dataset was confirmed to contain:

51 classes


Therefore:

class 50


is a valid class ID.

The final class mapping still needs to be treated as an important dataset contract and should be verified before training.

6.3 Normalized Coordinate Validation

The dataset was checked for coordinates outside the valid normalized range.

Result:

Invalid normalized coordinates:
0


No invalid normalized coordinates were found in this check.

6.4 Bounding Box Width / Height Validation

Bounding-box width and height were checked for values outside:

0 → 1


Result:

Invalid width/height:
0


No invalid width or height values were found in this check.

7. Empty Annotation Files

The dataset contains:

18 empty label files


This requires investigation.

An empty YOLO label file does not automatically mean the dataset is corrupted.

There are two possible interpretations.

Case A — Negative Sample

The image genuinely contains no traffic sign.

In that case:

image.jpg
label.txt


where label.txt is empty can be a valid object-detection negative sample.

Case B — Missing Annotation

The image contains one or more traffic signs, but the annotation is missing.

In that case the label is incomplete and should be fixed.

Therefore:

The 18 empty label files must not be deleted automatically.

They should be visually inspected and classified as either:

valid negative samples


or:

annotation errors

8. Class Distribution

The dataset contains annotations for all 51 class IDs.

The annotation distribution is highly imbalanced.

The current statistics show:

Minimum annotations/class = 4
Maximum annotations/class = 1,220


Therefore:

Maximum / Minimum = 305


The most frequent class has approximately 305 times as many annotations as the rarest class.

This is a significant class-imbalance issue.

9. Why Class Imbalance Matters

A model trained directly on this dataset may learn frequent classes much better than rare classes.

For example:

Frequent class
1,220 annotations
        │
        ▼
More training examples
        │
        ▼
Better opportunity to learn visual patterns


while:

Rare class
4 annotations
        │
        ▼
Very little training information
        │
        ▼
High risk of poor detection


Therefore overall mAP alone may not adequately describe model quality.

Per-class metrics will be important.

10. Images per Class vs Annotations per Class

Two different statistics must be maintained.

Annotations per Class

This measures:

How many bounding boxes belong to each class?

Example:

Class X → 1,220 annotations

Images per Class

This measures:

How many different images contain that class?

These values can differ because one image can contain multiple objects of the same class.

Therefore both metrics are useful.

11. Important Dataset Questions Still Open

Several questions remain unanswered.

11.1 Class Mapping

The exact mapping:

class_id → class name


must be verified carefully.

This is especially important because previous command output showed an apparent offset between class IDs and names.

Before training, we need to establish one authoritative mapping from:

classes.txt


to:

class IDs 0–50


No model experiment should rely on an unverified class-name mapping.

11.2 Empty Labels

The 18 empty label files need visual inspection.

We need to determine:

18 empty labels
        │
        ├── valid negative samples
        │
        └── missing annotations

11.3 Multiple Objects per Image

The dataset contains:

8,341 annotations
3,216 images


so the average is approximately:

2.59 annotations/image


However, the exact distribution still needs to be measured.

We should determine:

images with zero annotations
images with one annotation
images with multiple annotations
maximum number of annotations in one image
11.4 Bounding Box Distribution

We have verified that bounding-box coordinates are valid.

However, validity is not the same as quality.

We still need to analyze:

box width
box height
box area
aspect ratio
very small objects
very large objects
object location within images

This is important because traffic signs can occupy very small portions of road images.

12. Existing Model Experiment

A previous notebook experiment reported approximately:

Precision = 0.639
Recall = 0.413
mAP50 = 0.442
mAP50-95 = 0.323


However, the previous configuration used:

train: data/train
val:   data/train


This means the same dataset was used for both training and validation.

Therefore these metrics should not be treated as a proper independent validation benchmark.

The result can be kept as historical information, but it should not be used as the final baseline.

13. Correct Experiment Structure

The project should move toward:

Raw Dataset
     │
     ▼
Dataset Validation
     │
     ▼
Dataset Cleaning
     │
     ▼
Dataset Split
     │
     ├── train
     ├── val
     └── test
     │
     ▼
Baseline Model
     │
     ▼
Evaluation
     │
     ├── Precision
     ├── Recall
     ├── mAP@50
     ├── mAP@50:95
     ├── Per-class AP
     └── Confusion Matrix


The validation and test sets must be independent from the training set.

14. Incremental Training Problem

A previous approach used:

trained_files = []

files_to_train = [
    f for f in train_files
    if f not in trained_files
]


This should not be considered the main strategy for incremental ML training.

Machine-learning training should not be modeled as:

Image A was trained
Image B was not trained
→ train only Image B


Instead, experiments should be tracked using:

Dataset Version
       +
Model Version
       +
Experiment Configuration
       +
Checkpoint
       +
Metrics


For example:

Dataset v1
    │
    ▼
Experiment 001
    │
    ▼
YOLOv8n
    │
    ▼
best.pt
    │
    ▼
Metrics


Later:

Dataset v2
    │
    ▼
Experiment 002
    │
    ▼
YOLOv8n
    │
    ▼
Comparison
    │
    ▼
Model v2


This makes experiments reproducible and comparable.

15. Success Metrics

The project should eventually define success metrics at three levels.

15.1 Data Quality
Image count
Annotation count
Missing labels
Orphan labels
Invalid labels
Empty labels
Duplicate images
Corrupted images
Class imbalance
Bounding-box statistics

15.2 Model Quality
Precision
Recall
mAP@50
mAP@50:95
Per-class AP
Confusion matrix
False positives
False negatives

15.3 Production Quality
Inference latency
Throughput
Model size
Memory usage
CPU inference performance
GPU inference performance
API response time
API availability

16. Initial Success Criteria

Exact targets should not be chosen yet.

The targets should be defined after:

validating the dataset
creating a proper train/validation/test split
training a baseline model
measuring baseline performance
understanding the difficulty of each class

The final project may eventually define targets such as:

Model
------------------------------
mAP@50        >= target
mAP@50:95     >= target
Recall        >= target
Per-class AP  >= target

Production
------------------------------
Latency       <= target ms
Model size    <= target MB
API uptime    >= target


The target values should be based on the actual use case rather than arbitrary numbers.

17. Current Problem Statement

The current problem is therefore not simply:

"Train YOLO to detect Vietnamese traffic signs."

The real engineering problem is:

Build a reliable, reproducible and measurable computer-vision pipeline that can detect Vietnamese traffic signs from road images, while ensuring that the dataset is valid, class mappings are correct, class imbalance is understood, train/validation/test splits are independent, experiments are reproducible, and model performance can be evaluated at both overall and per-class levels.

18. Current Findings

At this stage, the following facts have been verified:

✓ 3,216 images
✓ 3,216 label files
✓ 8,341 annotations
✓ 51 classes
✓ Class IDs range from 0–50
✓ No image without a label
✓ No label without an image
✓ No invalid normalized coordinates found
✓ No invalid width/height found
✓ 18 empty label files
✓ Strong class imbalance
✓ Minimum class count = 4
✓ Maximum class count = 1,220
✓ Max/min imbalance ratio = 305x


The following items remain open:

□ Verify authoritative class ID → class name mapping
□ Inspect the 18 empty label files
□ Measure annotations per image
□ Measure multiple-object images
□ Analyze bounding-box size distribution
□ Check for corrupted images
□ Check for duplicate / near-duplicate images
□ Create proper train/validation/test split
□ Establish a clean baseline experiment
□ Define final model and production targets

19. Immediate Next Step

Do not train another model yet.

The next step is:

Verify class mapping
        ↓
Inspect empty labels
        ↓
Complete dataset validation
        ↓
Create dataset split
        ↓
Train clean baseline


The objective of the current phase is to understand and validate the data before drawing conclusions from model performance.

20. Working Principle

The project will follow this principle:

Understand the data
        ↓
Validate the data
        ↓
Measure the data
        ↓
Build a baseline
        ↓
Experiment
        ↓
Compare
        ↓
Improve
        ↓
Deploy


The project should avoid optimizing the model before establishing that the dataset and evaluation methodology are trustworthy.