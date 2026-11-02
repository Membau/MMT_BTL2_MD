# CÁC KỊCH BẢN TẢI CAO ĐIỂM VÀ MÔ HÌNH TĂNG TRƯỞNG MẠNG BỆNH VIỆN
**Tác giả:** Bảo (Băng thông, Thiết bị & Công nghệ WAN)  
**Phối hợp:** Phước (Kiến trúc & IP), Toàn (Bảo mật), Trung (Mô phỏng & Cấu hình)  
**Tài liệu chuẩn:** Network Contract v1.1  

---

## 1. Giới thiệu và Phương pháp luận xây dựng kịch bản

Hệ thống mạng của một Bệnh viện Đa khoa Chuyên sâu có tính chất đặc thù:
- **Tính nhạy cảm với độ trễ (Latency Sensitive):** Hệ thống chỉ định thuốc, xét nghiệm khẩn cấp (Emergency Orders), tín hiệu monitor bệnh nhân và hội chẩn phẫu thuật từ xa không được phép nghẽn mạch.
- **Tính bùng phát tải (Traffic Bursts):** Hình ảnh chẩn đoán y khoa kỹ thuật số (DICOM, CT Scanner, MRI 128 lát cắt) tạo ra các luồng dữ liệu tức thời từ vài chục MB đến hàng trăm MB.
- **Chu kỳ sinh học và hành chính:** Hoạt động tiếp nhận bệnh nhân tập trung vào đầu giờ sáng và đầu giờ chiều.

Do đó, để đảm bảo tính an toàn và khả thi cao trong thiết kế hạ tầng, chúng tôi xây dựng **3 Kịch bản Tải (Scenarios)** đại diện cho các điều kiện vận hành thực tế:
1. **Kịch bản 1: Tải thấp (Low / Off-Peak Scenario)** – Ngoài giờ hành chính và ca trực đêm.
2. **Kịch bản 2: Tải trung bình (Medium / Baseline Standard Scenario)** – Dựa trên chuẩn số liệu đề bài (80% lưu lượng trong 3h cao điểm).
3. **Kịch bản 3: Tải cao / Cực hạn (High / Worst-Case & Growth Scenario)** – Kết hợp tăng trưởng 5 năm (+20%), bùng phát lưu lượng ảnh y tế PACS, truyền dữ liệu camera và 500 khách hàng truy cập Wi-Fi đồng thời.

---

## 2. Chi tiết các Kịch bản và Công thức tính toán

### 2.1 Kịch bản 1: Tải Thấp (Off-Peak Scenario - Ban đêm & Ngoài giờ khám)

#### Giả định kịch bản:
- Thời gian áp dụng: 21 giờ không cao điểm (từ 16:00 đến 09:00 sáng hôm sau và từ 11:00 đến 15:00).
- Hoạt động chủ yếu:
  - Chỉ có các khoa trực cấp cứu, hồi sức tích cực (ICU), phòng trực nội trú và các tiến trình tự động (Data backup ban đêm, Server batch processing, Syslog, NTP).
  - Tỷ lệ lưu lượng phát sinh: **20% tổng lưu lượng cả ngày**.
  - Thiết bị khách (Guest Wi-Fi) hầu như không hoạt động.

#### Công thức tính toán:
```
                20% × Tổng dung lượng cả ngày (MB) × 8
R_offpeak = ---------------------------------------------  (Mbps)
                     21 giờ × 3.600 giây (75.600 s)
```

#### Kết quả tính toán Kịch bản 1:
- **Main Site:**
  - Dung lượng trong 21h: DL = 60.300 MB; UL = 12.400 MB.
  - Thông lượng: DL = **6,38 Mbps**; UL = **1,31 Mbps**; Tổng = **7,69 Mbps**.
- **Site DBP / BHTQ (mỗi site):**
  - Dung lượng trong 21h: DL = 26.040 MB; UL = 5.280 MB.
  - Thông lượng: DL = **2,76 Mbps**; UL = **0,56 Mbps**; Tổng = **3,32 Mbps**.
- **Toàn Viện (Aggregate):**
  - Thông lượng Internet/Core: DL = **11,89 Mbps**; UL = **2,43 Mbps**; Tổng = **14,32 Mbps**.

*Đánh giá:* Kịch bản tải thấp tiêu thụ chưa đến 3% công suất kênh truyền. Đây là khung giờ vàng lý tưởng để hệ thống tự động backup dữ liệu lớn từ Server Farm sang hệ thống lưu trữ dự phòng hoặc Offsite Cloud.

---

### 2.2 Kịch bản 2: Tải Trung Bình (Baseline Standard Scenario - Theo đề bài)

#### Giả định kịch bản:
- Thời gian áp dụng: Khung giờ cao điểm quy định trong đề bài (09:00 – 11:00 và 15:00 – 16:00, tổng T = 3 giờ = 10.800 giây).
- Quy mô: Đầy đủ 1.120 Workstation, 14 Server.
- Tỷ lệ phân bổ: **80% tổng lưu lượng ngày diễn ra dồn dập trong 3 giờ này**.
- Lưu lượng mỗi Workstation: 500 MB DL/ngày, 100 MB UL/ngày.
- Lưu lượng Server: 1.000 MB DL/ngày, 2.000 MB UL/ngày.
- Lưu lượng Guest: 500 MB DL/ngày (tính theo mức tối thiểu đề bài).

#### Công thức tính toán:
```
                     80% × Tổng dung lượng (MB) × 8
R_peak-baseline = ------------------------------------  (Mbps)
                     3 giờ × 3.600 giây (10.800 s)
```

#### Kết quả tính toán Kịch bản 2:
- **Main Site:**
  - DL = (0,80 × 301.500 × 8) / 10.800 = **178,67 Mbps**.
  - UL = (0,80 × 62.000 × 8) / 10.800 = **36,74 Mbps**.
  - Tổng thông lượng Main: **215,41 Mbps**.
- **Site DBP / BHTQ (mỗi site - lưu lượng Leased Line WAN):**
  - DL = (0,80 × 130.200 × 8) / 10.800 = **77,16 Mbps**.
  - UL = (0,80 × 26.400 × 8) / 10.800 = **15,64 Mbps**.
  - Tổng thông lượng Leased Line mỗi nhánh: **92,80 Mbps**.
- **Toàn Viện (Aggregate Internet Edge):**
  - DL = **332,98 Mbps**.
  - UL = **68,03 Mbps**.
  - Tổng thông lượng Internet tại Edge: **401,01 Mbps**.

*Đánh giá:* Kịch bản 2 phản ánh trực tiếp yêu cầu đề tài. Kênh Leased Line 100 Mbps cho DBP/BHTQ sẽ hoạt động ở mức ~93% tải, và 2 đường Internet DSL (tổng 600–1000 Mbps) đáp ứng rất tốt.

---

### 2.3 Kịch bản 3: Tải Cao / Cực Hạn (High / Worst-Case & Growth Scenario)

#### Giả định kịch bản:
Kịch bản này được thiết kế để kiểm tra sức chịu đựng (Stress test) của hạ tầng mạng trong tương lai 5 năm khi đạt đỉnh tải kép:
1. **Tăng trưởng 5 năm (+20%):**
   - Số workstation tăng từ 1.120 lên **1.344 máy** (Main: 720 máy, DBP: 312 máy, BHTQ: 312 máy).
   - Lưu lượng nghiệp vụ tăng 20%.
2. **Khách hàng vãng lai (Guest Wi-Fi Spike):**
   - Thay vì chỉ 500 MB/ngày tổng cộng, giả định trong giờ cao điểm có **500 thiết bị di động** của bệnh nhân/thân nhân truy cập đồng thời.
   - Mỗi khách hàng được cấp băng thông giới hạn (Rate-limiting) là 1 Mbps DL và 256 kbps UL.
   - Tỷ lệ hoạt động đồng thời (Concurrency Ratio) là 40% (200 khách truyền dữ liệu cùng lúc):
```
Traffic_Guest-Peak_DL = 200 user × 1 Mbps = 200 Mbps DL
Traffic_Guest-Peak_UL = 200 user × 0,256 Mbps ≈ 51,2 Mbps UL
```
3. **Bùng phát truyền ảnh y tế RIS-PACS (PACS Imaging Burst):**
   - Các khoa Chẩn đoán hình ảnh tại DBP và BHTQ chụp nhiều ca cắt lớp CT/MRI gửi về trung tâm hội chẩn Main Site.
   - Bổ sung tải truyền ảnh tức thời: **+50 Mbps** trên mỗi tuyến WAN Leased Line.
4. **Hệ thống giám sát Camera IP (VLAN 70):**
   - Các luồng streaming camera bảo vệ giám sát từ các site phụ về phòng an ninh trung tâm Main Site: 10 luồng × 2 Mbps = **20 Mbps**.

#### Kết quả tính toán Kịch bản 3:

| Thành phần mạng | Peak Baseline (+20% 5Y) | Thành phần bùng phát (Burst) | Tổng Peak Kịch bản Cao | Khả năng đáp ứng đề xuất |
| :--- | :---: | :---: | :---: | :---: |
| **WAN Leased Line Main-DBP** | 111,36 Mbps | PACS Burst (+50 Mbps) + Camera (+10 Mbps) | **171,36 Mbps** | Cần nâng cấp gói Leased Line 200 Mbps hoặc bật Dynamic QoS burst |
| **WAN Leased Line Main-BHTQ** | 111,36 Mbps | PACS Burst (+50 Mbps) + Camera (+10 Mbps) | **171,36 Mbps** | Cần nâng cấp gói Leased Line 200 Mbps hoặc bật Dynamic QoS burst |
| **Internet Download (Edge)** | 399,57 Mbps | Guest Wi-Fi Burst (+200 Mbps) | **599,57 Mbps** (~ 600 Mbps) | 2 đường FTTH 500 Mbps chạy Active/Active (Tổng 1 Gbps) đáp ứng hoàn hảo (60% tải) |
| **Internet Upload (Edge)** | 81,64 Mbps | Guest Wi-Fi Burst (+51,2 Mbps) | **132,84 Mbps** | Đáp ứng thoải mái |
| **Tổng Internet Edge 2 chiều** | 481,21 Mbps | Guest Wi-Fi (+251,2 Mbps) | **732,41 Mbps** | Hạ tầng 1 Gbps vẫn còn dư 27% headroom |

---

## 3. Bảng tổng hợp so sánh 3 Kịch bản tải

```
+---------------------------------------------------------------------------------------------------------+
|                                    BẢNG SO SÁNH 3 KỊCH BẢN TẢI                                         |
+--------------------+----------------------------+-----------------------------+-------------------------+
| Đại lượng mạng     | Kịch bản 1: Tải Thấp       | Kịch bản 2: Baseline Chuẩn  | Kịch bản 3: Tải Cực Hạn |
|                    | (Off-Peak 21h)             | (3h Cao Điểm Đề Bài)        | (Peak 5 Năm + Spike)    |
+--------------------+----------------------------+-----------------------------+-------------------------+
| Giờ áp dụng        | Ngoài giờ (16h-9h, 11h-15h)| 09h-11h và 15h-16h          | Đỉnh cao điểm 5 năm sau |
| Số Workstation     | 1.120 (chỉ ~10% active)    | 1.120 (100% active)         | 1.344 (120% active)     |
| Guest Wi-Fi        | Không đáng kể              | 500 MB/ngày tổng            | 200 user đồng thời      |
| Lưu lượng WAN DBP  | 3,32 Mbps                  | 92,80 Mbps                  | 171,36 Mbps             |
| Lưu lượng WAN BHTQ | 3,32 Mbps                  | 92,80 Mbps                  | 171,36 Mbps             |
| Internet Download  | 11,89 Mbps                 | 332,98 Mbps                 | 599,57 Mbps             |
| Internet Upload    | 2,43 Mbps                  | 68,03 Mbps                  | 132,84 Mbps             |
| Tổng Internet Edge | 14,32 Mbps                 | 401,01 Mbps                 | 732,41 Mbps             |
| Đề xuất kênh WAN   | Leased Line 100 Mbps       | Leased Line 100-150 Mbps    | Leased Line 200 Mbps    |
| Đề xuất Internet   | 2 x 300 Mbps FTTH          | 2 x 300-500 Mbps FTTH       | 2 x 500 Mbps FTTH       |
+--------------------+----------------------------+-----------------------------+-------------------------+
```

---

## 4. Giải pháp Kỹ thuật kiểm soát tải và đảm bảo chất lượng dịch vụ (QoS)

Để hệ thống mạng bệnh viện vận hành ổn định trong Kịch bản 2 và không bị sập kênh trong Kịch bản 3, các chính sách sau bắt buộc phải được áp dụng:

1. **Phân loại hàng đợi ưu tiên QoS (Quality of Service - CoS/DSCP):**
   - **Queue 1 - Strict Priority (EF - Expedited Forwarding / DSCP 46):** Các luồng dữ liệu y tế tức thời (tín hiệu cấp cứu, hội chẩn VoIP/Video mổ nội soi, dữ liệu sinh hiệu monitor ICU).
   - **Queue 2 - Assured Forwarding (AF31 / DSCP 26):** Dữ liệu HIS, LIS, Database bệnh án điện tử, hệ thống kê đơn thuốc. Cam kết băng thông tối thiểu 40%.
   - **Queue 3 - Assured Forwarding (AF21 / DSCP 18):** Dữ liệu ảnh RIS-PACS dung lượng lớn. Cấp băng thông linh hoạt, cho phép truyền burst khi kênh rỗi.
   - **Queue 4 - Best Effort (BE / DSCP 0):** Duyệt web thông thường của nhân viên, tải file tài liệu hành chính.
   - **Queue 5 - Scavenger / Rate-Limited (CS1 / DSCP 8):** Mạng Guest Wi-Fi (VLAN 60). Bị bóp băng thông tối đa không vượt quá 20% tổng đường truyền Internet và bị hạ mức ưu tiên thấp nhất.
2. **Cơ chế Cân bằng tải Internet (Load Balancing Mechanism):**
   - Triển khai **Per-Flow Equal-Cost Multi-Path (ECMP)** hoặc Policy-Based Routing (PBR) trên Edge Firewall/Router:
     - Kênh DSL 1: Chuyên dụng định tuyến lưu lượng nghiệp vụ Bệnh viện (HIS, Server DMZ, PACS, máy trạm các phòng ban).
     - Kênh DSL 2: Chuyên dụng định tuyến lưu lượng Web chung của các máy trạm và toàn bộ mạng Guest Wi-Fi.
   - Khi 1 kênh DSL gặp sự cố, hệ thống chuyển mạch **Failover tự động trong thời gian < 3 giây**, đồng thời chính sách QoS tự động ngắt hoặc bóp băng thông Guest Wi-Fi xuống 512 kbps để bảo toàn 100% băng thông cho ứng dụng khám chữa bệnh cứu người.
