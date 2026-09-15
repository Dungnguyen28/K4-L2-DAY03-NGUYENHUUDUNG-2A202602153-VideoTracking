# Peer review — Day 3

> **Giới hạn quan trọng, đọc trước:** bài này làm **cá nhân** (xem `TEAM.md` —
> không có thành viên nào khác được khai báo). GUIDE.md mục 4 yêu cầu hai người
> gán **cùng một clip, độc lập, không xem nhãn của nhau**, rồi đổi file chấm
> chéo bằng `evaluate_tracking.py --mode peer` — bước này cần một người thứ hai
> thật sự và không có cách nào tạo ra một cách trung thực nếu làm một mình.
> Không có bản nhãn độc lập thứ hai của cùng clip nên không thể tạo peer review
> thật. Phần dưới đây thay thế bằng **self-QC ba lượt tua** (đúng quy trình mục
> 3 của `GUIDE.md`, làm trước khi khóa pre-gold) cộng với **danh sách phát hiện
> khách quan từ `evaluate_tracking.py` đối chiếu với gold sau khi mở khóa**
> (không phải từ một reviewer con người). Hai nguồn được ghi rõ nguồn gốc riêng
> ở từng dòng để không đánh đồng với một kiểm chéo ngang hàng thật.
>
> Nếu tiêu chí "Kiểm chéo" (10 điểm, RUBRIC.md) yêu cầu bằng chứng hai người,
> nộp kèm ghi chú này để Lab Coach biết đây là giới hạn thật của bài nộp cá
> nhân, không phải bỏ sót.

| Trường | Giá trị |
| --- | --- |
| Author | Nguyễn Hữu Dũng |
| Reviewer | không có (bài cá nhân, không có lab partner) |
| Pair ID | N/A — solo submission |
| CVAT version | không xác định được từ export (không có version trong `gt.txt`/`labels.txt`); điền tay nếu nhớ |
| Thời điểm review | tự-kiểm ba lượt trước khóa pre-gold; đối chiếu gold sau khi `tools/lock_pre_gold.py` chạy thành công |

## Danh sách finding

Nguồn của mỗi dòng được ghi ở cột cuối cùng thay vì giả định là một reviewer
con người đã tìm ra.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 82-100 | 82-100 | 6 | Bbox trước khi track "chính thức" xuất hiện (theo gold) | Nhãn tay mở track 6 ở frame 82 (bbox 13x33px, rất nhỏ) trong khi gold coi xe "đủ rõ" từ frame 101. Không vi phạm luật lab (đã ghi ở `GUIDELINE_MINI.md` Ca 2) nhưng lệch ngưỡng chủ quan so với gold. Nguồn: `outputs/eval_vs_gold.json`, mục `ghost_pred_tracks`. | Nếu gán lại: chờ bbox đạt cạnh ngắn nhất >= 20px và ổn định 2 frame liên tiếp trước khi mở track. | needs-review (không bắt buộc sửa vì cổng đã đạt; để lại làm bài học cho lần gán tiếp theo) |
| 2 | 73-78 | 73-78 | 5 | Bbox trước khi track "chính thức" xuất hiện (theo gold) | Tương tự #1: track 5 mở ở frame 73 (14x23px), gold nhận từ frame 79. Nguồn: `outputs/eval_vs_gold.json`, mục `ghost_pred_tracks`. | Áp dụng ngưỡng đề xuất ở #1 nếu gán lại. | needs-review |
| 3 | 103-112 | 103-112 | 4, 5, 6 | Bbox trôi (IoU thấp so với gold, 0.51-0.58) quanh đoạn hai xe cắt nhau | Ở đoạn track 4/5/6 đi sát và cắt nhau (khoảng frame 54, 83, 103-112, 168), IoU với gold tụt xuống 0.51-0.58 — bbox vẫn đúng ID nhưng khoanh chưa khít bằng phần còn lại của clip. Nguồn: `outputs/eval_vs_gold.json`, mục `loose_boxes`. | Thêm keyframe dày hơn ngay trước/giữa/sau lúc chồng lấp lớn nhất (đã cập nhật vào `GUIDELINE_MINI.md` mục 5). | needs-review (không bắt buộc vì `MOTP 0.875 >= 0.70` đã qua cổng; ghi lại để cải thiện `LocA` nếu rework) |

## Reviewer checklist

Tự điền dựa trên bằng chứng script (`check_mot_labels.py`, `evaluate_tracking.py`), không phải một reviewer thứ hai xác nhận bằng mắt độc lập.

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | `clip_01`: 8 track (ID 1-8), `clip_02`: 7 track (ID 1-7); `check_mot_labels.py` 0 lỗi cho cả hai |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | `outputs/eval_vs_gold.json`: `IDSW 0` — không có ID switch nào so với gold |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Track 5 (frame 73-139) và track 6 (frame 82-157) mỗi track giữ đúng một ID xuyên suốt đoạn cắt nhau frame 87-113, trong khi cả ByteTrack và BoT-SORT+ReID đều tách hai track này thành 2 ID (xem `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`) |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING (nhẹ) | 2 track (ID 5, ID 6) mở sớm hơn gold 6-19 frame lúc xe còn rất nhỏ — xem finding #1, #2 |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING (nhẹ) | 7 frame có IoU 0.51-0.58 so với gold quanh đoạn giao cắt — xem finding #3 |
| Frame giữa hai keyframe không bị interpolation drift | FINDING (nhẹ) | cùng bằng chứng finding #3 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `check_mot_labels.py` xác nhận frame 1..190 (`clip_01`) và 1..60 (`clip_02`), cột 2 có track_id thật (8 và 7 giá trị khác nhau) |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | xem bảng finding ở trên |

## Self-QC attestation của reviewer

Ba lượt tua theo đúng thứ tự trong `GUIDE.md` mục 3, tự làm trước khi khóa pre-gold (không có gold để đối chiếu ở bước này).

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline (chỉ nhìn số ID) | PASS | Không thấy ID nhấp nháy/đổi số bất thường trong 8 track của `clip_01`; xác nhận lại sau khi có gold bằng `IDSW 0` |
| 2 — endpoint/scope (frame đầu/cuối từng track) | NEEDS-REVIEW | Track 5, 6 có frame đầu là bbox rất nhỏ (Ca 1, Ca 2 trong `GUIDELINE_MINI.md`) — tự thấy phân vân lúc gán, ghi lại thay vì tự sửa theo cảm tính |
| 3 — geometry/interpolation (giữa hai keyframe xa nhất) | PASS (đã xem), có finding nhẹ về sau | Xem lại bằng mắt đoạn giữa các keyframe xa nhất trước khi export; độ khít thực tế (`LocA 0.882`, `MOTP 0.875`) chỉ lộ ra sau khi có gold — không phát hiện được bằng mắt ở bước tự-kiểm |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: track 5 và track 6 (frame 73-157) là ca khó nhất của clip — hai xe đi sát và cắt nhau ở giữa clip, đúng chỗ cả hai tracker tự động (ByteTrack lẫn BoT-SORT+ReID) tách nhầm ID nhưng nhãn tay giữ đúng nhờ theo luật "giữ ID khi occlusion/crossing" trong `GUIDELINE_MINI.md`.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do: track 3 đứng gần như im hai đoạn frame 1-15 và 156-170 (cảnh báo tự động của `check_mot_labels.py`) — xác nhận là xe đang đỗ thật, không phải quên bấm `outside`, theo đúng luật "xe đang đỗ vẫn cần track" trong GUIDE.md.
3. Một rule cần Lab Coach làm rõ: ngưỡng chính xác (số pixel hoặc frame) để coi một xe "đủ rõ để mở track" khi nó vừa xuất hiện, còn rất nhỏ — bài này chênh với gold 6-19 frame ở hai track, cho thấy đây là chỗ guideline của lab còn mơ hồ, không riêng gì người gán này.
