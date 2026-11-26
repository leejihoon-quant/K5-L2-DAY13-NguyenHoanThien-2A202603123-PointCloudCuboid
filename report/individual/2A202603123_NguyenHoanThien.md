# Báo cáo Day 13 — Nguyễn Hoàn Thiện, 2A202603123

## 1. Kết luận trước
- **Mã số:** `2A202603123` · **Task CVAT:** `D13-2A202603123` (Job 32) · **10 frame:** `s01f15`, `s01f46`, `s02f27`, `s02f42`, `s03f07`, `s03f57`, `s04f32`, `s04f47`, `s05f07`, `s05f27`.
- **Môi trường:** Windows 11, trình duyệt Chrome/Edge, CVAT `v2.83.0`. · **MSSV bạn ngồi cạnh:** `2A202603217`.
- **Tổng kết quả:** Đã gán **75 cuboid** (`Car`: 66, `Truck`: 3, `Bus`: 3, `Cyclist`: 2, `Pedestrian`: 1); frame ít nhất là `s02f42` (4 cuboid) do đoạn đường thoáng, ít xe lưu thông xung quanh. Phiếu tự kiểm `-tukiem.csv` cờ **2/75 cuboid** cảnh báo điểm sát mép (`s04f47 #5` và `s05f27 #7`). Phần tự tin nhất là phân loại 5 class, căn hướng mũi tên đỏ `+X` và chạm đáy mặt đường; phần yếu nhất là suy luận chiều dài cho các xe ở xa `> 35 m` hoặc bị che khuất một phần.
- **Thao tác CVAT 3D:**
  - Đổi góc nhìn: dùng chuột trái xoay, chuột phải/giữ Shift để dời, con lăn để thu phóng, và dùng các khung nhìn `Top / Side / Front` để căn từng mặt.
  - Vẽ và chỉnh cuboid: chọn công cụ `Cuboid` chế độ `Shape`, nháy đúp hoặc kéo các điểm điều khiển trên `Top / Side / Front` để ôm sát cụm điểm và chạm đáy mặt đường.
  - Bật hướng & kiểm tra camera: bật `Cuboid orientation` để kiểm tra mũi tên đỏ `+X`, sau đó chạy `python scripts/lab.py preview` để đối chiếu hộp 3D chiếu lên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f15` và `s01f46` (ảnh tham chiếu `t1-s01f15-3d`, `t1-s01f15-top`, `t1-s01f46-3d`, `t1-s01f46-top`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f15` | Phía trước xe ở cạnh dưới (`-Y`); `+X` sang trái, `+Y` ra phía sau xe | Đường phẳng, thấp hơn LiDAR khoảng `1,75 m` | `~48,0 m` (`Car`) | `Car`, `Bus` | Bên trái có 1 `Car` song song (`3,99 m`), phía sau có 3 `Car` | Cụm điểm nhỏ sát dải phân cách ngoài `> 35 m` |
| `s01f46` | Phía trước xe ở cạnh dưới (`-Y`); `+X` sang trái, `+Y` ra phía sau xe | Đường phẳng, thấp hơn LiDAR khoảng `1,80 m` | `~39,5 m` (`Car`) | `Car`, `Truck`, `Bus` | Bên phải sát xe có 1 `Truck` (xe van chở hàng, `3,15 m`), phía sau có 1 `Car` (`7,86 m`) | Cụm điểm rất thưa trên vỉa hè ngoài `30 m` |

- **3 điều chỉ camera cho biết:** màu sắc/kiểu dáng xe (nhận diện xe van chở hàng hay minivan, xe buýt), phân biệt đầu và đuôi xe qua đèn pha/đèn hậu khi LiDAR chỉ có 1 mặt, và phân biệt người dắt xe (`Pedestrian`) với người đang lái (`Cyclist`).
- **3 điều chỉ LiDAR cho biết:** khoảng cách mét chính xác từ cảm biến đến vật (`dist_m`), kích thước thật 3D (`length`, `width`, `height`) không bị biến dạng phối cảnh, và độ cao đáy hộp so với mặt đường (`z_bottom`).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f27` | Cạnh dưới (`-Y`) | Có xe đi ngược chiều ở làn đối diện (`yaw ≈ 90°`) |
| `s02f42` | Cạnh dưới (`-Y`) | Đường thoáng, chỉ có 1 `Pedestrian` bên phải và ít xe; mặt đường phía xa `~40 m` hơi dốc lên (`-1,21 m`) |
| `s03f07` | Cạnh dưới (`-Y`) | Mật độ xe dày, có 1 xe van giao hàng phía trước và 1 xe con sát bên phải |
| `s03f57` | Cạnh dưới (`-Y`) | Có 1 `Bus` lớn phía sau (`dist_m = 24,85 m`) |
| `s04f32` | Cạnh dưới (`-Y`) | Có 1 `Cyclist` (người đi xe hai bánh) chạy song song sát bên phải (`6,04 m`) |
| `s04f47` | Cạnh dưới (`-Y`) | `Cyclist` tiến lên phía trước bên phải (`16,24 m`), có thêm xe làn đối diện |
| `s05f07` | Cạnh dưới (`-Y`) | Ngã tư lớn, có xe tải nặng (`Truck`, cao `3,05 m`) phía sau và xe rẽ ngang |
| `s05f27` | Cạnh dưới (`-Y`) | Qua ngã tư, có xe đi cắt ngang ở khoảng gần (`-6,59 m`) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f15`, `#1` | `Car` | Có (`270,0°`, hướng `-Y`) | Có (`z_bottom = -1,77 m`, `ground_gap = -0,01 m`) | Đã kiểm đủ 3 view + ảnh camera | Có (`4,50 × 1,85 × 1,48 m`, chuẩn Sedan) |
| (b) xe buýt hoặc xe tải | `s01f15`, `#4` | `Bus` | Có (`270,0°`, hướng `-Y`) | Có (`z_bottom = -1,89 m`, `ground_gap = 0,06 m`) | Đã kiểm đủ 3 view + ảnh camera | Có (`10,50 × 2,50 × 3,15 m`, chuẩn Bus thường) |
| (c) người đi bộ hoặc xe hai bánh | `s04f32`, `#1` | `Cyclist` | Có (`270,0°`, hướng `-Y`) | Có (`z_bottom = -1,73 m`, `ground_gap = 0,00 m`) | Đã kiểm đủ 3 view + ảnh camera | Có (`1,90 × 0,75 × 1,69 m`, chuẩn xe máy có người) |

- **Vòng hiệu chuẩn & vật bỏ qua:** Chốt nguyên tắc xe van chở hàng gán `Truck`, luôn kéo đáy hộp chạm mặt đường thay vì ôm mỗi cụm điểm thân xe, và suy luận đủ chiều dài theo bảng 4.3 khi chỉ thấy mặt sau. Vật bỏ qua: xe van phía sau ở rìa xa (`> 30 m`) trên frame `s03f07` do bị che khuất và cụm điểm LiDAR quá thưa (`UNCERTAIN_SCOPE`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f15` (`t1-s01f15-top`, task cá nhân) | Xe chạy phía trước chắn tia laser chiếu tới phần thân trước của xe buýt | Chỉ thấy mặt đuôi và một phần nóc của `Bus #4` ở `28,9 m` | Không thu nhỏ hộp theo điểm thấy được; phải suy luận đủ chiều dài `10,50 m` theo bảng 4.3 |
| 2. Thưa ở xa | `s01f15` (`Car #7` ở `48,02 m`, task cá nhân) | Các tia LiDAR phân kỳ theo khoảng cách nên số tia trúng vật ở xa giảm mạnh | Chỉ còn `47` điểm rời rạc (`points_in = 47`) | Đối chiếu ảnh camera và suy luận kích thước `4,40 × 1,85 × 1,65 m`, không đoán nếu không rõ |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f07` (`Car #1` đen ở gần, task cá nhân) | Sơn xe tối màu hấp thụ tia hồng ngoại mạnh hơn bề mặt sáng | Số điểm trên thân xe ít hơn các xe trắng cùng khoảng cách | Kiểm tra kỹ trên `Top/Side/Front` và ảnh camera để bao trọn mép xe |
| 4. Điểm lạ | `s05f07` (khu vực ngã tư, task cá nhân) | Phản xạ đa tia từ kính/kim loại hoặc bụi, cây ven đường | Xuất hiện vài điểm lơ lửng rời rạc trên cao hoặc sát mặt đường | Không kéo cuboid bao cả điểm nhiễu trên cao; giữ chiều cao hợp lý theo bảng 4.3 |

## 5. Số từ cuboid của tôi (Bước 5)

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét |
|---|---|---|---|---|---|---|---|
| `s01f46`, `#1`, `Truck` | -0,735 | 1,950 | -1,710 | 3,15 | 3,24 | 270,0 | Chạm đường (`ground_gap = 0,06 m`) |
| `s04f32`, `#1`, `Cyclist` | -0,887 | 1,689 | -1,732 | 6,04 | 6,11 | 270,0 | Chạm đường (`ground_gap = 0,00 m`) |
| `s01f15`, `#7`, `Car` | -0,636 | 1,650 | -1,461 | 48,02 | 48,03 | 267,9 | Chạm mặt đường dốc ở xa (`-1,46 m`) |

- **(a) Gần nhất & xa nhất:** Gần nhất là `s01f46 #1 Truck` (`dist_m = 3,15 m`, `r = 3,24 m`); xa nhất là `s01f15 #7 Car` (`dist_m = 48,02 m`, `r = 48,03 m`).
- **(b) So với bảng 4.3:** `Pedestrian` (`s02f42 #1`: `0,65 × 0,65 × 1,65 m`) nằm trọn trong khoảng người lớn (`0,4–0,9 × 0,5–0,9 × 1,50–1,95 m`). `Car` (`s01f15 #6`: `4,20 × 1,80 × 1,50 m`) nằm gọn trong khoảng Sedan (`3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`), không có chiều nào ngoài khoảng.
- **(c) Hướng `yaw_deg`:** Xe đi cùng chiều phía trước như `s01f15 #4` có `yaw_deg = 270,0°` (gần `270°`); xe ngược chiều ở làn đối diện như `s02f27 #1` có `yaw_deg = 90,0°` (gần `90°`).

Mật độ theo khoảng cách (4 `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---:|---:|
| `s03f07`, `#9`, `Car` | 3,49 | 2123 |
| `s05f27`, `#3`, `Car` | 12,55 | 1316 |
| `s05f27`, `#6`, `Car` | 22,53 | 325 |
| `s01f15`, `#7`, `Car` | 48,02 | 47 |

- **Quy luật mật độ:** Khi khoảng cách tăng từ `3,49 m` lên `22,53 m` (xa gấp `~6,5 lần`), số điểm giảm từ `2123` xuống `325` (còn khoảng `1/6,5` đến `1/10`, giảm mạnh theo khoảng cách tương tự đường cong tham chiếu: `~1968` điểm ở `0–10 m`, `~214` điểm ở `20–30 m`, và `47` điểm ở `48 m`).
- **Tự kiểm (`preview` / `selfcheck`):** Đã sửa `s01f15 #7 Car` (mở rộng chiều dài từ `3,00 m` lên `4,40 m` và nâng đáy từ `-2,17 m` lên `-1,46 m`) và `s03f57 #7 Bus` (thu chiều rộng từ `3,80 m` về `2,55 m` chuẩn Bus, nâng đáy lên `-1,69 m`). Giữ nguyên 2 cảnh báo nhẹ `s04f47 #5` (39 điểm sát mép) và `s05f27 #7` (104 điểm sát mép) vì kiểm tra ảnh và BEV thấy hộp đã ôm sát xe, các điểm ngoài mép là lề đường/cột nhỏ kế bên.
- **Đổi kiểm:** Đã kiểm chéo với MSSV `2A202603217`, đối chiếu hai con số trên bảng A (`s01f46 #1 Truck`: `z_bottom = -1,710 m`, `dist_m = 3,15 m`; và `s01f15 #4 Bus`: `yaw_deg = 270,0°`). Kết quả **khớp** hoàn toàn, không phát sinh sai lệch cần sửa.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa đối với model 3D:** Nhãn 3D đo bằng mét giúp mô hình học trực tiếp kích thước vật lý, khoảng cách an toàn và quỹ đạo chuyển động trong không gian thực thay vì bị méo phối cảnh theo pixel. Vật thưa điểm cần quy tắc riêng để tránh tạo nhãn nhiễu khi thông tin hình học quá ít, gây sai lệch hàm mất mát lúc huấn luyện.
- **(b) Dùng AI:** 
- **(c) Điều còn chưa chắc:** Xe van chở hàng ở `s01f46 #1` đang gán `Truck` theo quy tắc van chở hàng, cần xác nhận thêm nếu phân loại theo thân xe minivan; một số xe ở vùng `40–48 m` có điểm thưa nên độ chính xác mép xa phụ thuộc nhiều vào suy luận từ ảnh camera.
