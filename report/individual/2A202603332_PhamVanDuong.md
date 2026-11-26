# Báo cáo Day 13 — Phạm Văn Dương, 2A202603332

## 1. Kết luận trước
Mã số 2A202603332, task `D13-2A202603332`, 10 frame: s01f02, s01f22, s02f15, s02f37, s03f22, s03f42, s04f15, s04f39, s05f08, s05f44. Hệ điều hành Windows 11 Home Single Language, trình duyệt Microsoft Edge, CVAT 2.81.0 chạy local qua Docker. Người ngồi cạnh: 2A202604047.

Tổng cộng 55 cuboid (gồm Car, 1 Bus, 3 Pedestrian); frame ít nhất là s03f42 với 3 cuboid, vì frame này chỉ có đúng 3 vật thể, không có nhiều hơn. `selfcheck` cờ 1/55 cuboid (s01f02 idx 9, người đi bộ: có 55 điểm sát ngay ngoài mép). 

Phần tôi tin tưởng nhất: Việc xác định class của những vật thể có hình dạng và đặc điểm tương đối rõ ràng. Với các vật thể có đủ điểm LiDAR, tôi có thể quan sát và ước lượng vị trí tương đối của chúng.

Phần tôi chưa tự tin nhất: Xác định hướng, đáy và kích thước của những vật thể ở xa hoặc có mật độ điểm thấp. Đặc biệt, với các vật thể chỉ có một số điểm rời rạc, tôi chưa đủ cơ sở để xác định chính xác hình dạng và class nên không cố gắng suy đoán.

Cách sử dụng CVAT
1. Đổi góc nhìn: Sử dụng các chế độ Perspective, Top, Side và Front để quan sát point cloud từ nhiều hướng, hỗ trợ xác định vị trí và hình dạng vật thể.
2. Vẽ và chỉnh cuboid: Chọn Shape để tạo cuboid độc lập cho từng vật thể trong frame; nháy đúp để bắt đầu vẽ, sau đó chuyển qua Top, Side và Front để điều chỉnh vị trí, kích thước và hướng.
3. Kiểm tra hướng và ảnh camera: Bật Cuboid orientation để hiển thị mũi tên hướng của cuboid; sử dụng preview để quan sát cuboid trên ảnh camera và hỗ trợ đối chiếu vị trí vật thể.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f02-3d.png`, `t1-s01f02-top.png`, `t1-s01f22-3d.png`, `t1-s01f22-top.png` (ảnh `-top` nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f02 (`t1-s01f02-top.png`) |phía trước xe nằm ở cạnh bên phải của ảnh khi nhìn từ trên xuống, +x chỉ về bên trên, +y chỉ về bên trái |mặt đường phẳng |30m  |car |car |phần phía sau xe `car` đi phía sau xe ego: không quan sát được phần bị che |
| s01f22 (`t1-s01f22-top.png`) |phía trước xe nằm ở cạnh trái của ảnh, +x chỉ về cạnh dưới, +y chỉ về cạnh phải |mặt đường phẳng |30m |car|car |phần bị che phía sau xe bus |

Chỉ camera cho biết:
1. s01f02: Camera ghi nhận được các phương tiện ở xa, phía sau xe.
2. Camera cung cấp thông tin về màu sắc của phương tiện, giúp phân biệt các xe có đặc điểm hình dạng tương tự trên point cloud.
3. Camera hỗ trợ nhận diện loại phương tiện (car, bus, truck) khi số lượng điểm LiDAR chưa đủ để phân biệt rõ.

Chỉ LiDAR cho biết:
1. s02f15: Phát hiện một xe nằm khuất phía sau hai xe ở làn đối diện, vị trí này không được camera quan sát thấy.
2. s02f37: Phát hiện các xe ở phía trước, khoảng cách xa, nằm ngoài vùng quan sát của camera cam01.
3. s03f22: Phát hiện các xe ở phía trước, khoảng cách xa, camera cam01 không ghi nhận được.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f15 |phải |không phát hiện khi đọc lướt |
| s02f37 |phải | không phát hiện khi đọc lướt|
| s03f22 |phải | không phát hiện khi đọc lướt|
| s03f42 |phải | không phát hiện khi đọc lướt|
| s04f15 |phải |không phát hiện khi đọc lướt |
| s04f39 |phải | không phát hiện khi đọc lướt|
| s05f08 |phải | không phát hiện khi đọc lướt|
| s05f44 |phải |không phát hiện khi đọc lướt |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f22, idx 2  | car|có |có |Đã kiểm  |Dài 3,8–5,0 m; rộng 1,70–1,90 m; cao 1,40–1,60 m |
| (b) xe buýt hoặc xe tải | s01f22, idx 3 |bus|có |có |Đã kiểm  |Bus: 12,20 × 2,86 × 3,44 m; đáy −1,71 m |
| (c) người đi bộ hoặc xe hai bánh | s01f02, idx 9  |Pedestrian |có |có | Đã kiểm | 0,66 × 0,60 × 1,75 m|

Vòng hiệu chuẩn: cuboid của bạn lệch so với các bạn ở điểm nào, cả lớp chốt cách hiểu guideline nào? Và các vật bạn **bỏ qua vì không đủ cơ sở** (frame, vị trí, mô tả, loại nghi vấn như UNCERTAIN_CLASS / LOW_VISIBILITY), hoặc ghi "không có".
Không làm

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f22, `t4-1-bong-che.png` |Xe buýt màu xanh ở bên trái, cách khoảng 2 m, có thể đã chắn đường truyền LiDAR đến khu vực phía sau nó. | Một góc phía sau xe buýt gần như không có điểm phản hồi, trong khi các khu vực ở hai bên vẫn có mật độ điểm tương đối dày.| Vùng không có điểm phía sau xe buýt được xem là vùng chưa quan sát được do bị che khuất, không được mặc định là đường trống. Chỉ gán cuboid cho phần vật thể có đủ bằng chứng quan sát được.|
| 2. Thưa ở xa | s03f22, `t4-2-thua-o-xa.png` | Khoảng cách lớn có thể làm tín hiệu phản hồi suy yếu và mật độ điểm thu được giảm.| Một xe ở phía trước, cách khoảng 30–50 m, chỉ có vài chục điểm phản hồi, khiến hình dạng vật thể không rõ ràng.| Khó xác định chính xác kích thước, vị trí và hướng của cuboid. Chỉ gán khi có đủ cơ sở nhận diện và xác định hình dạng; không tự suy đoán phần không có dữ liệu.|
| 3. Phản xạ thấp / ít điểm bất thường | s04f15, `t4-3-phan-xa-thap.png` |Có thể liên quan đến đặc tính phản xạ của bề mặt vật thể, góc chiếu của tia LiDAR hoặc khoảng cách quan sát lớn.| Xuất hiện một số điểm phản hồi bất thường ở khu vực phía sau, cách khoảng 40 m. Các điểm này chưa thể hiện rõ một bề mặt hoặc hình dạng vật thể hoàn chỉnh.|Không nên tự tạo cuboid chỉ dựa trên một số điểm rời rạc. Cần có thêm bằng chứng để xác định đó là vật thể có phản xạ yếu hay chỉ là các điểm nhiễu. |
| 4. Điểm lạ | s05f08, `t4-4-diem-la.png` |Có thể do nhiễu đo, phản xạ đa đường hoặc sai số trong quá trình thu nhận dữ liệu LiDAR.| Một số điểm nằm lơ lửng giữa không trung ở khu vực phía sau, cách khoảng 40 m; không xác định được chúng thuộc về vật thể nào.|Không gán cuboid cho các điểm chưa xác định được là vật thể. Tránh suy đoán loại vật thể hoặc kích thước khi không có đủ bằng chứng. |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603332-bangA.csv` và cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét |
|---|---|---|---|---|---|---|---|
| s01f22, 2, Car | −0,972 | 1,502 | −1,723 | 3,25 | 3,39 m | 269,8° | chạm đường |
| s02f37, 3, Car | −0,585 | 1,607 | −1,388 | 31,79 | 31,79 m | 261,8° | lơ lửng ~0,31 m (vừa quá ngưỡng 0,3) |
| s01f02, 7, Car | −1,252 | 1,657 | −2,081 | 22,08 | 22,11 m | 88,3° | chìm ~0,38 m |

(a) Gần nhất: s01f22 idx 2, `dist_m` 3,25, r ≈ 3,39 m. Xa nhất: s02f37 idx 3, `dist_m` 31,79, r ≈ 31,79 m (r lớn hơn dist_m một chút vì có thành phần z).

(b) Người đi bộ s01f02 idx 9: l 0,66, w 0,60, h 1,75 — cả ba trong khoảng "người lớn" (0,4–0,9 / 0,5–0,9 / 1,50–1,95). Xe con s01f02 idx 1: l 4,32, w 1,71, h 1,51 — trong khoảng sedan. Ngoài khoảng sedan: có SUV: s01f02 idx 3 có w 2,00 và s01f22 idx 4 có w 2,07 vượt 1,90 của sedan.

(c) Xe cùng chiều phía trước: s01f22 idx 2, yaw 269,8° (gần 270°). Xe ngược chiều: s01f02 idx 4, yaw 89,6° (gần 90°). Cả hai khớp.

Mật độ theo khoảng cách:

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f22, 2, Car | 3,2 | 3214 |
| s01f02, 3, Car | 14,5 | 603 |
| s03f22, 1, Car | 21,1 | 267 |
| s02f37, 3, Car | 31,8 | 141 |

Quy luật: xa gấp ~4,5 lần (3,2 → 14,5 m) thì số điểm còn khoảng 1/5; xa gấp 2,2 lần nữa (14,5 → 31,8 m) thì còn khoảng 1/4. So với slide (1968 → 718 → 214 → 72), cùng xu hướng giảm mạnh theo khoảng cách; ở 30 m của tôi cao hơn (141 so với 72), có thể vì chỉ là một mẫu và cuboid có thể bao thêm điểm nền (giả thuyết).

Hai cuboid bạn **đã sửa** sau `preview` / `selfcheck` (frame, idx, sửa gì) và cảnh báo giữ nguyên kèm lý do: 

**Các cuboid đã kiểm tra sau `preview` / `selfcheck`:**

- `s01f02`, idx 7 (Car): Theo số liệu, đáy ở khoảng −2,08 m, thấp hơn mặt đường ước tính khoảng −1,7 m, tương ứng cuboid bị chìm khoảng 0,38 m. Đã kiểm tra và xác nhận, chỉnh lại độ cao của cuboid.
- `s02f37`, idx 3 (Car): Đáy ở khoảng −1,39 m, cao hơn mặt đường ước tính khoảng 0,31 m, cho thấy cuboid có dấu hiệu lơ lửng. Đã kiểm tra lại vị trí đáy của cubiod, xác nhận và chỉnh lại

**Cảnh báo được giữ nguyên:**

- `s01f02`, idx 9 (Pedestrian): Giữ nguyên cuboid dù `selfcheck` cảnh báo có 55 điểm nằm sát bên ngoài mép cuboid. Tôi giữ nguyên cuboid theo kết quả quan sát vì vị trí đó có 2 người đứng cạnh nhau, nên các điểm nằm sát bên ngoài mép box là của người bên cạnh.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model học trên nhãn 3D đo bằng mét vì xe tự lái cần biết vị trí và kích thước thật của vật để tính khoảng cách và tránh va chạm; pixel đổi theo khoảng cách và góc camera nên không cho số đo thật. Vật thưa điểm (xa, bị che, phản xạ thấp) cần quy tắc riêng vì có quá ít điểm để xác định kích thước và hướng, nên đoán sẽ tạo nhãn nhiễu làm model học sai.

(b) Dùng AI: 

- Claude Code: kiểm tra checkpoint/bài nộp bằng `check_submission.py`, soạn nháp các mục báo cáo, tôi kiểm tra bản nháp và đền số liệu; các mục quan sát từ ảnh và CVAT tôi tự điền.

- ChatGPT: hỏi về hướng dẫn thao tác, thực hành, cách diễn đạt, trình bày bảng.

(c) Điều còn chưa chắc và lỗi phát hiện sau hạn: hiện tại chưa có