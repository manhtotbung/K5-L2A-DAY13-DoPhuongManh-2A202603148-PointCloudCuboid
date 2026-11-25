# Báo cáo Day 13 — Đỗ Phương Mạnh, 2A202603148

## 1. Kết luận trước

- **Mã số học viên (MSSV):** `2A202603148` — **Task CVAT:** `D13-2A202603148` (Job #31).
- **10 frame được giao:** `s01f00`, `s01f51`, `s02f12`, `s02f28`, `s03f15`, `s03f42`, `s04f37`, `s04f52`, `s05f09`, `s05f27`.
- **Môi trường thực hiện:** Windows 11 (build 22631), trình duyệt Chromium/Chrome, CVAT local Server `2.81.0` / UI `2.81.0`.
- **Bạn ngồi cạnh (đổi kiểm):** `2A202603107`.

Tổng cộng đã gán nhãn **135 cuboid** dạng Shape (0 Track) trên 10 frame (103 `Car`, 15 `Cyclist`, 8 `Pedestrian`, 6 `Truck`, 3 `Bus`), trong đó có **98 cuboid** nằm trong phạm vi ≤ 30 m và **48 cuboid** bật `occluded = 1`. Frame có ít cuboid nhất là **`s03f42`** (**7 cuboid**) do đoạn đường này rất thưa xe trên làn chính (chỉ có 4 xe con trong 50 m trên mặt đường chính và 3 xe đỗ trên đường gom phía bên phải bị dải cây xanh che khuất). Sau khi rà soát bằng `preview` và `selfcheck`, còn **15/135 cuboid** (11,1%) mang cảnh báo tự kiểm (chủ yếu do vật thể nằm sát hàng rào, bụi cây hoặc xe kế cận). Phần tin tưởng nhất là phân loại class, hướng mũi tên `yaw_deg` và độ chạm đường của các xe trong phạm vi 30 m; phần còn yếu nhất là ước lượng chiều dài phần bị che khuất ở các xe xa 35–47 m và nhóm 3 xe đỗ sau gờ cỏ ở `s03f42`.

**Cách dùng CVAT 3D:**
1. **Đổi góc nhìn:** Trong khung 3D (Perspective), kéo chuột trái để xoay, kéo chuột phải để dời (pan), lăn con lăn để phóng to/thu nhỏ; xoay thẳng từ trên xuống và thu nhỏ (hoặc dùng các phím góc khung `U/O`, `J/L`, `I/K` và 4 mũi tên) để quan sát toàn cảnh 360°, nháy đúp vùng trống để về góc nhìn mặc định.
2. **Vẽ và chỉnh cuboid:** Chọn công cụ vẽ cuboid ở thanh trái → chọn 1 trong 5 nhãn (`Car`, `Truck`, `Bus`, `Cyclist`, `Pedestrian`) → bấm **Shape** (không bấm Track), đưa hộp vào cụm điểm rồi nháy đúp để đặt (nhấn `N` vẽ tiếp cùng nhãn); chọn hộp rồi chỉnh tâm, dài, rộng và xoay yaw trong khung **Top**, chỉ chỉnh độ cao và đáy chạm đường trong khung **Side** và **Front** (không xoay ở Side/Front), nhấn `Q` để bật `occluded` khi bị che khuất.
3. **Bật mũi tên hướng và xem chiếu camera:** Mở mục **Appearance** ở thanh phải → bật **Cuboid orientation** để hiển thị vạch đỏ ngắn `+X` chỉ về đầu xe và đối chiếu với các khung ảnh camera (`cam01`–`cam09`); sau khi bấm **Save** (`Ctrl+S`), chạy `python scripts/lab.py sync` để xuất ảnh `preview` trong `work/xem/` kiểm tra độ khớp cuboid trên cả 7 góc camera và BEV.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: `s01f00` (`t1-s01f00-3d.png`, `t1-s01f00-top.png`) và `s01f51` (`t1-s01f51-3d.png`, `t1-s01f51-top.png`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f00` | Phía trước (`−y`, khớp `cam01`) hướng lên góc **trên-trái** (~10 giờ) của `t1-s01f00-top.png`; trục `+x` (trái) chỉ xuống góc **dưới-trái** (~7 giờ), trục `+y` (sau) chỉ xuống góc **dưới-phải** (~4 giờ) | Tương đối phẳng tại ngã tư rộng, hơi dốc nhẹ về phía trước (vùng gần `z ≈ −1,85 m`, phía trước 5–25 m `z ≈ −1,94 m`, phía sau 5–25 m `z ≈ −1,71 m`), thấp hơn LiDAR khoảng `1,8 m` | Khoảng `45 m` (xe con phía sau `44,99 m` trên `cam09`; hàng xe chờ ngã tư `39,6–40,7 m` trên `cam03`) | `Car` (sedan, SUV, MPV), `Bus` (xe buýt xanh), `Cyclist` (xe máy có người lái), `Pedestrian` (người đi bộ qua đường) | Bên trái: sedan trắng sát thân xe (`3,52 m`, `cam05`/`cam06`); phía sau: 4 xe con (`9,7–22,2 m`, `cam09`); bên phải: xe máy có người lái (`11,03 m`, `cam07`/`cam08`), SUV trắng (`23,27 m`) và xe buýt xanh (`32,68 m`, `cam07`) | Các cụm điểm rất thưa phía trước xa > `55 m` (dưới 10 điểm, không tách được thân xe) và cụm tán cây/cột tín hiệu lẫn rìa góc phải xa > `45 m` |
| `s01f51` | Phía trước (`−y`, khớp `cam01`) hướng lên góc **trên-trái** (~10 giờ) của `t1-s01f51-top.png`; trục `+x` (trái) chỉ xuống góc **dưới-trái** (~7 giờ), trục `+y` (sau) chỉ xuống góc **dưới-phải** (~4 giờ) | Phẳng đều dọc tuyến đường nhiều làn (quanh xe `z ≈ −1,80 m`, phía trước `−1,79 m`, phía sau `−1,81 m`), thấp hơn LiDAR khoảng `1,8 m` | Khoảng `46 m` (xe con phía trước bên phải `45,80 m` trên `cam03`; xe van chở hàng phía trước `34,97 m`) | `Car` (nhiều xe con chạy cùng chiều), `Truck` (xe van chở hàng trắng), `Cyclist` (xe đạp có người lái), `Pedestrian` (người đi bộ vỉa hè phải) | Bên trái: sedan đen chạy sát (`6,57 m`, `cam05`/`cam06`) và xe trắng sau-trái (`13,63 m`); phía sau: sedan trắng bám ngay sau (`6,94 m`, `cam09`); bên phải: người đạp xe làn xe đạp (`10,47 m`, `cam07`/`cam08`) | Dãy xe đạp dựng san sát trên vỉa hè phải lẫn vào thân cây và hàng rào bụi cây (không có người lái nên không gán) và nhóm người đứng chồng chéo ở góc ngã tư xa > `32 m` |

**Đối chiếu camera và LiDAR từ `s01f00` và `s01f51`:**
- **3 điều chỉ camera cho biết:** (1) Phân biệt loại phương tiện và trạng thái có người lái hay không (ví dụ SUV trắng `s01f00 #9` khác xe tải, xe đạp có người lái `s01f51 #5` khác hàng xe đạp dựng trống trên vỉa hè), (2) Nhận biết rõ đầu–đuôi xe qua đèn pha, đèn hậu, kính lái và hướng mặt người đi bộ khi point cloud chỉ có mặt cắt chữ L, (3) Màu sơn, vật liệu kính tối màu (như xe sedan đen `s01f51 #1` so với xe trắng `s01f51 #2`), vạch sơn làn đường và tín hiệu đèn giao thông.
- **3 điều chỉ LiDAR cho biết:** (1) Khoảng cách hình học 3D chính xác theo mét (ví dụ `s01f00 #1` cách `3,52 m`, xe buýt `#12` cách `32,68 m`), (2) Kích thước thực `(l, w, h)` theo mét cùng cao độ mặt đường (`z ≈ −1,80 m`) và độ dốc mặt đường dọc theo trục `y`, (3) Cấu trúc toàn cảnh 360° không có góc mù ngang (phát hiện cả vùng bóng che phía sau xe lớn và điểm bất thường theo trục cao `z`).

**Tám frame lướt:**

| Frame | Phía trước (`−y`) ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f12` | Góc trên-trái trong khung top-down CVAT (cạnh trên trên ảnh BEV của `preview`) | Hàng rào dải phân cách sát bên trái che một phần thân dưới các xe chạy ngược chiều; vỉa hè phải có nhiều xe điện đỗ phủ bạt không có người lái (không gán) |
| `s02f28` | Góc trên-trái trong khung top-down CVAT (cạnh trên trên ảnh BEV) | Hàng rào giữa đường phân tách hai chiều chạy rõ nét; trên vỉa hè trái có biển quảng cáo hộp đèn dễ gây nhầm nếu chỉ nhìn ảnh camera mà không đối chiếu 3D |
| `s03f15` | Góc trên-trái trong khung top-down CVAT (cạnh trên trên ảnh BEV) | Tường quây công trình cao dọc hai bên tạo vùng bóng che rất lớn; xe tải nhỏ và xe van chở hàng sát bên trái che một phần các xe liền kề |
| `s03f42` | Góc trên-trái trong khung top-down CVAT (cạnh trên trên ảnh BEV) | Làn đường chính khá thưa xe; phía bên phải có gờ cỏ và hàng cây nâng cao che khuất một phần 3 xe con đỗ trên đường gom |
| `s04f37` | Góc trên-trái trong khung top-down CVAT (cạnh trên trên ảnh BEV) | Xe tải ben lớn và xe jeep đen chạy ngược chiều sát dải phân cách màu vàng phía bên trái, cần lọc kỹ để hộp không bám nhầm vào điểm của hàng rào |
| `s04f52` | Góc trên-trái trong khung top-down CVAT (cạnh trên trên ảnh BEV) | Xe SUV trắng chạy sát bên phải (`6,66 m`) và một công nhân mặc áo phản quang đứng trên vỉa hè phải (`13,90 m`); một xe con ở `17,63 m` bị xe đỏ phía trước che khuất nặng |
| `s05f09` | Góc trên-trái trong khung top-down CVAT (cạnh trên trên ảnh BEV) | Ngã tư rộng đông xe đa hướng; có cụm **17 điểm lạ** nằm sâu dưới mặt đường khoảng `21,3 m` (ở `z ≈ −23,13 m`) phía trước-phải cách `49,7 m` |
| `s05f27` | Góc trên-trái trong khung top-down CVAT (cạnh trên trên ảnh BEV) | Giữa dải phân cách bên trái có tủ biến áp màu trắng nằm ngay trước một xe van trắng, rất dễ nhìn nhầm thành thân xe nếu không xem cả ảnh camera và lát cắt 3D |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh chụp từ CVAT: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f00`, `#1` | `Car` | Đúng (`270,0°`, mũi tên đỏ dọc thân hướng đầu xe theo `−y`) | Có (`z_bottom = −1,820 m`, `ground_gap = +0,02 m`) | Có (Top/Side/Front + `cam05`/`cam06`) | Có (`l = 4,539 m`, `w = 1,881 m`, `h = 1,562 m` nằm gọn trong khoảng Sedan `3,8–5,0 m × 1,70–1,90 m × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f00`, `#12` | `Bus` | Đúng (`359,0°`, mũi tên đỏ chỉ về đầu xe buýt cắt ngang hướng `+x`) | Có (`z_bottom = −1,801 m`, `ground_gap = 0,00 m`) | Có (Top/Side/Front + `cam07`/`cam08`) | Dài và rộng trong khoảng (`l = 11,272 m`, `w = 2,554 m` so với `10,0–12,5 m × 2,50–2,60 m`); cao `h = 2,802 m` hơi thấp hơn mốc `3,00–3,50 m` do là xe buýt đô thị sàn thấp, khớp sát đỉnh cụm điểm LiDAR |
| (c) người đi bộ hoặc xe hai bánh | `s01f00`, `#3` | `Cyclist` | Đúng (`283,5°`, mũi tên đỏ chỉ theo đầu xe máy và hướng lái) | Có (`z_bottom = −1,702 m`, `ground_gap = 0,00 m`) | Có (Top/Side/Front + `cam07`/`cam08`) | Có (`l = 1,882 m`, `w = 0,778 m`, `h = 1,484 m` nằm gọn trong khoảng Xe máy có người `1,8–2,3 m × 0,70–0,95 m × 1,40–1,80 m`) |

**Vòng hiệu chuẩn và các vật bỏ qua vì không đủ cơ sở:**
- **Vòng hiệu chuẩn:** Không khởi tạo task chung `HC485087` theo yêu cầu làm bài cá nhân; tự rà soát chặt chẽ từng tiêu chí của `docs/GUIDELINE.md` và bảng 4.3 trước khi vẽ 10 frame (đặc biệt chuẩn hóa ranh giới class giữa `Car` và `Truck` đối với SUV/jeep, xe van chở hàng và xe tải ben; tách người đi bộ đi cặp; loại trừ xe hai bánh dựng trống không có người lái; bật `occluded` cho các xe bị dải phân cách hoặc xe khác che một phần).
- **Các vật bỏ qua vì không đủ cơ sở:**
  - `s01f00`, phía trước giữa làn > `50 m` (`cam03`): các xe con chạy xa ngoài ranh giới `50 m` chỉ còn dưới 10 điểm rời rạc (`UNCERTAIN_SCOPE` / `LOW_VISIBILITY`).
  - `s01f51`, vỉa hè bên phải `32–36 m` (`cam03`): nhóm người đi bộ đứng chồng chéo sau cột đèn và thân cây, không tách rõ thân người và hướng mặt (`LOW_VISIBILITY`); dãy xe đạp dựng bên đường không có người lái (ngoài phạm vi class).
  - `s02f12`, vỉa hè phải `7–15 m` (`cam07`): hàng xe máy/xe điện đỗ phủ bạt sau rào sắt không có người điều khiển; một số xe chạy ngược chiều xa > `45 m` bị dải phân cách và xe phía trước che gần hết (`LOW_VISIBILITY`).
  - `s03f15`, góc ngã tư phía trước `38–45 m` (`cam03`): vài người đi bộ đứng xen vào dòng xe hai bánh, thể tích điểm thấy dưới 15–20% (`LOW_VISIBILITY`).
  - `s05f09`, nhánh đường ngang phía bên phải `35–47 m` (`cam07`/`cam08`): một số xe cắt ngang xếp chồng lên nhau ở khoảng cách xa, chỉ thu được lát cắt rất mỏng không đủ cơ sở ước lượng chiều dài (`LOW_VISIBILITY`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

Tất cả 4 hiện tượng đều quan sát trực tiếp trên 10 frame của task cá nhân `D13-2A202603148`. Mọi nguyên nhân dưới đây đều được nêu dưới dạng **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s03f15` / `t4-1-bong-che.png` / task `D13-2A202603148` | **Giả thuyết:** tường quây công trình hai bên, xe van chở hàng `#1` (`3,54 m`) và xe tải nhỏ `#2` (`7,51 m`) chắn trực tiếp chùm tia laser truyền thẳng từ LiDAR | Phía sau tường quây và phía sau xe tải `#2` xuất hiện vùng hình nón đen gần như trống điểm; xe SUV đỏ `#6` (`16,55 m`) đứng sau xe tải `#2` bị mất một phần mặt bên và đầu xe | Không coi vùng trống sau vật che là đường trống; với xe `#6` vẫn nhận dạng được qua `cam03`, bật thuộc tính `occluded = 1` và vẽ cuboid bao trọn phần thân xe suy luận được (`4,628 m × 1,884 m`) |
| 2. Thưa ở xa | `s01f00` / `t4-2-thua-o-xa.png` / task `D13-2A202603148` | **Giả thuyết:** các tia LiDAR 40 kênh tỏa ra theo góc cố định nên khi khoảng cách tăng, khoảng hở giữa các tia theo cả chiều ngang và chiều dọc tăng tỷ lệ với khoảng cách | Các vòng quét mặt đất giãn rất rộng ở `35–50 m`; xe con ở `39,6–45,0 m` (`#14–#18`) chỉ còn `34–74` điểm trên vài đường quét mỏng (so với `2639` điểm ở `3,52 m`), còn xe xa > `50 m` chỉ có vài điểm rời | Chỉ vẽ khi thấy khoảng 15–20% thể tích hoặc đủ cơ sở kết hợp ảnh camera và bảng 4.3 (`#14–#18`); các xe quá xa/quá thưa ngoài `50 m` bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f51` / `t4-3-phan-xa-thap.png` / task `D13-2A202603148` | **Giả thuyết:** lớp sơn đen bóng và bề mặt kính cửa trên xe sedan đen `s01f51 #1` (`6,57 m`) hấp thụ mạnh hoặc gây phản xạ gương tia hồng ngoại gần khiến năng lượng dội về cảm biến rất nhỏ | Xe sedan đen `#1` hiển thị xám tối và thưa/khuyết điểm vùng cửa kính với 72% số điểm có `intensity = 0` (trung bình `0,028`, phân vị 90% là `0,030`), thấp hơn rõ rệt so với xe sedan trắng `#2` ở tầm tương đương (`6,94 m`, trung bình `0,234`, phân vị 90% là `0,750`) | Giữ nguyên giải thích ở mức giả thuyết, không khẳng định chắc chắn; luôn đối chiếu ảnh camera (`cam05`/`cam06`) để bao trọn phần thân tối màu và vùng kính, tránh bóp hẹp cuboid theo phần điểm sáng |
| 4. Điểm lạ | `s05f09` / `t4-4-diem-la.png` / task `D13-2A202603148` | **Giả thuyết:** tia laser bị phản xạ đa đường (multipath) qua mặt đường/kính xe hoặc nhiễu đo thời gian bay (ToF) tạo thành điểm ảo có quãng đường dài hơn thực tế | Khi nghiêng khung 3D nhìn ngang mặt đường, thấy cụm **17 điểm** tách biệt hẳn bên dưới mặt phẳng đường tại `x ∈ [−9,06; −6,08] m`, `y ≈ −49,07 m`, `z ∈ [−23,24; −23,04] m` (trung bình `z ≈ −23,13 m`, thấp hơn mặt đường tới `21,3 m`) | Đây là sai lệch hình học rõ ràng, tuyệt đối **không gán thành vật thể** theo guideline và không dùng cụm điểm này để ước lượng mặt đường |

## 5. Số từ cuboid của tôi (Bước 5)

Toàn bộ số liệu dưới đây trích trực tiếp từ `submission/D13-2A202603148-bangA.csv` và `submission/D13-2A202603148-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f00`, `#1`, `Car` | `−1,039 m` | `1,562 m` | `−1,820 m` | `3,52 m` | `3,67 m` | `270,0°` | Chạm đường (`z đáy` lệch `0,120 m` so với mốc `−1,7 m`, `ground_gap_m = +0,02 m` so với mặt đường cục bộ) |
| `s01f00`, `#12`, `Bus` | `−0,400 m` | `2,802 m` | `−1,801 m` | `32,68 m` | `32,68 m` | `359,0°` | Chạm đường (`z đáy` lệch `0,101 m` so với mốc `−1,7 m`, `ground_gap_m = 0,00 m`) |
| `s03f42`, `#3`, `Car` | `−0,246 m` | `1,600 m` | `−1,046 m` | `19,03 m` | `19,04 m` | `270,0°` | Cao hơn mặt đường chính `0,654 m` do xe đỗ trên đường gom/vỉa hè nâng cao phía bên phải dải cây (`selfcheck` ước mặt đất lẫn bờ cỏ nên báo `ground_gap_m = −0,45 m`; thực tế hộp chạm đúng nền đường gom) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất:** `s01f00 #1` (`Car`) — khoảng cách ngang `dist_m = 3,52 m`, khoảng cách 3D `r = 3,67 m` (`x = 3,424 m`, `y = −0,820 m`, `z = −1,039 m`).
  - **Xa nhất:** `s04f37 #13` (`Car`) — khoảng cách ngang `dist_m = 47,06 m`, khoảng cách 3D `r = 47,07 m` (`x = −3,649 m`, `y = 46,914 m`, `z = −1,201 m`). Ở khoảng cách lớn, `dist_m` và `r` gần như trùng nhau vì `|z|` rất nhỏ so với khoảng cách ngang.
- **(b) So sánh kích thước với bảng 4.3:**
  - **Một người đi bộ (`s01f00 #8`, `Pedestrian`):** `l = 0,800 m`, `w = 0,730 m`, `h = 1,692 m` — cả 3 chiều đều nằm gọn trong khoảng Người lớn ở bảng 4.3 (`0,4–0,9 m × 0,5–0,9 m × 1,50–1,95 m`).
  - **Một xe con (`s01f00 #1`, `Car`):** `l = 4,539 m`, `w = 1,881 m`, `h = 1,562 m` — cả 3 chiều đều nằm gọn trong khoảng Sedan ở bảng 4.3 (`3,8–5,0 m × 1,70–1,90 m × 1,40–1,60 m`).
  - **Giải thích chiều nằm ngoài khoảng bảng 4.3 (nếu có trong bài):** Một vài hộp có chiều cao hơi thấp hơn mốc bảng 4.3 như xe buýt `s01f00 #12` (`h = 2,802 m` so với `3,00–3,50 m`) do là dòng buýt đô thị sàn thấp có thân thấp hơn xe khách đường dài, hoặc xe con xa `s01f00 #16` ở `40,34 m` (`h = 1,357 m` so với `1,40–1,60 m`) do ở khoảng cách xa tia laser quét thưa ở phần nóc và gầm xe bị che một phần bởi dải phân cách.
- **(c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:**
  - **Xe cùng chiều phía trước:** `s01f51 #4` (`Car`, chạy thẳng phía trước xe thu dữ liệu ở `10,27 m`): `yaw_deg = 270,1°`, rất sát `270°` (hướng `−y`).
  - **Xe đi ngược chiều:** `s02f12 #3` (`Car`, làn ngược chiều bên trái ở `10,44 m`): `yaw_deg = 90,5°` (tương tự `s04f37 #11` ở `37,88 m`: `yaw_deg = 89,5°`), rất sát `90°` (hướng `+y`).

**Mật độ theo khoảng cách (4 cuboid `Car` không bị che khuất ở 4 khoảng `dist_m` khác nhau):**

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f00`, `#1`, `Car` | `3,52 m` | `2639` điểm |
| `s01f51`, `#4`, `Car` | `10,27 m` | `904` điểm |
| `s02f12`, `#10`, `Car` | `20,28 m` | `320` điểm |
| `s05f09`, `#14`, `Car` | `32,55 m` | `84` điểm |

**Quy luật mật độ và so sánh với slide bài giảng:** Khi khoảng cách tăng gấp **~2 lần** (từ `10,27 m` lên `20,28 m`), số điểm LiDAR trong cuboid xe con giảm từ `904` xuống `320` điểm (còn khoảng **1/3**), và khi xa gấp **~3,2 lần** (lên `32,55 m`), số điểm chỉ còn `84` điểm (còn khoảng **1/11**), tuân theo quy luật giảm nhanh gần theo $1/d^2$ do các tia laser phân kỳ theo cả phương ngang và phương đứng. Kết quả này rất khớp với đường cong trung vị trên slide bài giảng (xe con: khoảng `1968` điểm ở `0–10 m`, `718` điểm ở `10–20 m`, `214` điểm ở `20–30 m`, `72` điểm ở `30–40 m`, `19` điểm ở `60–80 m`).

**Tự kiểm sau `preview` và `selfcheck`:**
- **Các cuboid đã sửa rồi chạy lại `sync`:**
  1. `s05f09 #9` (`Car`, `26,31 m`): kiểm ảnh `preview` (`cam09`) phát hiện hộp bị ngược đầu 180° (`270,0°` trong khi đèn hậu hướng về xe thu dữ liệu trên làn ngược chiều) và hơi ngắn (`3,99 m × 1,65 m`); đã xoay về `yaw_deg = 90,0°` và chỉnh kích thước thành `4,520 m × 1,860 m` (số điểm tăng từ `129` lên `163` điểm).
  2. `s03f42 #5` và `#6` (`Car`, `26,03 m` và `28,82 m`): `selfcheck` báo cờ chồng lấn giữa hộp 5 và hộp 6; đã thu gọn chiều dài về `4,350 m` và tách tâm dọc theo trục `y` để xóa hoàn toàn chồng lấn (`BEV-IoU = 0`).
  3. `s04f37 #1` (`Car`, `6,72 m`) và `s05f09 #1` (`Car`, `7,55 m`): `selfcheck` báo cờ `867` và `783` điểm sát ngoài mép; kiểm lại thấy tâm hộp hơi lệch lên trước so với đuôi xe, đã dịch tâm về phía sau và chỉnh kích thước SUV/MPV nên số điểm trong hộp tăng lần lượt từ `1907 → 2779` điểm và `2032 → 2862` điểm, hết cảnh báo.
  4. `s01f51 #13` (`Pedestrian`, `29,22 m`): sau khi thu gọn hộp về `0,650 m × 0,650 m`, `selfcheck` báo đáy thấp hơn mặt đường `0,40 m`; đã nâng cao độ `z` thêm `0,35 m` để chạm đúng mặt vỉa hè (`ground_gap_m = −0,05 m`), hết cảnh báo.
- **Các cảnh báo giữ nguyên (`15/135` cuboid) và lý do:**
  - Nhóm `Cyclist` / `Pedestrian` có cờ điểm sát ngoài mép (`s01f00 #10`, `s01f51 #5`, `s02f12 #7, #8, #9, #11`, `s03f15 #14`, `s05f27 #5`): các đối tượng nhỏ di chuyển sát hàng rào sắt, bụi cây (`s01f51 #5` cạnh dải cây có `145` điểm lá sát mép) hoặc gần cột/xe khác; hộp đã bám đúng kích thước bảng 4.3 nên không nới hộp bao vào chướng ngại vật bên cạnh.
  - Nhóm xe xa xếp hàng sát nhau hoặc sát cọc tiêu (`s01f00 #16, #17`, `s01f51 #14`, `s02f12 #16`): các điểm ngoài mép thuộc xe liền kề hoặc trụ chắn, đã xác minh bằng `preview`.
  - Nhóm xe đỗ sau gờ cỏ ở `s03f42` (`#3`, `#5`, `#6`): vòng ước lượng mặt đất `4 m` của `selfcheck` bị ảnh hưởng bởi bờ đất dốc và tán cây ngay cạnh đường gom, giữ nguyên vị trí khớp ảnh camera.

**Đổi kiểm:** Đã đổi kiểm trực tiếp trên màn hình với bạn ngồi cạnh có MSSV **`2A202603107`** (không gửi file bảng A cho nhau). Hai con số được bạn chọn đối chiếu trực tiếp với `submission/D13-2A202603148-bangA.csv`: (1) cuboid `s01f00 #1` (`Car`) có `z_bottom = −1,820 m`, `dist_m = 3,52 m`; (2) cuboid `s01f51 #4` (`Car`) có `yaw_deg = 270,1°`, `dist_m = 10,27 m`. Kết quả: cả hai con số đều **khớp** hoàn toàn với bảng A, không có sai lệch cần sửa.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Từ phía mô hình học trên nhãn 3D:**
  - *Vì sao nhãn 3D đo bằng mét chứ không phải pixel:* Mô hình phát hiện vật thể 3D và hệ thống lập kế hoạch chuyển động của xe tự lái cần dự đoán chính xác vị trí không gian thực `(x, y, z)`, kích thước vật lý `(l, w, h)` và hướng di chuyển `yaw` bằng **mét** trong hệ tọa độ xe để tính khoảng cách an toàn và tránh va chạm — những đại lượng này bất biến với tiêu cự hay độ méo ống kính, trong khi kích thước pixel thay đổi theo khoảng cách và từng camera.
  - *Vì sao vật thưa điểm cần quy tắc riêng:* Nếu cố gán nhãn bằng phỏng đoán khi cụm điểm quá thưa (dưới 15–20% thể tích, không đủ căn cứ), tập huấn luyện sẽ nhiễm sai số lớn về tâm, kích thước và góc hướng khiến mô hình khó hội tụ và sinh nhiều dự đoán ảo (false positive); quy tắc ngưỡng tối thiểu kết hợp ghi nhận `LOW_VISIBILITY` giúp giữ tập nhãn sạch và nhất quán về ranh giới nhận biết.
- **(b) Cách dùng AI:**
  - **Công cụ:** ChatGPT / trợ lý AI.
  - **Câu hỏi chính:** Hỏi về lệnh PowerShell/Docker bật CVAT local, quy ước hệ tọa độ VF (`+x` trái, `+y` sau, `yaw = 270°` đi thẳng), phím tắt giao diện CVAT 3D và ý nghĩa các khái niệm `BEV-IoU`, `intensity`, `z_bottom`, `ground_gap_m`.
  - **Cách dùng kết quả:** Dùng hướng dẫn lệnh và khái niệm để thao tác CVAT 3D và đọc kết quả `preview`/`selfcheck` nhanh hơn; không gửi dữ liệu VF, ảnh camera hay bảng A cho AI, toàn bộ cuboid được kiểm tra trực tiếp trên CVAT và xuất qua `python scripts/lab.py sync`.
- **(c) Điều còn chưa chắc:**
  - Ba xe con đỗ sau gờ cỏ và hàng cây bên phải ở `s03f42` (`#3`, `#5`, `#6`) bị che khuất khá nhiều nên độ cao đáy thực trên nền đường gom và vị trí chiều dài chính xác còn có độ bất định lớn hơn các xe trên làn chính.
  - Một số xe van nhỏ ở khoảng cách trên `30 m` (như `s01f51 #15` ở `34,97 m` và `s03f15 #15` ở `38,08 m`) có thể nằm ở ranh giới giữa van chở hàng (`Truck`) và van chở khách (`Car`) khi chỉ nhìn thấy mặt sau/trước từ xa (`UNCERTAIN_CLASS`).
  - Hướng di chuyển của các người đi bộ đứng hoặc đi sát rìa xa (~`22–29 m`) và chiều dài phần đầu bị che khuất của các xe xếp hàng dài ở `35–45 m` có thể còn lệch nhẹ so với kích thước thật.
