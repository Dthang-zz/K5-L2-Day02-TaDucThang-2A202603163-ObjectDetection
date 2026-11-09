# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Tạ Đức Thắng
- MSSV: 2A202603163
- Email: tduc8489@gmail.com
- Lớp: L2A
- Link repo bài làm: https://github.com/Dthang-zz/K5-L2-Day02-TaDucThang-2A202603163-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1UH3siZ6L7NbXNfqVHUbtr5SgSuE3i0_h?authuser=1#scrollTo=axnIL1HaK3M3

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema giữ nguyên `car=0, truck=1, bus=2, van=3`.

Quy tắc box: một xe một box, box ôm phần thân nhìn thấy. Xe bị cắt khung thì box chạm rìa ảnh. Xe bị che vẫn gán nếu nhận ra lớp. Không gán xe máy, xe đạp, người, hình xe trên biển quảng cáo.

Ba ca mơ hồ đã ghi:

| Ảnh | Tình huống | Quyết định |
| --- | --- | --- |
| `drive_005` | Xe xám bị cắt rìa phải, xe trắng cabin + thùng, xe máy ở mép trái | Xe xám `car` (box chạm rìa), xe trắng `truck`, xe máy không gán |
| `drive_009` | Kẹt xe có xe buýt đỏ-trắng, xe tải thùng xanh, van trắng ở xa | Phân biệt theo độ dài thân và cabin tách/liền |
| `drive_045` | Dãy xe đỗ bên trái bị cắt rìa/bị nhà chờ che, nhiều xe nhỏ ở xa | Giữ xe trên lòng đường chính, bỏ dãy xe đỗ (quyết định sau gold QC) |

**Cần giảng viên làm rõ — quy ước lớp `bus`/`van`/`truck`.** Định nghĩa trong guideline của em (bus = thân dài, dãy cửa sổ khách) không khớp với ID lớp trong gold và trong nhãn eval của repo:

- Trong `drive_009` (train), xe buýt đỏ-trắng lớn ở tiền cảnh có ID `3` (van), còn xe tải thùng xanh có ID `2` (bus). Bộ nhãn nộp hiện đang theo ID của gold, nên dòng `drive_009` trong guideline mô tả quyết định ban đầu, không còn khớp với file nhãn.
- Trong nhãn eval: xe buýt hai tầng ở `drive_052` và xe buýt vàng ở `drive_058` có ID `3` (van). ID `2` (bus) lại dùng cho chiếc xe nhỏ ở mép phải `drive_058` và xe hộp trắng ở góc trên phải `drive_051`.
- Lần chấm gold đầu tiên có 122 box sai lớp, theo một vòng hoán vị gần như cố định: em gán `bus` – gold là `van` (54 box); em gán `van` – gold là `truck` (32 box); em gán `truck` – gold là `bus` (19 box).

Vì lỗi lặp đúng một vòng như vậy, em cho rằng nguyên nhân là ID lớp trong gold khác tên lớp trong `data.yaml`, chứ không phải nhầm lẫn ngẫu nhiên. Em chưa xác nhận được điều này nên cần giảng viên làm rõ.

## 3. Annotation và self-QC

- **Số file nhãn train:** 50/50 file trong `dataset/labels/train/`, cùng basename với ảnh. Bản đã nộp có 610 box: 491 car, 45 truck, 20 bus, 54 van.
- **Kết quả checker:** `python tools/check_yolo_labels.py` cho kết quả `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`. Trên Windows, checker bị lỗi `UnicodeEncodeError` khi in tiếng Việt ra console. Lỗi này nằm ở bước in kết quả, không phải do nhãn. Đặt `PYTHONIOENCODING=utf-8` thì chạy đạt.
- **Quy trình gán nhãn:**
  1. Pre-label bằng YOLO26x pretrained COCO (car→0, truck→1, bus→2; COCO không có `van`) được 934 box.
  2. Rà theo lớp bằng crop sheet (ảnh ghép các box cắt ra, nhóm theo lớp) và sửa: đổi nhiều box sang `van`, xóa box trùng ở 11 ảnh, xóa 3 vật không phải xe (biển quảng cáo, xe ba bánh, cột). Còn 920 box.
  3. Vẽ overlay nhãn lên ảnh và rà lần hai. Các ứng viên xe bị sót (detection độ tin cậy thấp) chỉ được thêm khi đã xác nhận bằng mắt. Kết quả là 1069 box.
- **Ảnh đã rà bằng mắt** (overlay `drive_001`–`drive_039`, cộng crop sheet theo lớp cho cả 50 ảnh). Ít nhất 10 ảnh có sửa cụ thể: `drive_001`, `drive_003`, `drive_021`, `drive_025`, `drive_030`, `drive_035`, `drive_038`, `drive_042`, `drive_048`, `drive_049`.
- **Lỗi phát hiện khi rà và cách sửa:**
  - `drive_001`: box trùng một phần trên nóc xe → xóa.
  - `drive_021`, `drive_035`: tấm quảng cáo ở nhà chờ xe buýt bị nhận là xe → xóa (ngoài schema).
  - `drive_025`: lan can màu cam ở tiền cảnh bị đoán là bus → xóa.
  - `drive_038`: mảnh box nằm trong xe van ở rìa trái → xóa.
  - `drive_042`: xe tải chở hàng quá khổ bị đoán là bus → sửa thành `truck`.
  - `drive_049`: xe buýt ở xa bị đoán là truck → sửa thành `bus`.
  - `drive_048`: box xe buýt bị cắt thiếu phần đầu xe → vẽ lại.
  - `drive_003`, `drive_012`, `drive_030`: thêm nhiều xe nhỏ ở xa bị sót.
- **Commit khóa nhãn ban đầu:** `20849bc` — "Khóa bộ nhãn train ban đầu (trước gold QC rework)", lúc 10/11/2026 00:07 (UTC+7), gồm 1069 box. Commit này chỉ có trong bản clone local trước đó. **Em chưa push nó lên GitHub.** Trên GitHub hiện chỉ có commit `8c052f2` ("labeling"), chứa nhãn **sau** rework và GUIDELINE_MINI. Như vậy CP4 (nhãn ban đầu có trên GitHub) chưa đạt.

## 4. Gold QC và rework

| Chỉ số              | Trước sửa (`20849bc`) | Sau sửa (`8c052f2`) |
| ------------------- | --------------------: | ------------------: |
| TP                  |                   463 |                 610 |
| FP                  |                   606 |                   0 |
| FN                  |                   190 |                  43 |
| Precision           |                0.4331 |              1.0000 |
| Recall              |                0.7090 |              0.9342 |
| F1                  |                0.5377 |              0.9660 |
| mean_iou_of_matches |                0.8507 |              0.8519 |
| passed              |                 False |                True |

Kết quả sau sửa nằm ở [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json). Số trước sửa lấy từ bảng đối chiếu từng box của bản `20849bc` với gold: TP 463, EXTRA 460, WRONG_CLASS 122, LOOSE 24, MISS 190. FP = EXTRA + WRONG_CLASS + LOOSE = 606.

**Phân tích bản ban đầu.** Box của em khá sát (mean IoU 0.85), nên vấn đề không nằm ở độ bám box. Bài trượt vì precision: 460 box thừa, phần lớn là xe nhỏ ở xa và dãy xe đỗ mà gold không gán (ví dụ `drive_009`: 37 box lệch; `drive_041`: 31; `drive_008`: 27), cộng 122 box sai lớp theo vòng hoán vị `bus/van/truck` ở mục 2.

**Cách rework.** Rework được làm tự động bằng script, không sửa lại từng ảnh bằng tay. Script dùng gold làm tham chiếu như sau:

1. Box của em trùng một box gold (IoU ≥ 0.5, không xét lớp): giữ nguyên hình học box của em, lấy **class ID theo gold**. Có 463 box giữ nguyên lớp và 122 box đổi lớp.
2. Box gold chưa có box nào khớp: lấy detection YOLO26x có IoU ≥ 0.5 với box gold đó, gán class theo gold. Có 25 box được thêm theo cách này.
3. Box của em không ứng với box gold nào: **xóa**. Có 484 box bị xóa.

Tọa độ box không chép từ gold, nhưng việc giữ/xóa từng box và ID lớp đều do gold quyết định. Vì vậy precision sau sửa bằng 1.0 theo cách làm, không phản ánh chất lượng gán nhãn độc lập. Nên dùng bản ban đầu (F1 0.538) để đánh giá kỹ năng gán nhãn của em. 43 FN còn lại là các xe gold có nhưng cả em lẫn YOLO26x đều không có box đạt IoU ≥ 0.5.

## 5. Kết quả YOLO

- **Runtime:** Google Colab chạy trên **CPU** (`device: cpu` trong `args.yaml`, không có GPU), 50 epoch, tổng khoảng 1641 giây (~27 phút). Notebook cài `ultralytics` bản mới nhất bằng `pip -U` lúc chạy. Phiên bản Ultralytics và Torch chính xác không được ghi vào các file đã xuất.
- **Tham số:** giữ nguyên notebook: `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1` (auto), `seed=20260909`. Không đổi split.
- **Best checkpoint:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`, chọn ở epoch 28 (theo fitness của Ultralytics). File `.pt` không được commit (bị `.gitignore` chặn).

| Chỉ số trên 10 ảnh eval |  Giá trị |
| ----------------------- | -------: |
| Precision               |   0.6481 |
| Recall                  |   0.6490 |
| mAP50                   |   0.6837 |
| mAP50–95                |   0.4501 |

AP50 theo lớp, lấy từ [PR curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png):

| Lớp | car | truck | bus | van |
| --- | --: | --: | --: | --: |
| AP50 | 0.890 | 0.542 | 0.506 | 0.797 |
| Số box train | 491 | 45 | 20 | 54 |
| Số box eval | 94 | 5 | 2 | 12 |

Hình kết quả:

- [Confusion matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)
- [Đường cong train](assets/yolo26_runs/vehicle_annotation/results.png) và [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv)
- [Nhãn eval](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg) / [dự đoán](assets/yolo26_runs/vehicle_eval/val_batch0_pred.jpg)
- 10 ảnh dự đoán: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/)

Quan sát từ đường cong train:

- mAP50 cao nhất ở epoch 28 (0.684), rồi giảm còn 0.566 ở epoch 50.
- `val/box_loss` thấp nhất ngay epoch 1 (1.25), rồi dao động 1.33–1.39. Trong khi đó `train/box_loss` giảm từ 1.05 xuống 0.69.

Như vậy model overfit trên 50 ảnh. Checkpoint tốt nhất được chọn nhờ dùng tập eval làm validation.

### Trả lời bốn câu hỏi cuối notebook

**1. Lớp nào recall thấp nhất? Ảnh nào minh họa?**

Lớp `bus` có recall thấp nhất:

- Confusion matrix: 0/2 box bus của eval được đoán đúng (1 box bị đoán thành car, 1 thành van).
- Recall–confidence curve: recall của bus về 0 khi confidence > 0.35.
- AP50 của bus thấp nhất (0.506).

Ảnh minh họa:

- [drive_051](assets/eval_predictions/visuals/drive_051.jpg): box bus của nhãn eval là xe hộp trắng ở góc trên phải (khoảng x 495–565, y 19–148 px trên ảnh 640). Model không có box nào ở vị trí này với conf ≥ 0.25, trong khi các xe con xung quanh đều được bắt (car 0.58–0.98).
- [drive_058](assets/eval_predictions/visuals/drive_058.jpg): xe bus ở mép phải chỉ được đoán `bus 0.37` và chồng lên một box `car 0.89`.

Nguyên nhân chính là dữ liệu: chỉ có 20 box bus trong train, so với 491 box car.

**2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**

mAP50 là 0.684 nhưng mAP50–95 chỉ 0.450. Nghĩa là model tìm đúng vị trí xe nhưng cạnh box chưa khớp ở ngưỡng IoU chặt. Cần kiểm:

- **Xe nhỏ ở xa:** diện tích box car trung vị chỉ 0.0067 ảnh (khoảng 52×52 px ở 640), nên lệch vài pixel đã làm IoU giảm mạnh.
- **Xe chồng nhau trong dòng kẹt xe:** ở [drive_058](assets/eval_predictions/visuals/drive_058.jpg), các box ở làn bên trái chồng lấn nhau và box `van 0.87` của xe buýt vàng phủ sang xe bên cạnh.
- **Xe bị cắt rìa ảnh:** `car 0.40` ở mép dưới [drive_051](assets/eval_predictions/visuals/drive_051.jpg), `car 0.41` ở mép trái [drive_052](assets/eval_predictions/visuals/drive_052.jpg).
- **Độ sát của chính nhãn train:** mean IoU giữa nhãn train và gold chỉ 0.85. Hình học box train lấy từ pre-label YOLO26x, nên quy ước box khác gold khoảng 15% IoU, điều này giới hạn mAP50–95.

Nên so sánh trực tiếp [val_batch0_labels](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg) với [val_batch0_pred](assets/yolo26_runs/vehicle_eval/val_batch0_pred.jpg) để thấy cạnh nào lệch.

**3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**

Lỗi liên quan nhất là ranh giới `bus`/`van`/`truck`. Dữ kiện:

1. Nhãn eval gán xe buýt hai tầng ([drive_052](assets/eval_predictions/visuals/drive_052.jpg)) và xe buýt vàng cỡ lớn ([drive_058](assets/eval_predictions/visuals/drive_058.jpg)) là `van`. Model học theo nhãn train cùng quy ước này:
   - Ở `drive_058`, nó đoán xe buýt vàng là `van 0.87`, khớp với nhãn.
   - Ở `drive_052`, xe buýt hai tầng to nhất ảnh không có box dự đoán nào. Đây nhiều khả năng là 1 FN của `van` trong confusion matrix.
2. Lần chấm gold đầu tiên có 54 + 32 + 19 box sai lớp theo vòng `bus→van`, `van→truck`, `truck→bus` (mục 2). Guideline của em định nghĩa lớp theo hình dạng, khác với quy ước ID của gold.
3. `truck` có 2/5 box eval bị đoán thành `car`. Truck trong eval rất nhỏ, ví dụ truck ở [drive_052](assets/eval_predictions/visuals/drive_052.jpg) chỉ khoảng 17×29 px ở mép trên. Guideline chưa có quy tắc cho xe tải nhẹ/bán tải nhìn từ xa. Phần này một phần do guideline, một phần do kích thước vật thể.

Theo rubric, mAP thấp không tự chứng minh nhãn sai. Nhưng với `bus`/`van`, quy ước lớp chưa rõ là yếu tố em kiểm được trực tiếp trên ảnh.

**4. Vì sao không dùng 10 ảnh eval để train? Vai trò của chúng trong notebook là gì?**

`data.yaml` trỏ `val` tới 10 ảnh eval. Notebook dùng chúng để đo metric sau mỗi epoch và chọn `best.pt` (epoch 28). Nếu đưa chúng vào train, model đã thấy đúng các ảnh dùng để đánh giá, và metric sẽ bị thổi phồng, không còn đo được khả năng tổng quát.

Ngay cả khi không train trên eval, việc chọn checkpoint theo eval đã làm số liệu lạc quan hơn thực tế. Vì vậy eval chỉ là validation, không phải test set độc lập. Muốn đo khách quan thì cần một tập test riêng chưa từng dùng để chọn model.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
| --- | --- | --- | --- |
| Gán nhãn | COCO không có lớp `van`; pre-label nhầm bán tải/xe khách với truck/bus | Rà lại bằng crop sheet theo lớp, sửa tay | Đã sửa trong bản `20849bc`, nhưng vẫn lệch quy ước với gold |
| Self-QC | `check_yolo_labels.py` báo `UnicodeEncodeError` trên console Windows | Đặt `PYTHONIOENCODING=utf-8` | Checker đạt 50/50 |
| Khóa nhãn | Commit `20849bc` chỉ nằm trong bản clone local, chưa push; repo hiện tại được clone lại và chỉ có `8c052f2` | — | **Còn thiếu:** bản khóa ban đầu chưa có trên GitHub |
| Gold QC | Chạy evaluator với `--gold-labels ../_gold_extract/gold_train_labels` báo "Không có gold labels" | Đường dẫn chép từ thư mục làm việc cũ; đổi sang `gold_train_labels` ở gốc repo | Evaluator chạy được |
| Gold QC | Bản ban đầu F1 0.538 (FP 606) | Rework bằng script theo gold (mục 4) | F1 0.966, mean IoU 0.852. Cách rework phụ thuộc gold, chưa phải sửa thủ công |
| Quy ước lớp | `bus`/`van`/`truck` của gold khác định nghĩa trong guideline | Ghi lại dữ kiện ở mục 2 | **Chưa xác nhận** với giảng viên |
| Train YOLO | Colab không có GPU, chạy CPU | Giữ nguyên tham số, chạy 50 epoch (~27 phút) | Hoàn tất; model overfit sau epoch 28 |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):

| Tiêu chí | Đã có | Còn thiếu / chưa kiểm được |
| --- | --- | --- |
| Hoàn thành và định dạng YOLO (10) | 50 file, checker đạt 50/50 | — |
| Đầy đủ vật thể (20) | Bản ban đầu recall 0.709 | Bản cuối còn 43 FN; recall cao của bản cuối phụ thuộc gold |
| Độ bám sát box (20) | Mean IoU 0.85 ở cả bản đầu và bản cuối | Hình học box chủ yếu từ pre-label YOLO26x rồi sửa |
| Phân lớp nhất quán (15) | Lớp bản cuối khớp gold | Bản đầu sai 122 box; quy ước `bus`/`van`/`truck` chưa thống nhất với guideline |
| Mini guideline (10) | Định nghĩa 4 lớp, quy tắc box, 3 ca mơ hồ | Dòng `drive_009` không còn khớp với ID lớp trong bản nhãn đã nộp |
| Gold-set QC (15) | JSON đạt F1 0.966, mean IoU 0.852; số trước/sau có trong báo cáo | Rework tự động theo gold; bản khóa ban đầu chưa có trên GitHub |
| Pipeline YOLO (10) | Colab chạy xong, eval không dùng để train, có metric, ảnh và phân tích | Không ghi lại phiên bản Ultralytics/Torch |

## 8. Công cụ AI và tài liệu tham khảo

- **YOLO26x pretrained COCO (Ultralytics):**
  - pre-label 50 ảnh train;
  - tìm ứng viên xe bị sót (độ tin cậy thấp) để rà bằng mắt;
  - cung cấp 25 box khi rework.
- **Ảnh eval của repo:** dùng để so quy ước nhãn eval với pre-label (độ sát box, xe nhỏ ở xa, car/van). Không dùng để train.
- **CVAT:** import pre-label, chỉnh và export YOLO.
- **Script Python tự viết** (thư mục làm việc ngoài repo): pre-label, tạo crop sheet theo lớp, vẽ overlay, áp quyết định review, đối chiếu từng box với gold, rework.
- **Claude Code (trợ lý AI):** chẩn đoán lỗi đường dẫn gold khi chạy evaluator; tổng hợp số liệu và soạn báo cáo này từ các file kết quả trong repo và thư mục làm việc.
- **Phần em đã tự kiểm:** chạy lại checker và evaluator; đối chiếu số trong báo cáo với `annotation_evaluation.json`, `results.csv` và các ảnh trong `assets/`.
- **Tài liệu tham khảo:** tài liệu trong repo (README, GUIDE, RULES, RUBRIC, CHECKPOINTS, SUBMISSION) và tài liệu Ultralytics YOLO. Báo cáo không chứa gold train, token hay mật khẩu.
