Được. Vì bạn muốn **một file README độc lập bên ngoài project**, mục tiêu của nó không phải là mô tả GitHub chung chung, mà là làm **“checkpoint phục hồi công việc”**: ngày mai mở máy lên chỉ cần đọc từ đầu đến cuối là biết hôm nay đã làm gì, phát hiện gì, chưa làm gì, và bước tiếp theo là gì.

 Bạn có thể tạo file ở `~/Documents/Vietnam-traffic-sign-detection-README.md` để nó nằm **ngoài repository**.

 # Vietnam Traffic Sign Detection — Work Continuation README

 > **Mục đích:** Đây là file checkpoint để tiếp tục project sau khi phiên làm việc trước bị mất/reset.\
>  **Ngày ghi nhận:** 2026-09-05\
>  **Project:** `Vietnam-traffic-sign-detection`

---

 # 1\. Project đang làm gì?

 Project xây dựng một hệ thống **Vietnam Traffic Sign Detection**.

 Mục tiêu cuối cùng:

```
Road image / video
        │
        ▼
Object Detection Model
        │
        ├── Sign type
        ├── Bounding box
        ├── Confidence
        └── Location
                │
                ▼
Structured traffic-sign information
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
    Analytics   API   Visualization
```

 Project không chỉ nhằm mục tiêu:

 > "Train YOLO và lấy mAP"

 Mà hướng tới tư duy một hệ thống ML hoàn chỉnh:

```
Data
 ↓
Data Validation
 ↓
Data Analysis
 ↓
Dataset Versioning
 ↓
Training
 ↓
Evaluation
 ↓
Model Versioning
 ↓
Inference
 ↓
API
 ↓
Monitoring
```

---

 # 2\. Các vai trò cần suy nghĩ trong project

 ## Data Analyst

 Cần trả lời:

 - Traffic sign nào xuất hiện nhiều nhất?
- Traffic sign nào hiếm nhất?
- Phân bố class như thế nào?
- Dataset có mất cân bằng không?
- Class nào có quá ít dữ liệu?
- Bounding box có kích thước như thế nào?
- Có image bất thường không?
- Annotation có vấn đề không?

 ## Data Scientist

 Cần trả lời:

 - Dataset imbalance ảnh hưởng model thế nào?
- Số lượng training samples ảnh hưởng mAP thế nào?
- Model khó nhận diện class nào?
- False positive / false negative tập trung ở đâu?
- Confidence threshold phù hợp?
- Data augmentation có tác dụng?
- Model có overfit không?

 ## Data Engineer

 Cần xây dựng:

```
Raw Dataset
     ↓
Validation
     ↓
Cleaning
     ↓
Transformation
     ↓
Dataset Version
     ↓
Training Dataset
```

 Cần quan tâm:

 - Dataset validation
- Annotation schema
- Corrupted image
- Missing image/label
- Dataset version
- Reproducibility
- Data pipeline

 ## AI Engineer

 Cần quan tâm:

```
Client
  ↓
FastAPI
  ↓
Preprocessing
  ↓
ONNX Model
  ↓
Postprocessing
  ↓
JSON Response
```

 Các vấn đề:

 - Inference latency
- CPU/GPU performance
- ONNX
- Model size
- Batch inference
- API serving
- Input validation
- Model versioning
- Monitoring

---

 # 3\. Dataset hiện tại

 Dataset nằm trong:

```
data/
├── images/
├── labels/
└── classes.txt
```

 Không có train/val split chuẩn ở thời điểm hiện tại.

 ## Images

```
data/images/
```

 Có:

```
0001.jpg
0002.jpg
...
3216.jpg
```

 Tổng:

```
3216 images
```

 Dung lượng thực tế kiểm tra bằng:

```
du -sh data/images
```

 Kết quả:

```
764M    data/images
```

---

 # 4\. Labels

 Labels nằm ở:

```
data/labels/
```

 Có:

```
0001.txt
0002.txt
...
3216.txt
```

 Tổng:

```
3216 label files
```

 Dung lượng:

```
13M
```

 Đã kiểm tra image/label matching.

 Kết quả:

```
Images without labels: 0
Labels without images: 0
```

 =\> **Tất cả 3216 images đều có label file tương ứng và ngược lại.**

---

 # 5\. Format annotation

 Một annotation mẫu:

```
10 0.368750 0.509259 0.018750 0.033333
```

 Đây là format YOLO:

```
class_id
x_center
y_center
width
height
```

 Các tọa độ được normalize về khoảng `[0, 1]`.

 Ví dụ:

```
10
0.368750
0.509259
0.018750
0.033333
```

---

 # 6\. Classes

 File:

```
data/classes.txt
```

 Có:

```
51 classes
```

 Đã kiểm tra:

```
grep -c . data/classes.txt
```

 Kết quả:

```
51
```

 Class ID hợp lệ về mặt danh sách phải là:

```
0 → 50
```

---

 # 7\. Một vấn đề rất quan trọng: class ID 50

 Ban đầu chúng ta tưởng dataset có 50 classes vì một số command trước đó cho kết quả không chính xác.

 Sau khi kiểm tra kỹ:

```
classes.txt = 51 classes
```

 Class ID được sử dụng:

```
0 → 50
```

 Trong đó class `50` xuất hiện:

```
41 annotations
```

 Các file chứa class 50:

```
3085.txt
3086.txt
...
3125.txt
```

 Kiểm tra:

```
grep -h '^50 ' data/labels/*.txt | wc -l
```

 Kết quả:

```
41
```

 10 class cuối trong `classes.txt`:

```
42 Speed limit (80km/h)
43 Speed limit (40km/h)
44 Left Turn
45 Low Clearance
46 Other Danger
47 Go Straight
48 No Parking
49 Containers Only
50 No U-Turn for Cars
51 Level Crossing with Barriers
```

 **Lưu ý cực kỳ quan trọng:**

 `nl -ba data/classes.txt` đánh số dòng từ `1`.

 Nhưng YOLO class ID bắt đầu từ `0`.

 Do đó:

```
class ID 0 → dòng 1 trong classes.txt
class ID 1 → dòng 2
...
class ID 50 → dòng 51
```

 Không được nhầm line number với class ID.

---

 # 8\. Phân bố annotation theo class

 Đã chạy:

```
awk '{print $1}' data/labels/*.txt |
sort -n |
uniq -c |
sort -k2,2n
```

 Kết quả số annotation theo class:

```
0  → 312
1  → 22
2  → 455
3  → 453
4  → 32
5  → 257
6  → 303
7  → 178
8  → 213
9  → 43
10 → 1220
11 → 26
12 → 285
13 → 69
14 → 372
15 → 84
16 → 29
17 → 198
18 → 90
19 → 105
20 → 29
21 → 52
22 → 11
23 → 132
24 → 470
25 → 56
26 → 25
27 → 85
28 → 36
29 → 27
30 → 40
31 → 139
32 → 38
33 → 186
34 → 19
35 → 82
36 → 87
37 → 75
38 → 47
39 → 189
40 → 459
41 → 210
42 → 274
43 → 21
44 → 42
45 → 4
46 → 196
47 → 286
48 → 221
49 → 16
50 → 41
```

 Tổng annotation:

```
8341
```

---

 # 9\. Dataset imbalance

 Đã tính:

```
Total annotations: 8341
Classes used: 51
Min annotations/class: 4
Max annotations/class: 1220
Max/Min ratio: 305
```

 Đây là một phát hiện quan trọng.

 Class nhiều nhất:

```
Class 10
1220 annotations
```

 Class ít nhất:

```
Class 45
4 annotations
```

 Tỷ lệ:

```
1220 / 4 = 305
```

 =\> Dataset **rất mất cân bằng**.

 Nhưng chưa được kết luận rằng phải oversampling/undersampling ngay.

 Trước tiên cần hiểu:

 - class nào thực sự quá ít?
- các annotation thuộc sequence/video liên tục hay độc lập?
- train/validation split có làm vấn đề nghiêm trọng hơn không?
- class hiếm có đủ diversity không?
- model baseline hoạt động thế nào?

---

 # 10\. Images per class

 Đã chạy thêm:

```
for id in $(seq 0 50); do
    count=$(grep -l "^$id " data/labels/*.txt | wc -l)
    class=$(sed -n "$((id + 1))p" data/classes.txt)
    echo "$id | $count | $class"
done
```

 Điều này **khác với annotation count**.

 Ví dụ:

```
annotation count
```

 đếm số bounding boxes.

 Trong khi:

```
images per class
```

 đếm số image có chứa class đó.

 Hai metric này cần được giữ riêng.

 Một image có thể có:

```
class 10
class 40
class 24
```

 thì image đó được tính cho cả 3 classes.

 Vì vậy tổng `images per class` không nhất thiết bằng 3216.

---

 # 11\. Empty label files

 Đã kiểm tra:

```
find data/labels -maxdepth 1 -type f -empty | wc -l
```

 Kết quả:

```
18
```

 =\> Có **18 label files rỗng**.

 Điều này rất quan trọng.

 Không được tự động xóa chúng.

 Một label file rỗng có thể có hai ý nghĩa:

 ### Trường hợp A

 Image thực sự không có traffic sign.

 Khi đó empty label là hợp lệ trong object detection.

 ### Trường hợp B

 Annotation bị thiếu.

 Khi đó là lỗi dataset.

 =\> Bước tiếp theo cần kiểm tra 18 image tương ứng bằng visual inspection.

---

 # 12\. Image/label consistency

 Đã chạy kiểm tra:

```
echo "=== Images without labels ==="
for f in data/images/*; do
    base=$(basename "${f%.*}")
    [ -f "data/labels/$base.txt" ] || echo "$f"
done

echo "=== Labels without images ==="
for f in data/labels/*.txt; do
    base=$(basename "${f%.txt}")
    [ -f "data/images/$base.jpg" ] || echo "$f"
done
```

 Kết quả:

```
Images without labels:
NONE

Labels without images:
NONE
```

 =\> Pairing giữa images và labels hiện tại là tốt.

---

 # 13\. Annotation format

 Đã kiểm tra format bằng:

```
awk 'NF != 5 {
    print FILENAME ":" FNR ":" $0
}' data/labels/*.txt
```

 Command này cho ra nhiều dòng.

 Nhưng cần lưu ý:

 **Kết quả này có thể bị ảnh hưởng bởi khoảng trắng cuối dòng hoặc cách command được copy/paste.**

 Ví dụ:

```
10 0.472053 0.522558 0.022574 0.039772
```

 nhìn bằng mắt vẫn có 5 trường dữ liệu.

 Do đó chưa được kết luận rằng tất cả các dòng đó là annotation format lỗi.

 Ngày mai cần viết một validation script Python sạch hơn để kiểm tra:

```
exactly 5 tokens
class_id integer
class_id in [0, 50]
x_center ∈ [0,1]
y_center ∈ [0,1]
width ∈ (0,1]
height ∈ (0,1]
```

---

 # 14\. Normalized coordinates

 Đã kiểm tra:

```
=== Invalid normalized coordinates ===
```

 Kết quả:

```
NONE
```

 và:

```
=== Invalid width/height ===
```

 Kết quả:

```
NONE
```

 =\> Chưa phát hiện bounding box nào có coordinate rõ ràng vượt khỏi normalized range.

 Đây là tín hiệu tốt.

---

 # 15\. Một vấn đề đặc biệt trong annotation

 Class ID `50` được phát hiện trong:

```
3085 → 3125
```

 và xuất hiện liên tục trong nhiều frame.

 Điều này gợi ý rằng dataset có thể chứa **sequence/frame liên tiếp**, chứ không hoàn toàn là 3216 image độc lập.

 Đây là điều rất quan trọng khi chia train/validation.

 Ví dụ nếu:

```
3085
3086
3087
...
3125
```

 là các frame của cùng một video/sequence thì việc random split có thể gây:

```
train:
3085
3086
3087

validation:
3088
3089
```

 =\> model nhìn thấy những frame gần như giống nhau ở cả train và validation.

 Khi đó validation metric có thể bị optimistic.

 **Cần điều tra trước khi split dataset.**

---

 # 16\. Tổng annotation chính xác hiện tại

 Có một điểm cần nhớ:

 Ban đầu có một command cho:

```
Classes:
50 data/classes.txt
Annotations:
7239
```

 Nhưng command đó bị viết sai logic vì:

```
cat data/labels/*.txt | wc -l
```

 chỉ đếm số dòng annotation, và output bị ghép với:

```
50 data/classes.txt
```

 Sau đó đã có thống kê class distribution đầy đủ và xác nhận:

```
Total annotations: 8341
```

 =\> **8341 là con số hiện đang dùng.**

---

 # 17\. Git status hiện tại

 Repository:

```
Vietnam-traffic-sign-detection
```

 Branch:

```
main
```

 Status ban đầu:

```
On branch main
Your branch is up to date with 'origin/main'.

Untracked files:
    docs/
```

 =\> `docs/` hiện đang là untracked.

 Dataset `data/` đang bị ignore.

 Đã kiểm tra:

```
git check-ignore -v data/images/0001.jpg
git check-ignore -v data/labels/0001.txt
git check-ignore -v data/classes.txt
```

 Kết quả:

```
.gitignore:2:data/
```

 =\> Toàn bộ `data/` đang bị `.gitignore` ignore.

 Điều này **không phải lỗi** nếu dataset không muốn commit lên Git.

 Dataset khoảng:

```
776M
```

 nên không nên commit trực tiếp vào repository thông thường.

---

 # 18\. Vấn đề validation trước đây

 Notebook/model experiment trước đây có:

```
train: /.../data/train
val:   /.../data/train
```

 Tức là:

```
TRAIN
  ↓
data/train

VALIDATION
  ↓
data/train
```

 Train và validation đang dùng cùng dataset.

 Kết quả từng thấy:

```
Precision    0.639
Recall       0.413
mAP50        0.442
mAP50-95     0.323
```

 **Không nên coi đây là validation experiment chuẩn.**

 Trước khi so sánh model hoặc tối ưu hyperparameter, cần tạo train/validation/test split đúng.

---

 # 19\. Không nên tiếp tục dùng trained\_files.txt

 Một ý tưởng cũ là:

```
trained_files = []

files_to_train = [
    f for f in train_files
    if f not in trained_files
]
```

 Cách này không phải incremental ML training chuẩn.

 Không nên nghĩ:

```
Image A đã train
Image B chưa train
→ chỉ train B
```

 Thay vào đó phải quản lý:

```
Dataset version
       +
Model version
       +
Experiment configuration
       +
Checkpoint
       +
Metrics
```

 Ví dụ:

```
dataset v1
    ↓
experiment 001
    ↓
YOLOv8n
    ↓
best.pt
    ↓
metrics
```

 Sau này:

```
dataset v2
    ↓
experiment 002
    ↓
YOLOv8n
    ↓
comparison
    ↓
model v2
```

---

 # 20\. Vì sao hôm nay chạy nhiều command trên Terminal?

 Các command vừa chạy không phải để "làm project ngay".

 Chúng là **Data Discovery / Dataset Audit**.

 Mục đích là trả lời:

```
Dataset thực tế đang có gì?
```

 Trước khi train model cần biết:

```
Có bao nhiêu image?
Có bao nhiêu label?
Có thiếu pair không?
Có bao nhiêu class?
Class ID có hợp lệ không?
Có annotation lỗi không?
Class imbalance thế nào?
Có empty labels không?
Có sequence không?
Có nguy cơ data leakage không?
```

 Đây là bước rất quan trọng.

 Không nên nhảy thẳng vào:

```
YOLO training
```

 khi chưa hiểu dataset.

---

 # 21\. Những gì đã xác nhận được

 Hiện tại có thể ghi nhận:

```
✓ 3216 images
✓ 3216 label files
✓ Không có image thiếu label
✓ Không có label thiếu image
✓ 51 classes
✓ Class ID thực tế: 0 → 50
✓ Class 50 xuất hiện 41 lần
✓ Tổng annotations: 8341
✓ Dataset imbalance rất mạnh
✓ 18 empty label files
✓ Normalized coordinates chưa phát hiện vượt range
✓ width/height chưa phát hiện vượt range
✓ data/ đang được .gitignore
✓ Dataset khoảng 776 MB
✓ images khoảng 764 MB
✓ labels khoảng 13 MB
```

---

 # 22\. Những gì CHƯA được xác nhận

 Chưa được kết luận:

```
? 18 empty labels có hợp lệ không?
? Dataset có duplicate images không?
? Dataset có sequence/video leakage không?
? Train/validation/test split nên chia thế nào?
? Annotation có thực sự chính xác không?
? Có bounding boxes quá nhỏ không?
? Có bounding boxes bất thường không?
? Có image corrupted không?
? Có image duplicate không?
? Class distribution theo image sau khi split thế nào?
? Model baseline đúng cần được train lại thế nào?
```

---

 # 23\. Bước tiếp theo khi quay lại project

 **Không train model ngay.**

 Tiếp tục Data Audit theo thứ tự:

```
STEP 1
↓
Kiểm tra 18 empty labels
↓
STEP 2
↓
Kiểm tra image corrupted
↓
STEP 3
↓
Kiểm tra annotation bằng Python validator
↓
STEP 4
↓
Phân tích bounding box statistics
↓
STEP 5
↓
Kiểm tra duplicate / sequence
↓
STEP 6
↓
Visualize dataset
↓
STEP 7
↓
Thiết kế train / val / test split
↓
STEP 8
↓
Tạo dataset version
↓
STEP 9
↓
Train baseline
↓
STEP 10
↓
Evaluation
```

---

 # 24\. Điều cần nhớ khi bắt đầu ngày làm việc tiếp theo

 Đọc file này xong, hãy nhớ:

 > **Chúng ta đang ở giai đoạn Data Discovery / Data Engineering, chưa phải giai đoạn tối ưu model.**

 Dataset hiện tại có dấu hiệu:

```
3216 images
51 classes
8341 annotations
18 empty labels
class imbalance 305x
possible sequential frames
```

 Vì vậy việc quan trọng nhất hiện tại là **hiểu dataset và đảm bảo validation experiment đúng**.

 Đừng vội:

```
train YOLOv8m
tune epochs
tune batch size
tune confidence
```

 trước khi xử lý các vấn đề dữ liệu.

---

 # 25\. Một nguyên tắc cho toàn bộ project

 Mỗi quyết định sau này nên có:

```
Question
   ↓
Evidence
   ↓
Experiment
   ↓
Result
   ↓
Decision
   ↓
Documentation
```

 Ví dụ:

```
Question:
Dataset có imbalance không?

Evidence:
Class 10 = 1220
Class 45 = 4

Result:
Max/Min = 305

Decision:
Cần đánh giá imbalance trước khi training chính thức.

Documentation:
docs/01_problem_discovery.md
```

 Không nên:

```
"Tôi nghĩ dataset bị imbalance"
```

 mà phải:

```
"Dataset có imbalance mạnh vì max/min annotation ratio = 305,
dựa trên thống kê trực tiếp từ 3216 label files."
```

---

 # 26\. File quan trọng trong project

 Hiện tại:

```
Vietnam-traffic-sign-detection/
│
├── data/
│   ├── images/
│   ├── labels/
│   └── classes.txt
│
├── docs/
│
└── .gitignore
```

 `data/` đang ignored.

 `docs/` đang được dùng để lưu documentation/diary.

 Một file quan trọng cần có:

```
docs/01_problem_discovery.md
```

 File này nên mô tả vấn đề, stakeholder, success metrics và các phát hiện ban đầu.

 README này có vai trò khác:

```
README này
    ↓
"Ngày mai cần nhớ những gì?"
```

 Trong khi:

```
docs/01_problem_discovery.md
    ↓
"Project đang giải quyết vấn đề gì?"
```

---

 # 27\. Khi tiếp tục, hãy bắt đầu bằng câu này

 Khi mở lại project, hãy tự nhắc:

 > **"Mình đã audit sơ bộ dataset. Bây giờ chưa train model. Tiếp tục kiểm tra 18 empty labels, corrupted images, annotation schema và sequence/duplicate trước khi tạo train/val/test split."**

 Sau đó mới tiếp tục các command kiểm tra.

---

 # 28\. Trạng thái checkpoint

```
PROJECT STATUS
==============

Problem discovery        [DONE - sơ bộ]
Dataset inventory        [DONE]
Image/label matching     [DONE]
Class count              [DONE]
Class ID audit           [DONE]
Class distribution       [DONE]
Imbalance discovery      [DONE]
Empty label discovery    [DONE]
Coordinate range audit   [DONE - sơ bộ]

Empty label investigation [TODO]
Corrupted image check     [TODO]
Annotation validator      [TODO]
BBox analysis             [TODO]
Duplicate detection       [TODO]
Sequence analysis         [TODO]
Visualization             [TODO]
Train/Val/Test split      [TODO]
Dataset versioning        [TODO]
Baseline training         [TODO]
Evaluation                [TODO]
Production pipeline       [TODO]
API                       [TODO]
ONNX                      [TODO]
Monitoring                [TODO]
```

---

 # 29\. Quan trọng nhất

 **Không xóa dataset.**

 **Không sửa hàng loạt annotation bằng tay.**

 **Không train lại chỉ vì muốn có một con số mAP đẹp.**

 Trước tiên phải hiểu:

```
DATA
 ↓
QUALITY
 ↓
SPLIT
 ↓
EXPERIMENT
 ↓
MODEL
```

 Nếu DATA hoặc SPLIT sai thì metric MODEL không đáng tin.

---

 ## END OF CHECKPOINT

 Ngày tiếp theo hãy mở file này trước.

 Sau đó tiếp tục từ:

```
18 empty label files
        ↓
visual inspection
        ↓
Python dataset validator
        ↓
bbox analysis
        ↓
sequence / duplicate analysis
        ↓
proper train/val/test split
```

 Bạn có thể lưu nguyên nội dung trên thành:

```
nano ~/Documents/Vietnam-traffic-sign-detection-README.md
```

 hoặc nếu muốn dùng VS Code:

```
code ~/Documents/Vietnam-traffic-sign-detection-README.md
```

 **Điểm quan trọng:** README này đã cố tình ghi cả những **command từng chạy sai / kết quả từng bị hiểu nhầm**, để ngày mai bạn không lặp lại lỗi và biết vì sao con số cuối cùng `8341 annotations` mới là con số chúng ta đang tin cậy.

 Ngày mai chỉ cần mở README này, đọc mục **27 → 28**, bạn sẽ biết chính xác đang dừng ở đâu và phải làm gì tiếp theo.
