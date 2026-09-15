# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Nguyễn Hữu Dũng (làm cá nhân)
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | không log lại lúc gán — không có mốc thời gian đáng tin cậy để điền (mtime file chỉ phản ánh thời điểm export, không phải thời điểm bắt đầu/kết thúc gán) |
| Thời gian gán `clip_01` | không log lại lúc gán, lý do như trên |
| Số track đã vẽ trong `clip_01` | 8 (ID 1-8) |
| Số keyframe trung bình mỗi track | không trích xuất được từ `gt.txt` — export MOT 1.1 ghi một dòng cho **mọi** frame (kể cả frame interpolated), không phân biệt keyframe thật với frame nội suy. Số keyframe thật chỉ xem được trực tiếp trong CVAT job, không có trong file đã nộp. |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Track 5 và track 6 cắt nhau ở giữa clip (frame ~87-113).** Hai xe đi sát nhau, có lúc bbox chồng lấp đáng kể. Xử lý: đặt keyframe dày hơn ngay trước/trong/sau đoạn chồng lấp, xác nhận bằng mắt ở nửa đường giữa hai keyframe rằng mỗi bbox vẫn bám đúng xe của nó trước khi lưu. Kết quả: `IDSW = 0` khi chấm với gold — không có ID switch nào, kể cả ở đoạn khó nhất này (trong khi cả hai tracker tự động ByteTrack và BoT-SORT+ReID đều bị đổi ID ở chính đoạn này, xem mục 4-5).
2. **Xe mới xuất hiện, bbox rất nhỏ (track 5 tại frame 73, track 6 tại frame 82).** Quyết định mở track ngay khi nhận ra hình chữ nhật đặc trưng của xe, dù bbox chỉ 13-23px. So với gold, hai track này mở sớm hơn 6-19 frame — ghi lại thành ca mơ hồ cụ thể trong `GUIDELINE_MINI.md` (Ca 1, Ca 2) và trong `reports/review_partner.md`.
3. **Track 3 gần như đứng yên hai đoạn dài (frame 1-15 và 156-170) — `check_mot_labels.py` cảnh báo nghi quên bấm `outside`.** Xử lý: xem lại bằng mắt, xác nhận đây là xe đang đỗ thật (đúng luật "xe đang đỗ vẫn cần track" trong `GUIDE.md`), không sửa. Kết quả: đoạn này không xuất hiện trong bất kỳ danh sách lỗi nào của `outputs/eval_vs_gold.json`.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (chỉ nhìn ID): không thấy số ID nhấp nháy/đổi bất thường trong 8 track của `clip_01`; xác nhận lại sau khi có gold bằng `IDSW = 0`.
- Lượt 2 (frame đầu/cuối từng track): phát hiện phân vân ở frame đầu của track 5 và track 6 (bbox rất nhỏ lúc mới xuất hiện) — ghi lại làm ca mơ hồ thay vì tự quyết theo cảm tính.
- Lượt 3 (giữa mỗi keyframe xa nhau): xem bằng mắt trước khi export, không thấy lệch rõ; độ khít thực tế thấp hơn kỳ vọng (`LocA 0.882`, `MOTP 0.875`) chỉ lộ ra sau khi có gold, tập trung ở 7 frame quanh đoạn track 4/5/6 cắt nhau (frame 54, 83, 103-112, 168).

**Kiểm chéo với:** không có — bài làm **cá nhân**, không có lab partner để gán độc lập cùng clip theo đúng quy trình `GUIDE.md` mục 4. Đây là giới hạn thật của bài nộp, không phải bỏ sót; chi tiết và lý do được ghi rõ ở đầu `reports/review_partner.md`, thay bằng self-QC ba lượt (ở trên) cộng danh sách phát hiện khách quan từ `evaluate_tracking.py` đối chiếu gold.

Số lỗi bạn tìm được trong bản của bạn ấy: N/A (không có bạn cùng nhóm).
Số lỗi bạn ấy tìm được trong bản của bạn: N/A (không có bạn cùng nhóm).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Không có ca "hai người quyết khác nhau" thật vì không có người thứ hai. Luật còn thiếu mà tự phát hiện được (đã bổ sung vào `GUIDELINE_MINI.md` mục 5): (a) chưa có ngưỡng pixel/frame cụ thể cho "xe đủ rõ để mở track" khi mới xuất hiện — đây là nguồn gốc của cả 3 finding trong `reports/review_partner.md`; (b) chưa có quy trình đặt keyframe cụ thể quanh lúc hai xe cắt nhau.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `8815e59a55fb5d72257b6bdc8ec18be91eb015fb45b8d1303333b1f8314526be` |
| Thời điểm khóa | 2026-09-15T10:19:45Z (UTC) |
| Số row / frame / track trước khi mở reference | 610 dòng · 190 frame · 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.819 | 0.801 | 0.838 | 0.882 | 0.947 | 0.890 | 0.875 | 50 | 13 | 0 |
| Sau rework | 0.819 | 0.801 | 0.838 | 0.882 | 0.947 | 0.890 | 0.875 | 50 | 13 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Bản pre-gold và bản sau rework **giống hệt nhau về mặt số liệu** vì cổng đã đạt
ngay từ lần export đầu tiên (không cần vòng sửa nhãn nào để qua cổng). Theo mức
chất lượng trong `RUBRIC.md`: `HOTA 0.819`, `IDF1 0.947`, `LocA 0.882` đều đạt
ngưỡng "Xuất sắc" (>= 0.80 / >= 0.90 / >= 0.80); chỉ `MOTA 0.890` còn thiếu 0.01
so với ngưỡng "Xuất sắc" (>= 0.90) — nằm ở tầng "Đạt" cho tiêu chí này, sát tầng
"Xuất sắc".

Vì không phải rework để qua cổng, phần "sửa gì" dưới đây liệt kê các **finding
nhẹ, không bắt buộc** mà `evaluate_tracking.py` tìm được so với gold (đã ghi
đầy đủ hơn trong `reports/review_partner.md`), không sửa trực tiếp vì không có
lượt gán lại trong phiên này:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo trước khi track "chính thức" xuất hiện theo gold | 82-100 | 6 | Chưa sửa (needs-review) — mở track sớm hơn gold vì đã thấy hình dạng xe rõ; ghi thành Ca 2 trong `GUIDELINE_MINI.md` để chuẩn hóa ngưỡng cho lần gán sau |
| Bbox treo trước khi track "chính thức" xuất hiện theo gold | 73-78 | 5 | Chưa sửa (needs-review) — tương tự, ghi thành Ca 1 trong `GUIDELINE_MINI.md` |
| Bbox trôi (IoU 0.51-0.58 so với gold) quanh đoạn hai xe cắt nhau | 54, 83, 103-112, 168 | 4, 5, 6, 8 | Chưa sửa (needs-review) — không bắt buộc vì `MOTP` đã qua cổng; đã bổ sung quy trình đặt keyframe quanh giao cắt vào `GUIDELINE_MINI.md` mục 5 cho lần sau |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.7 / 8.4.145 / 2.14.0 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control) và `configs/trackers/botsort-reid.yaml` (treatment, BoT-SORT + ReID) |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / [2, 5, 7] (car, bus, truck — COCO) |
| device | cpu (không có GPU CUDA khả dụng trên máy chạy notebook) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.819 | 0.801 | 0.838 | 0.882 | 0.947 | 0.890 | 0.875 | 50 | 13 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.748 | 0.695 | 0.808 | 0.875 | 0.888 | 0.775 | 0.863 | 81 | 53 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Ngược lại với cảnh báo trong `GUIDE.md`: ở nhãn của tôi, `MOTA (0.890)` **thấp
hơn** `IDF1 (0.947)`, không phải trường hợp "MOTA cao mà IDF1 thấp". Điều này
nhất quán với `IDSW = 0` — không có lỗi identity nào để MOTA hay IDF1 phải phạt,
nên khoảng cách giữa hai số chỉ đến từ FP/FN (50 FP, 13 FN trên 573 bbox gold),
tức là sai số hình học/coverage (ví dụ 2 track mở sớm hơn gold ở mục 3), không
phải sai số định danh. Tuy vậy hiện tượng "MOTA cao, IDF1 thấp" vẫn rõ ràng khi
so sánh **ByteTrack control vs gold**: `MOTA 0.749` nhưng `IDF1 0.875` — đúng
kiểu lệch mà `GUIDE.md` mô tả, dù không quá cực đoan ở đây. Lý do MOTA không
phạt nặng lỗi ID: công thức MOTA đếm `FP + FN + IDSW` trên tổng số bbox gold,
và mỗi lần ID switch chỉ tính là **một** lỗi bất kể track đó dài bao nhiêu frame
— một track 95 frame (như track gold 4 trong ByteTrack, bị tách thành ID 14 và
15 ở frame 59) chỉ đóng góp 1 vào tử số của MOTA, trong khi IDF1 và AssA đo trên
**toàn bộ quãng đời** của track nên phạt đúng cả nửa track bị gán sai ID. Đó là
lý do IDF1/AssA là chỉ số đáng tin hơn MOTA khi đánh giá riêng phần giữ ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

Cả hai đều có `IDSW = 2` — không đổi. Nhưng `IDF1` tăng từ `0.875` (ByteTrack)
lên `0.900` (ReID), và `AssA` tăng từ `0.776` lên `0.820`. Số lượng ID switch
bằng nhau nhưng IDF1/AssA của treatment vẫn cao hơn, vì đó là **track khác nhau
bị switch**, không phải "ReID sửa được lỗi của ByteTrack": ByteTrack đổi ID ở
track gold 4 (frame 59, ID 14→15) và track gold 5 (frame 94, ID 23→32); ReID lại
đổi ID ở track gold 5 (frame 87, ID 17→18) và track gold 6 (frame 113, ID 24→31).
Cả hai tracker đều thất bại đúng ở đoạn track 4/5/6 đi sát và cắt nhau (frame
~87-113) — đây là chỗ nhãn tay xử lý tốt (mục 1, câu 3) nhưng cả hai hệ thống
tự động đều tách ID. IDF1 cao hơn ở treatment nhiều khả năng đến từ việc bbox
khít hơn ở các đoạn còn lại (`LocA 0.872` so với `0.846`) làm việc ghép cặp với
gold ổn định hơn, chứ không phải vì appearance embedding "nhận ra" xe đúng hơn
tại đúng frame cắt nhau. Vì ByteTrack (Kalman+IoU thuần) và BoT-SORT+ReID
(motion+IoU+appearance, khác cả tracker implementation lẫn buffer/threshold) là
hai hệ thống khác nhau ở nhiều chỗ, không chỉ khác ở có/không có ReID, nên
không thể kết luận riêng ReID là nguyên nhân của khoảng cải thiện IDF1 0.025 —
đây là system comparison, không phải ablation.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA` tăng từ `0.649` (ByteTrack) lên `0.711` (ReID); `FN` giảm mạnh từ 54
xuống 26, còn `FP` tăng nhẹ từ 88 lên 91. Vì detector weights, conf, IoU, imgsz
và classes giống hệt nhau giữa hai run, phần lớn khoảng cách FP không đến từ
detector mà từ **một lỗi detector-level lặp lại giống nhau ở cả hai**: cả
ByteTrack (ID 10, frame 17-116, 42 frame) lẫn ReID (ID 7, frame 16-116, 43
frame) đều giữ một track gần như đứng yên tuyệt đối tại toạ độ ~(490-498,
211-214), kích thước ~(96-106)×(56-60)px, confidence thấp và dao động (0.25-
0.49) — đặc điểm điển hình của **false positive tĩnh** (vật cố định ven đường
bị nhận nhầm là xe), không phải lỗi association. Vì bbox này gần như trùng khít
giữa hai tracker, nó là lỗi của YOLO26n ở bước detect, không phải của
ByteTrack/BoT-SORT ở bước ghép ID. Phần FN còn lại (nhất là ở ByteTrack: 54 so
với 26 của ReID) tập trung ở các track gold 5, 6, 8 chỉ được phủ 61-77% quãng
đời (`outputs/eval_bytetrack_vs_gold.json`, mục `partially_covered_gt_tracks`)
— đây có thể là lỗi association (track bị mất dấu giữa chừng rồi không nối lại
được) hơn là detector hoàn toàn bỏ sót, vì cùng detector input mà ReID phủ được
nhiều hơn ở đúng những track này.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Track gold 5 (frame 79-138, xe đi từ phải sang trái, dần bị xe khác che một
phần): nhãn tay giữ đúng **một ID (5)** suốt 67 frame liên tục (frame 73-139).
ReID treatment tách xe này thành **hai ID (17 rồi 18)**, đổi tại frame 87 —
đúng đoạn track 5 và track 6 đi sát/cắt nhau (xem bbox tại frame 91: gold ID 5
ở (639,266,56,27) rất gần track 4/6 khác trong khung). Tôi đúng vì đã coi đây
là occlusion ngắn giữa các xe cùng chiều, đặt keyframe đủ dày quanh đoạn chồng
lấp và giữ nguyên ID theo đúng luật trong `GUIDELINE_MINI.md`; ReID sai vì
appearance cue không đủ phân biệt hai xe cùng loại, cùng góc nhìn, cùng lúc bị
che một phần bởi nhau, nên motion+IoU (đã bị nhiễu bởi chồng lấp) lẫn appearance
đều không đủ tin cậy để giữ đúng track qua đoạn đó.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID (và cả ByteTrack) liên tục báo một bbox tại toạ độ cố định ~(490-498, 211-
214) từ frame 16 đến 116 (ID 7 ở output ReID) — không khớp với bất kỳ track nào
trong gold lẫn trong nhãn của tôi. Ban đầu finding này khiến tôi xem lại
annotation ở khu vực đó (x≈490-600, y≈210-270) để chắc chắn không có xe bốn
bánh nào tôi bỏ sót thật. Sau khi đối chiếu tọa độ qua nhiều frame (bbox hầu
như không di chuyển, kích thước ổn định, confidence thấp và dao động 0.25-0.49
thay vì tăng dần như một xe đang tiến lại), kết luận đây là **false positive
tĩnh của detector** (khả năng cao là một vật cố định ven đường, không phải xe
bốn bánh), không phải lỗi bỏ sót trong nhãn tay — evidence khớp với mô tả trong
`docs/day3-reid-theory.md`/notebook về loại lỗi "track đứng im" cần được nhận
ra là model sai, không phải người gán thiếu.

## 6. Nếu phải gán thêm 10 clip nữa

Trong `GUIDELINE_MINI.md`, tôi sẽ cụ thể hóa ngưỡng "xe đủ rõ để mở track" thành
một con số đo được (đề xuất: cạnh ngắn nhất của bbox >= 20px và hình dạng ổn
định trong >= 2 frame liên tiếp) thay vì để cảm tính — bằng chứng là hai track
(ID 5, ID 6) trong `clip_01` mở sớm hơn gold 6-19 frame chỉ vì ngưỡng này chưa
được viết thành số. Tôi cũng sẽ viết rõ quy trình đặt keyframe quanh lúc hai xe
cắt nhau (tối thiểu 1 keyframe ngay trước, 1 ở giữa lúc chồng lấp lớn nhất, 1
ngay sau khi tách) vì đây là đúng chỗ cả hai tracker tự động đều tách nhầm ID
mà nhãn tay chỉ tránh được nhờ có đủ keyframe.

Về quy trình làm việc: sẽ ưu tiên tìm một người kiểm chéo thật trước khi khóa
pre-gold — bài này làm cá nhân nên phần "Kiểm chéo" chỉ thay được bằng self-QC
ba lượt và đối chiếu gold sau khi mở khóa (xem `reports/review_partner.md`),
không phát hiện được góc nhìn khác biệt thật sự như một reviewer thứ hai độc
lập sẽ mang lại — nhất là ở đúng loại quyết định chủ quan như "khi nào một xe
đủ rõ để mở track", nơi hai người có khả năng cao sẽ chọn khác nhau.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md` (self-QC thay thế peer review thật — xem giới hạn ghi ở đầu file)
- [x] `reports/REPORT.md` (file này)
