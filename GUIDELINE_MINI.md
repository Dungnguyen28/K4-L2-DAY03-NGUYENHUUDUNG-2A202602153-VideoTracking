# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Hữu Dũng (làm cá nhân)`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm: giữ nguyên mặc định của lab, không thêm ngoại lệ nào khác.
`clip_01` có 8 track hợp lệ (ID 1-8), `clip_02` (warm-up) có 7 track (ID 1-7);
không có track nào bị gán nhầm người đi bộ/xe máy.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | dưới ngưỡng này motion + ngữ cảnh vẫn đủ để chắc chắn đó là cùng một xe, không cần bằng chứng appearance |
| Xe bị che lâu hơn ngưỡng trên | mở track mới, không cố "đoán" giữ ID cũ | giữ ID qua khoảng che dài dễ gán nhầm ID nếu có xe khác đi vào giữa lúc đó — thà tách track còn hơn tạo một ID sai xuyên suốt |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | ra khỏi khung là hết vòng đời quan sát được; không có cách nào xác nhận "chiếc quay lại" chính là chiếc vừa ra, kể cả với ReID |
| Hai xe cắt nhau / chồng lên nhau | mỗi xe giữ ID của nó trước và sau khi cắt nhau; đặt keyframe ngay trước, giữa và ngay sau lúc chồng lấp để tracker/nhãn không "đảo" ID | đây chính là tình huống mô hình (ByteTrack/BoT-SORT) hay nhầm nhất — evidence trong `outputs/eval_reid_vs_gold.json`: track gold 5 và gold 6 đều bị model tách ID quanh lúc hai xe này đi sát nhau ở frame 87-113, trong khi nhãn tay giữ đúng một ID suốt |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh (không phải một chấm mơ hồ); ngưỡng thực tế đã dùng: bbox tối thiểu ~10-15px theo cạnh ngắn nhất và hình dạng đã gợi ý rõ khung xe — xem Ca 1, Ca 2 bên dưới |
| Xe đang đỗ, không di chuyển | vẫn là `vehicle`, vẫn track suốt thời gian nó còn trong khung — không coi bbox đứng yên là lỗi "quên outside" nếu xe thật sự đang đỗ (xem Ca 3) |
| Keyframe đặt dày ở đâu | dày ở đoạn xe rẽ/phanh/đổi kích thước nhanh (ví dụ lúc hai xe cắt nhau ở giữa clip); thưa ở đoạn xe đi thẳng đều tốc độ ổn định |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_01`, frame 73, track ID 5
- Tình huống: Xe xuất hiện ở rìa phải khung hình với bbox rất nhỏ (14×23 px). So với gold (track 5 chỉ bắt đầu ghi nhận từ frame 79, bbox 54×26 px), nhãn của tôi bắt đầu track sớm hơn 6 frame.
- Quyết định: Giữ quyết định bắt đầu sớm (frame 73) vì ở thời điểm đó bbox tuy nhỏ nhưng đã đủ hình chữ nhật đặc trưng của thân xe, không phải nhiễu.
- Lý do: Ưu tiên coverage (không bỏ sót đoạn đầu track) hơn là chờ xe "rõ hẳn"; chấp nhận rủi ro bbox nhỏ kém chính xác hơn là bỏ sót 6 frame đầu đời của track. Đánh đổi này thể hiện trong `outputs/eval_vs_gold.json`: track 5 bị tính là có bbox "trước khi track tham chiếu xuất hiện" (frame 73-78) — không phải lỗi định danh, chỉ là ngưỡng "đủ rõ" khác gold.

### Ca 2
- Clip / frame / ID: `clip_01`, frame 82, track ID 6
- Tình huống: Tương tự Ca 1 — xe mới xuất hiện ở góc phải trên, bbox cực nhỏ (13×33 px), gần như một vệt mờ. Gold ghi nhận track 6 muộn hơn nhiều, từ frame 101 (61×30 px).
- Quyết định: Bắt đầu track từ frame 82 vì đã phân biệt được hình dạng xe (không phải bóng cây hay vệt sáng).
- Lý do: Đây là ca mơ hồ nhất trong clip — chênh lệch 19 frame so với gold cho thấy ngưỡng "xác định được là xe bốn bánh" chủ quan hơn tôi nghĩ lúc gán. Nếu gán lại, tôi sẽ chờ thêm 2-3 frame để bbox đạt tối thiểu ~20px cạnh ngắn trước khi mở track, thay vì mở ngay khi nghi ngờ.

### Ca 3
- Clip / frame / ID: `clip_01`, frame 1-15 và frame 156-170, track ID 3
- Tình huống: `check_mot_labels.py` cảnh báo "bbox gần như đứng im" ở hai đoạn này (dấu hiệu thường gặp của quên bấm `outside`).
- Quyết định: Giữ nguyên — đây là xe đang đỗ ở lề đường, đúng luật "xe đang đỗ vẫn là vehicle và vẫn cần track suốt thời gian nó trong khung" trong GUIDE.md.
- Lý do: Đã xem lại bằng mắt (lượt 3 trong tự kiểm) và xác nhận xe không di chuyển thật, không phải bbox treo; không có xe nào khác đi vào che khuất trong đoạn này nên không tạo nhầm lẫn ID. Kết quả: đoạn này không xuất hiện trong danh sách lỗi của `outputs/eval_vs_gold.json`, xác nhận quyết định đúng.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

- Ngưỡng "xe mới xuất hiện, đủ nhỏ để bắt đầu track" cần một con số cụ thể hơn thay vì cảm tính: đề xuất tối thiểu cạnh ngắn nhất của bbox >= 20px VÀ giữ hình dạng ổn định trong >= 2 frame liên tiếp trước khi mở track. Ca 1 và Ca 2 ở trên là bằng chứng — cả hai lần tôi mở track sớm hơn gold 6-19 frame.
- Cần ghi rõ quy trình đặt keyframe quanh lúc hai xe cắt nhau (không chỉ "đặt dày hơn" chung chung): tối thiểu 1 keyframe ngay trước, 1 ở giữa lúc chồng lấp lớn nhất, 1 ngay sau khi tách ra — vì đây chính xác là chỗ mô hình (ByteTrack và BoT-SORT-ReID) tách nhầm ID của track 5 và track 6 trong `outputs/eval_reid_vs_gold.json`, còn nhãn tay giữ đúng nhờ có đủ keyframe ở đoạn đó.
- Không có bạn cùng nhóm để kiểm chéo độc lập (làm cá nhân) — xem giới hạn này ghi rõ ở đầu `reports/review_partner.md`; nếu làm lại, ưu tiên tìm ít nhất một người kiểm chéo trước khi mở gold, vì tự-kiểm một mình dễ bỏ sót góc nhìn khác về ngưỡng "đủ rõ để mở track".
