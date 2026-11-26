# Báo cáo Day 13 — Doãn Tùng Lâm, 2A202603227


## 1. Kết luận trước
- Mã số: 2A202603227. Task CVAT: `D13-2A202603227` (task #23, job #29)
- 10 frame: s01f23, s01f48, s02f03, s02f35, s03f13, s03f55, s04f24, s04f57, s05f15, s05f46
- Môi trường: Windows 11, CVAT 2.81.0 chạy local (`http://localhost:8080`).
-  Trình duyệt: Google Chrome
- MSSV bạn ngồi cạnh (đổi kiểm): Tự kiểm

Tôi vẽ tổng cộng **65 cuboid** (54 Car, 6 Cyclist, 3 Truck, 2 Bus). Frame đầu s01f23 có 9 hộp. Ít nhất là s02f03, s03f55 và s04f57, mỗi frame 5 hộp: ở các đoạn đường này trong 30 m chỉ có ít xe, còn các cụm ở xa quá thưa hoặc bị che nên tôi không đủ cơ sở để vẽ. Sau khi sửa và chạy lại `sync`, `selfcheck` cờ **0/65 cuboid** (lần đầu cờ 3/65).

Tôi tin nhất ở **hướng** (xe cùng chiều ≈ 270°, ngược chiều ≈ 90°, đối chiếu với ảnh camera) và **đáy hộp** của các xe gần. Phần yếu nhất là **vật xa trên 30 m**: kích thước dễ sai vì ít điểm, và hướng của hai hộp xa ở s05f46 (idx 5, 6) chỉ suy ra từ hình dạng cụm điểm. Phần yếu thứ hai là hướng mũi tên của các xe hai bánh.

Cách dùng CVAT:
- **Đổi góc nhìn:** cuộn chuột để phóng to/thu nhỏ; kéo chuột trái để xoay (kéo lên để nghiêng từ nhìn từ trên xuống sang nhìn ngang); dùng các phím `U I O / J K L` và mũi tên trên khung 3D để xoay và dời camera.
- **Vẽ và chỉnh cuboid:** chọn công cụ cuboid, chọn label, chọn Shape (không dùng Track); đặt hộp vào cụm điểm, nháy đúp để chọn hộp, rồi kéo các điểm đỏ ở ba khung Top/Side/Front để căn sát cụm điểm và chỉnh L/W/H ở Object details. Bấm Save sau khi sửa.
- **Mũi tên hướng và ảnh camera:** bật `Cuboid orientation` ở mục Appearance để thấy mũi tên đầu xe; CVAT 2.81 không chiếu cuboid lên ảnh camera, nên tôi so với 3 khung camera bên phải và dùng `python scripts/lab.py preview` để xem cuboid trên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-<frame>-3d`, `t1-<frame>-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).


| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f23 | Trong ảnh `t1-s01f23-top`: phía trước (−y) ở cạnh **trái** của ảnh; +y chỉ sang **phải** (về phía sau xe); +x chỉ xuống **dưới** ảnh (bên trái xe). Kiểm bằng xe buýt (x = +6,5; y = +12,0) nằm dưới-phải gốc tọa độ, hai xe máy (y ≈ −14 đến −16) nằm trên-trái; khớp cam01 nhìn về phía trước. | Phẳng (các vòng quét đều quanh xe), thấp hơn LiDAR ~1,8 m (z mặt đường ≈ −1,75 m) | Xe gần nhất rõ ~10 m; xe xa còn nhận ra ~30 m (ước lượng) | Xe con (xe trắng), xe buýt, xe máy/xe hai bánh gần vạch qua đường ở phía trước, cây và dải phân cách | Xe con bên trái và bên phải xe thu dữ liệu; xe buýt ở sau-trái, xe con ở sau | Cụm điểm rời ở vạch qua đường phía trước và các điểm sát hàng cây hai bên  |
| s01f48 | Trong ảnh `t1-s01f48-top` (khung nhìn bị xoay ~20° so với s01f23): phía trước (−y) chỉ về góc **trên-trái**; +y chỉ sang **phải, hơi xuống**; +x chỉ xuống **dưới-trái**. Kiểm bằng xe ngay sau (y = +7,0) nằm bên phải gốc, xe đạp (x = −10,0; y = +11,4) nằm trên-phải | Phẳng, thấp hơn LiDAR ~1,8 m (z mặt đường ≈ −1,8 m) | Xe phía trước ~10–20 m rõ; xe xa còn nhận ra ~30 m (ước lượng) | Xe con (trắng, có xe điện), xe đạp trên làn riêng bên phải, cây, dải phân cách | Xe con ngay phía sau; xe đạp bên phải; xe xa phía sau | Dải điểm dày dọc hai mép đường (cây, hàng rào) |

- 3 điều **chỉ camera** cho biết: màu sắc của xe, đèn giao thông và biển chỉ dẫn (đọc được trạng thái, nội dung), vạch kẻ đường và mũi tên sơn trên mặt đường.
- 3 điều **chỉ LiDAR** cho biết: khoảng cách và vị trí chính xác bằng mét, hình dạng 3D và độ cao của vật (z), vật ở góc mù của 7 camera (nhìn được 360°).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f03 | Góc **trên-phải** (−y); +x chỉ lên trên-trái, +y xuống dưới-trái. Kiểm bằng xe buýt (x = +2,6; y = +5,5) nằm bên trái gốc |  |
| s02f35 | Cạnh **trái**, hơi chếch lên (−y); +y sang phải hơi xuống, +x xuống dưới. Kiểm bằng xe con (x = +4,5; y = −3,8) nằm dưới-trái gốc |  |
| s03f13 | Góc **trên-phải** (−y); +x chỉ lên trên, +y xuống dưới-trái. Kiểm bằng xe con (x = +2,8; y = −2,0) nằm trên-phải gốc | |
| s03f55 | Cạnh **trái** (−y); +y sang phải, +x xuống dưới (giống s01f23). Kiểm bằng xe con phía trước (y = −8,3) nằm bên trái gốc | |
| s04f24 | Góc **trên-trái** (−y); +y xuống dưới-phải, +x xuống dưới-trái. Kiểm bằng SUV (x = +7,4; y ≈ 0) nằm dưới-trái gốc |  |
| s04f57 | Cạnh **trái** (−y); +y sang phải, +x xuống dưới. Kiểm bằng xe con (x = −2,6; y = −9,4) nằm trên-trái gốc | |
| s05f15 | Cạnh **trái**, hơi chếch lên (−y); +y sang phải hơi xuống, +x xuống dưới. Kiểm bằng SUV (x = −4,5; y = −0,9) nằm phía trên gốc |  |
| s05f46 | Cạnh **trên**, hơi chếch phải (−y); +y xuống dưới, +x sang trái. Kiểm bằng xe con (x = +7,6; y ≈ 0) nằm bên trái gốc |  |



## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f23, idx 4 (tâm −3,2; 5,9; cách 6,7 m) | Car | Đúng: yaw 269°, gần 270° cho xe cùng chiều; mũi tên khớp đầu xe | Chạm đường: z đáy −1,75 m | Đã kiểm trên Top/Side/Front và ảnh camera (`t2-a-car`) | Trong khoảng SUV/van: 5,24 × 2,10 × 1,68 m (bảng: 4,4–5,3 × 1,80–2,10 × 1,65–2,10) |
| (b) xe buýt hoặc xe tải | s01f23, idx 2 (tâm 6,5; 12,0; cách 13,6 m) | Bus | Đúng: yaw 270,5°, cùng chiều | Chạm đường: z đáy −1,68 m | Đã kiểm trên Top/Side/Front và ảnh camera (`t2-b-bus-truck`) | Dài 12,26 m trong khoảng 10,0–12,5 m; rộng 2,84 m vượt 2,50–2,60 m vì tôi bao cả gương chiếu hậu chìa ra hai bên, theo quy tắc bao trọn phần chìa ra của guideline; cao 3,34 m trong khoảng 3,00–3,50 m |
| (c) người đi bộ hoặc xe hai bánh | s01f23, idx 3 (tâm −7,1; −13,9; cách 15,6 m) | Cyclist | Yaw 20°, xe hai bánh đang đi ngang theo hướng +x, không cùng chiều xe thu dữ liệu; cần xác nhận mũi tên với ảnh camera | Chạm đường: z đáy −1,73 m | Đã kiểm trên Top/Side/Front và ảnh camera (`t2-c-ped-cyclist`) | Xe máy có người 1,8–2,3 × 0,70–0,95 × 1,40–1,80 m: dài 2,07 và cao 1,70 trong khoảng, rộng 0,65 m hơi hẹp (trong sai số ±10%) |


**Vòng hiệu chuẩn:** Tôi không tham gia vòng hiệu chuẩn trên task chung `D13-HC485087`, nên không có số liệu so với các bạn. Thay vào đó, tôi tự kiểm 3 cuboid mẫu ở bảng trên theo guideline: yaw (cùng chiều ≈ 270°), z đáy (so với mặt đường quanh xe ≈ −1,7 m) và kích thước (so với bảng 4.3). Điểm tôi chưa chắc khi tự hiểu guideline là quy tắc "bao trọn phần chìa ra": tôi bao cả gương xe buýt nên chiều rộng 2,84 m vượt khoảng 2,50–2,60 m. Tôi chưa đối chiếu cách hiểu này với coach hay các bạn.

**Vật bỏ qua vì không đủ cơ sở:**
- s01f23, phía sau, cách ~40 m: cụm điểm xe con rất thưa, mặt đường ở đó dốc (z ≈ −2,3 m) nên không xác định được đáy và kích thước. Tôi đã vẽ rồi xoá. **LOW_VISIBILITY**.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s02f03, ảnh `t4-1-bong-che`, frame của tôi | Xe buýt đỏ sát bên trái xe thu dữ liệu (idx 1, cách 7,0 m, dài 12 m) chặn tia LiDAR, nên vùng phía sau xe buýt không có tia tới | Vùng phía sau xe buýt (x từ 6 đến 12 m, y từ 0 đến 12 m) chỉ có 149 điểm, trong khi vùng cùng kích thước ở bên phải xe có 4264 điểm và ở phía trước-trái có 1733 điểm. Phía sau xe buýt gần như trống | Vùng trống này là **chưa quan sát được**, không phải đường trống, nên không vẽ vật nào trong đó. Nếu có xe bị che một phần thì phải bật occluded và chỉ bao phần suy luận được, không đoán toàn bộ |
| 2. Thưa ở xa | s01f23, ảnh `t4-2-thua-o-xa`, frame của tôi | Mật độ tia LiDAR giảm khi xa, và mỗi tia chiếu lên xe một diện tích nhỏ hơn | Car idx 9 cách 37,3 m (phía sau) chỉ còn 94 điểm, trong khi Car idx 1 cách 4,2 m có 3004 điểm (gấp khoảng 32 lần) | Còn đủ hình dạng xe nên vẫn vẽ (theo guideline vẽ nếu thấy khoảng 15–20% thể tích hoặc suy luận được). Vật xa hơn nữa mà chỉ còn vài điểm thì ghi LOW_VISIBILITY và bỏ qua. Kích thước ở xa dễ sai nên ghi là chưa chắc |
| 3. Phản xạ thấp / ít điểm bất thường | s03f13, ảnh `t4-3-phan-xa-thap`, frame của tôi | Xe đen bóng (Peugeot hatchback màu đen, thấy trong cam06) hấp thụ tia hoặc phản xạ kính ra hướng khác, nên điểm ít và cường độ thấp| Car idx 1 cách 3,70 m chỉ có 1611 điểm và `intensity` trung bình 0,007. Car idx 1 ở s01f23 (xe sáng màu, cách 4,18 m, xa hơn) có 3004 điểm và `intensity` 0,242: điểm ít hơn khoảng một nửa và intensity thấp hơn khoảng 35 lần | Xe ở gần nên vẫn đủ điểm để vẽ. Dùng ảnh camera để biết đó là xe, không nới hộp theo `intensity`. Nếu xe đen ở xa thì rất dễ rơi xuống mức LOW_VISIBILITY, và chỉ ghi giả thuyết |
| 4. Điểm lạ | s02f35, ảnh `t4-4-diem-la`, frame của tôi (cũng là frame gợi ý của lab) | Chưa rõ, có thể là phản xạ nhiều đường (tia dội qua mặt kính hoặc kim loại bóng) hoặc nhiễu cảm biến | 50 điểm ở z từ −13,49 đến −10,24 m, x từ 3,0 đến 7,5 m, y từ 28,0 đến 29,6 m (cách khoảng 28,5 m, phía sau-trái). Mặt đường quanh đó ở z khoảng −1,7 m, nên cụm nằm dưới mặt đường khoảng 9–12 m. Cường độ trung bình 4,3, cao hơn hẳn điểm của xe (khoảng 0,1–0,2) | Điểm dưới mặt đường là sai hình học chắc chắn, nên **không gán thành vật** và không vẽ cuboid cho cụm này. Chỉ ghi nhận trong báo cáo |

## 5. Số từ cuboid của tôi (Bước 5)

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s03f13, idx 1, Car | −1,058 | 1,434 | −1,775 | 3,70 | 3,85 | 270,0 | Chạm đường (khoảng hở với mặt đường 0,02 m) |
| s01f23, idx 2, Bus | −0,013 | 3,336 | −1,681 | 13,61 | 13,61 | 270,5 | Chạm đường (0,07 m) |
| s01f48, idx 8, Cyclist | −0,447 | 1,600 | −1,247 | 15,1 | 15,1 | 270,0 | Z đáy cao hơn mốc −1,7 m tới 0,45 m, nhưng làn xe đạp ở đây cao hơn làn xe nên `selfcheck` đo chỉ lệch 0,07 m so với mặt đường tại chỗ: chạm đường |
| s04f57, idx 4, Car | −1,379 | 1,400 | −2,079 | 27,7 | 27,7 | 90,0 | Z đáy thấp hơn mốc −1,7 m khoảng 0,38 m vì đường dốc xuống ở xa; `selfcheck` đo lệch 0,10 m so với mặt đường tại chỗ: chạm đường |
| s05f46, idx 3, Truck | −0,619 | 3,021 | −2,130 | 17,7 | 17,7 | 90,0 | Mặt đường ở bên +x thấp hơn, z đáy −2,13 m; `selfcheck` đo lệch 0,08 m: chạm đường |

(a) Cuboid **gần nhất** là s03f13 idx 1 (Car): `dist_m` = 3,70 m, r = 3,85 m. Cuboid **xa nhất** là s05f46 idx 6 (Truck): `dist_m` = 42,84 m, r = 42,85 m.

(b) Xe hai bánh (s01f23, idx 3, Cyclist) dài 2,07 m, rộng 0,65 m, cao 1,70 m. So với dòng "xe máy có người" của bảng 4.3 (1,8–2,3 × 0,70–0,95 × 1,40–1,80 m), dài và cao nằm trong khoảng, chỉ có **chiều rộng hơi hẹp** (0,65 m, lệch khoảng 7%, trong sai số ±10% của guideline), vì cụm điểm của người ngồi sát thân xe nên điểm ở hai bên thưa. Xe con (s01f23, idx 4, Car) dài 5,24 m, rộng 2,10 m, cao 1,68 m. So với dòng "SUV, minivan" (4,4–5,3 × 1,80–2,10 × 1,65–2,10 m) cả ba chiều đều nằm trong khoảng, nhưng sát trần ở dài và rộng, nên xe đó là SUV chứ không phải sedan.

(c) Xe cùng chiều phía trước, s01f23 idx 1 (Car), có `yaw_deg` = 269,1°, gần 270°. Xe ngược chiều ở làn bên +x, s05f15 idx 5 (Car), có `yaw_deg` = 90,0°, đúng 90°. Cả hai khớp quy ước của lab (cùng chiều ≈ 270°, ngược chiều ≈ 90°).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f48, idx 4, Car | 7,0 | 1433 |
| s01f48, idx 3, Car | 16,3 | 405 |
| s01f48, idx 7, Car | 22,7 | 313 |
| s02f03, idx 5, Car | 36,6 | 111 |

Quy luật: xa gấp khoảng 5 lần (từ 7,0 m ra 36,6 m) thì số điểm còn khoảng 1/13 (1433 xuống 111), tức giảm nhanh hơn tỉ lệ nghịch với khoảng cách (số mũ khoảng 1,5). So với đường cong ở slide (xe con: khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m), số liệu cùng cỡ ở gần (1433 so với 1968), thấp hơn ở 16 m (405 so với 718), và cao hơn ở xa (111 so với 72 ở 30–40 m). Nguyên nhân có thể là hộp vẽ hơi rộng hơn xe thật ở xa, nên gom thêm điểm nền

Phần tự kiểm: `selfcheck` lần đầu cờ 3 cuboid trên 65. Tôi đã sửa ba hộp rồi chạy lại `sync`, sau đó không còn cờ nào. Các hộp đã sửa là: s05f15 idx 6 (xe van chở người, giảm chiều cao từ 2,38 m xuống 2,25 m cho đúng khoảng cao của xe van), s05f46 idx 2 và s05f46 idx 4 (hai xe đứng sát dải phân cách, nâng đáy hộp: khoảng hở với mặt đường từ −0,37 m và −0,35 m về −0,24 m). Sau `sync` vẫn còn vài cuboid có khoảng hở đáy trên 0,2 m (s05f46 idx 5: 0,34 m; s05f15 idx 6: 0,30 m; s02f35 idx 1: 0,25 m; hai hộp s05f46 idx 2, 4: −0,24 m). Các hộp này **giữ nguyên** vì chưa dưới ngưỡng cờ của công cụ (0,45 m nổi, 0,25 m chìm) và mặt đường ở đó không phẳng (sát dải phân cách, đường dốc)

**Đổi kiểm:** Tôi làm bài và tự kiểm hoàn toàn, không có bạn kiểm chéo. Thay vào đó, tôi tự đối chiếu hai con số của bảng A với CVAT cho cuboid s01f23 idx 2 (Bus): chiều dài 12,26 m và yaw 270,5° trong bảng A khớp với L ở Object details và mũi tên hướng (Cuboid orientation) trên CVAT. Không có chỗ lệch nên không phải sửa.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
**(a)** Model 3D (ví dụ dùng cho xe tự lái) phải dự đoán vật nằm ở đâu và to bao nhiêu trong không gian thật để lập kế hoạch phanh, né, nên nhãn phải đo bằng mét trong hệ tọa độ LiDAR. Pixel thì thay đổi theo khoảng cách và theo camera, nên không cho biết xe cách bao xa hay dài bao nhiêu. Vật thưa điểm cần quy tắc riêng (vẽ khi thấy khoảng 15–20% thể tích, nếu không thì ghi LOW_VISIBILITY): nếu người gán đoán bừa hộp cho vài điểm lẻ, model sẽ học theo nhãn sai, hoặc học "bịa" ra vật ở nơi không có đủ dữ liệu. Còn nếu bỏ hết vật xa thì model không bao giờ học được cách nhận vật ở xa.

**(b)** Có dùng AI:
- **Công cụ:** Claude.
- **Câu hỏi chính:** giải thích hệ tọa độ, `idx`, `dist_m`, `r`; soạn nháp các mục của báo cáo.
- **Cách tôi dùng kết quả:** frame 1 tôi tự vẽ. Với cuboid của frame 2–10 do AI đề xuất, tôi mở từng frame trong CVAT để xem lại, đối chiếu ảnh camera, xoá hộp xe ở xa không đủ cơ sở, sửa các hộp bị `selfcheck` cờ rồi Save và chạy lại `sync`. Các số trong báo cáo lấy từ bảng A và phiếu tự kiểm do công cụ của lab xuất ra; phần nhận xét tôi đọc lại và sửa theo những gì tôi thấy trên CVAT.

**(c)** Điều còn chưa chắc:
- Hướng của hai hộp xa nhất ở s05f46 (idx 5 và 6, cách hơn 35 m) chỉ suy từ hình dạng cụm điểm thưa, có thể ngược đầu.
- Mũi tên hướng của các xe hai bánh (ví dụ s01f23 idx 3) khó xác định vì cụm điểm nhỏ, gần như đối xứng.
- Vài hộp vẫn hở đáy hơn 0,2 m (s05f46 idx 5, s05f15 idx 6, s02f35 idx 1) ở chỗ mặt đường không phẳng; chưa chắc là do hộp sai hay do mặt đường ở đó thực sự cao/thấp hơn.
- Kích thước các xe trên 30 m có thể lệch quá ±5% vì chỉ thấy một mặt xe.
- Xe buýt s01f23 idx 2 rộng 2,84 m do bao cả gương; tôi hiểu guideline là bao cả phần chìa ra, nhưng chưa đối chiếu với cách hiểu của cả lớp ở vòng hiệu chuẩn.
