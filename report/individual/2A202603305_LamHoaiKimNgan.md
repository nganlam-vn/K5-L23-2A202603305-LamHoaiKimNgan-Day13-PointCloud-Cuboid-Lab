# Báo cáo Day 13 — 2A202603305-LamHoaiKimNgan

## 1. Kết luận trước
- **Thông tin chung:** Mã số `2A202603305`, task `D13-2A202603305`, 10 frame được giao gồm `s01f15`, `s01f43`, `s02f18`, `s02f33`, `s03f06`, `s03f55`, `s04f04`, `s04f22`, `s05f00`, `s05f46`. Môi trường làm việc: Windows 11, trình duyệt Google Chrome, phiên bản CVAT `v2.83.0`. MSSV bạn ngồi cạnh: `2A202603067`.
- **Tổng quan kết quả:** Tổng cộng đã vẽ **89 cuboid** (67 `Car`, 13 `Pedestrian`, 3 `Bus`, 3 `Cyclist`, 3 `Truck`). Frame ít cuboid nhất là `s01f15` với 8 cuboid (các frame còn lại đều có 9 cuboid) do mật độ phương tiện và người đi bộ ở tầm gần tại thời điểm này thưa hơn. Lệnh `selfcheck` cờ **25/89 cuboid**. Phần tin cậy nhất là kích thước và vị trí của các xe con gần (0–20 m) có cụm điểm rõ; phần còn yếu nhất là căn chỉnh góc `yaw` sát thân xe ở một số ca, kiểm soát chồng lấn và độ cao đáy hộp ở khu vực lề đường hoặc vật xa.
- **Cách dùng CVAT:**
  - Đổi góc nhìn: dùng chuột và con lăn để xoay, dời (pan) và thu phóng khung 3D; dùng các nút điều hướng trên giao diện để đưa nhanh về góc nhìn từ trên xuống (Top).
  - Vẽ và chỉnh một cuboid: chọn công cụ vẽ ở chế độ **Shape**, đặt hộp tại cụm điểm rồi nháy đúp vào cuboid để chỉnh 6 mặt trên 3 cửa sổ **Top / Side / Front**.
  - Bật mũi tên hướng & đối chiếu ảnh camera: bật hiển thị hướng (**Cuboid orientation**) để kiểm tra mũi tên đỏ `+X` chỉ đúng đầu xe, đồng thời chạy `python scripts/lab.py preview` để xem cuboid chiếu lên 7 ảnh camera (do CVAT 2.83.0 không có nút chiếu vàng trực tiếp).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f15` và `s01f43`. Ảnh: `t1-s01f15-3d.png`, `t1-s01f15-top.png`, `t1-s01f43-3d.png`, `t1-s01f43-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f15` | Phía trước (`-y`, hướng `cam01`) ở cạnh dưới; `+x` (trái) chỉ sang phải, `+y` (sau) chỉ lên trên | Phẳng, thấp hơn LiDAR ~ `1,7 m` (`z ≈ -1,7 m`) | `47,69 m` (`Car` #8 phía trước) | `Car`, `Bus`, `Pedestrian` | Xe con chạy song song bên trái (`cam05`/`cam06`), xe con và người đi bộ phía sau (`cam09`) | Các xe ở rất xa (> 50 m) phía trước và dải cây xanh hai bên lề đường |
| `s01f43` | Phía trước (`-y`, hướng `cam01`) ở cạnh dưới; `+x` (trái) chỉ sang phải, `+y` (sau) chỉ lên trên | Phẳng, thấp hơn LiDAR ~ `1,7 m` (`z ≈ -1,7 m`) | `17,05 m` (`Pedestrian` #9 phía sau) | `Car`, `Cyclist`, `Pedestrian` | Bên phải (`cam07`/`cam08`) có 2 `Cyclist` và `Pedestrian`; phía sau (`cam09`) có xe con và người đi bộ | Cụm xe hai bánh dựng sát lề và các cụm ở xa (> 35 m) lẫn vào tán cây |

Ba điều **chỉ camera** cho biết và ba điều **chỉ LiDAR** cho biết (từ `s01f15` và `s01f43`):
- **Chỉ camera cho biết:** (1) màu sắc và đặc trưng nhận diện loại xe (ví dụ `Bus` màu trắng đỏ ở phía trước trong `s01f15`); (2) phân biệt đầu xe và đuôi xe nhờ đèn pha, đèn hậu và kính lái; (3) phân biệt người đang điều khiển xe (`Cyclist` trên làn phải ở `s01f43` qua `cam07`/`cam08`) với người đi bộ hoặc xe dựng không người.
- **Chỉ LiDAR cho biết:** (1) tọa độ 3D và khoảng cách chính xác theo mét (`x, y, z`, `dist_m`); (2) kích thước thật 3D (`length, width, height`) và cao độ mặt đường `z ≈ -1,7 m`; (3) các vật nằm trong vùng quét 360° kể cả ở góc mù giữa các camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f18` | Cạnh dưới (hướng `-y`) | Có hai xe con gần nhau phía trước dễ chồng hộp nếu không tách kỹ |
| `s02f33` | Cạnh dưới (hướng `-y`) | Có nhiều xe con nối đuôi nhau ở tầm 23–32 m, điểm thưa dần |
| `s03f06` | Cạnh dưới (hướng `-y`) | Xe cộ dày đặc ở cự ly gần (6–13 m), có xe tải `Truck` bên trái |
| `s03f55` | Cạnh dưới (hướng `-y`) | Có xe buýt lớn `Bus` tạo vùng bóng che rộng phía sau |
| `s04f04` | Cạnh dưới (hướng `-y`) | Có xe con rất gần tâm LiDAR (`3,05 m`) với gần 5.000 điểm |
| `s04f22` | Cạnh dưới (hướng `-y`) | Nhiều xe con và người đi bộ tập trung quanh cự ly 10–14 m |
| `s05f00` | Cạnh dưới (hướng `-y`) | Xe tải `Truck` ở `6,66 m` chắn phần lớn góc nhìn phía sau |
| `s05f46` | Cạnh dưới (hướng `-y`) | Có cả `Truck` (`6,84 m`) và `Bus` (`16,51 m`) cùng các xe con ở xa trên 43 m |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f15, #6` | `Car` | Đúng (`270,3°`, hướng cùng chiều) | Chạm đường (`z_bottom = -1,733 m`) | Đã kiểm Top/Side/Front + `preview` | `4,797 × 2,071 × 1,522 m` (khớp bảng 4.3) |
| (b) xe buýt hoặc xe tải | `s03f55, #8` | `Bus` | Đúng (`90,0°`, hướng ngược chiều) | Chạm đường (`z_bottom = -1,700 m`) | Đã kiểm Top/Side/Front + `preview` | `11,000 × 2,550 × 3,000 m` (khớp xe buýt thường) |
| (c) người đi bộ hoặc xe hai bánh | `s02f18, #6` | `Pedestrian` | Đúng (`270,0°`) | Chạm đường (`z_bottom = -1,807 m`) | Đã kiểm Top/Side/Front + `preview` | `0,650 × 0,650 × 1,700 m` (khớp người lớn) |

- **Vòng hiệu chuẩn & vật bỏ qua:** Theo quy định mới của lớp, không thực hiện task chung (`D13-HC485087`). Các vật bỏ qua vì không đủ cơ sở là những cụm điểm ở khoảng cách > 50 m hoặc nằm sâu sau bóng che xe buýt/xe tải chỉ còn vài điểm rời rạc (`LOW_VISIBILITY` / `UNCERTAIN_SCOPE`), không đủ 15–20% thể tích để suy luận hình dạng.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | Bộ 10 frame cá nhân / `t4-1-bong-che.png` | Tia LiDAR bị thân xe gần chắn nên không chiếu tới vùng phía sau | Vùng tối trống điểm kéo dài phía sau vật cản, trong khi hai bên vẫn dày điểm | Vật bị che một phần thì bật `occluded` và suy luận kích thước; bị che hoàn toàn thì không vẽ |
| 2. Thưa ở xa | Bộ 10 frame cá nhân / `t4-2-thua-o-xa.png` | Các tia LiDAR phân kỳ theo khoảng cách nên ở tầm 30–50 m số tia trúng vật ít đi | Xe ở xa chỉ còn vài đường quét thưa thớt, không rõ đường nét đầy đủ | Chỉ vẽ khi thấy ~15–20% thể tích hoặc suy luận được từ ảnh camera + bảng 4.3; không thì bỏ qua (`LOW_VISIBILITY`) |
| 3. Phản xạ thấp / ít điểm bất thường | Bộ 10 frame cá nhân / `t4-3-phan-xa-thap.png` | Bề mặt xe tối màu hoặc kính hấp thụ/khúc xạ tia laser | Trên thân xe xuất hiện vùng thưa điểm bất thường dù ở cự ly gần | Đối chiếu ảnh camera để lấy đúng ranh giới vật, bao trọn cả phần bị mất điểm |
| 4. Điểm lạ | Bộ 10 frame cá nhân / `t4-4-diem-la.png` | Nhiễu đa đường (multipath) hoặc bụi/lá cây phản xạ | Xuất hiện cụm điểm rời rạc trôi nổi phía trên hoặc ngoài vật thể thật | Không gán cuboid cho cụm điểm lạ và không kéo giãn hộp để bao nhầm chúng |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603305-bangA.csv` và cột `points_in` của `submission/D13-2A202603305-tukiem.csv` (mặt đường khoảng `-1,7 m`).

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f15, #6, Car` | `-0,973 m` | `1,522 m` | `-1,733 m` | `3,66 m` | `3,78 m` | `270,3°` | Chạm đường (`ground_gap_m = 0,04 m`) |
| `s03f55, #8, Bus` | `-0,200 m` | `3,000 m` | `-1,700 m` | `21,75 m` | `21,75 m` | `90,0°` | Chạm đường (`ground_gap_m = -0,09 m`) |
| `s01f15, #5, Car` | `-1,436 m` | `1,443 m` | `-2,157 m` | `33,61 m` | `33,65 m` | `93,1°` | Hơi chìm (`ground_gap_m = -0,26 m`) |

- **(a) Cuboid gần nhất và xa nhất:** Gần nhất là `s04f04, #1` (`Car`) với `dist_m = 3,05 m`, `r = 3,13 m`; xa nhất là `s01f15, #8` (`Car`) với `dist_m = 47,69 m`, `r = 47,69 m`.
- **(b) So sánh kích thước với bảng 4.3:** `Pedestrian` (`s01f15, #3`) có `l = 0,669 m`, `w = 0,753 m`, `h = 1,681 m`, cả 3 chiều đều nằm gọn trong khoảng người lớn (`0,4–0,9 m × 0,5–0,9 m × 1,50–1,95 m`). Với `Car` (`s01f15, #7`), kích thước đo được là `l = 2,827 m`, `w = 2,302 m`, `h = 1,408 m`: chiều dài ngắn hơn khoảng chuẩn (`3,8–5,0 m`) và chiều rộng hơi lớn hơn (`1,70–1,90 m`) do hộp bị cắt hơi sát theo phần điểm nhìn thấy ở khoảng cách `18,72 m`.
- **(c) `yaw_deg`:** Xe cùng chiều phía trước `s01f15, #1` (`Car`) có `yaw_deg = 270,5°` (gần `270°`); xe ngược chiều `s01f15, #5` (`Car`) có `yaw_deg = 93,1°` (gần `90°`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f43, #1, Car` | `6,76 m` | 1645 |
| `s04f22, #9, Car` | `14,31 m` | 944 |
| `s02f33, #3, Car` | `23,89 m` | 230 |
| `s05f46, #7, Car` | `36,48 m` | 68 |

- **Quy luật mật độ & so với slide:** Khi khoảng cách tăng gấp ~2 lần (`6,76 m → 14,31 m`), số điểm còn khoảng một nửa (`1645 → 944`); tăng lên `23,89 m` và `36,48 m` thì số điểm giảm mạnh xuống `230` và `68` điểm, phù hợp với đường cong tham chiếu trên slide (0–10 m: ~1968 điểm, 10–20 m: ~718 điểm, 20–30 m: ~214 điểm, 30–40 m: ~72 điểm).
- **Tự kiểm (`preview` / `selfcheck`):** Trong 25 cuboid bị cờ của bản hiện tại, các cảnh báo *"điểm sát ngay ngoài mép"* ở `Pedestrian` và `Cyclist` (`s01f43 #3, #4`, `s03f06 #7`, `s04f22 #7, #8`) được giữ nguyên vì điểm nằm ngoài mép là điểm lề đường/cây cối bên cạnh; cảnh báo *"đáy thấp hơn mặt đường 0,34–0,38 m"* ở `s01f43 #3, #4, #5` cũng giữ nguyên do đối tượng đứng trên vỉa hè/lề đường có cao độ khác mặt đường chính. Các cảnh báo nặng hơn (2 hộp 0 điểm ở `s01f43 #9`, `s03f55 #9` và 2 cặp chồng hộp `s02f18 #2-#3`, `s03f55 #3-#5`) cần rà soát chỉnh lại trên CVAT nếu cập nhật thêm trước hạn nộp.
- **Đổi kiểm:** Đã trao đổi với bạn ngồi cạnh MSSV `2A202603067`, đối chiếu hai con số `z_bottom` (quanh `-1,7 m`) và `dist_m` (giới hạn `≤ 50 m`) trong bảng A; cả hai đều khớp cách tính `z_bottom = z - height/2` và `dist_m = √(x² + y²)`.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa của nhãn:** Mô hình 3D Object Detection cần biết chính xác vị trí, kích thước và hướng của vật thể trong không gian thực để tính khoảng cách an toàn và lập quỹ đạo tránh va chạm, nên nhãn 3D bắt buộc đo bằng **mét** thay vì pixel trên ảnh 2D (vốn bị biến dạng theo khoảng cách và tiêu cự). Với các vật thưa điểm, cần quy tắc riêng (tối thiểu 15–20% thể tích hoặc có ảnh camera hỗ trợ) để tránh việc người gán nhãn đoán mò làm sai lệch phân bố kích thước và hướng mà mô hình học được.
- **(b) Dùng AI:**
- Hỗ trợ viết báo cáo
- **(c) Điều còn chưa chắc:** Một số cuboid ở cự ly xa hoặc gần lề đường vẫn còn cảnh báo trong phiếu tự kiểm (hộp rỗng, hộp chồng lấn, hoặc kích thước/góc `yaw` chưa thật sát thân xe) cần đối chiếu thêm trên 3 góc nhìn Top/Side/Front nếu có thời gian tinh chỉnh lại trên CVAT.
