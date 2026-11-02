# BÁO CÁO PHÂN TÍCH LƯU LƯỢNG VÀ TÍNH TOÁN BĂNG THÔNG MẠNG BỆNH VIỆN
**Dự án:** Thiết kế hệ thống mạng Bệnh viện Đa khoa Chuyên sâu (Hospital Network)  
**Tác giả phân tích:** Bảo (Vai trò: Băng thông, Thiết bị & Công nghệ WAN)  
**Phối hợp:** Phước (Kiến trúc & IP), Toàn (Bảo mật), Trung (Mô phỏng & Cấu hình)  
**Phiên bản Network Contract áp dụng:** v1.1  

---

## 1. Dữ liệu đầu vào trích xuất từ đề bài

Hệ thống mạng được triển khai trên 3 cơ sở (sites) gồm **Main Site** (TP. Hồ Chí Minh) và 2 cơ sở phụ (**DBP Street** và **BHTQ Street**).

### 1.1 Thống kê quy mô thiết bị đầu cuối và máy chủ

| Địa điểm (Site) | Quy mô kiến trúc | Workstation (PC) | Servers | Thiết bị mạng tối thiểu |
| :--- | :--- | :---: | :---: | :---: |
| **Main Site** | 2 tòa nhà A & B (5 tầng, 10 phòng/tầng), Data Center cách 50m | 600 | 10 | >= 12 thiết bị |
| **DBP Street** | Tòa nhà 2 tầng (1 IT room, 1 Cabling Central) | 260 | 2 | >= 5 thiết bị |
| **BHTQ Street** | Tòa nhà 2 tầng (1 IT room, 1 Cabling Central) | 260 | 2 | >= 5 thiết bị |
| **TOÀN VIỆN** | **3 Cơ sở** | **1.120** | **14** | **>= 22 thiết bị** |

### 1.2 Thông số lưu lượng và hành vi người dùng theo ngày

1. **Workstations (Máy trạm các phòng ban - Staff theo Contract v1.1):**
   - Số lượng: 1.120 máy tính toàn hệ thống (Main: 600, DBP: 260, BHTQ: 260).
   - Tác vụ theo đề bài: Duyệt web (web browsing), tải tài liệu (document downloads), thực hiện giao dịch khách hàng (customer transactions) và vận hành các phần mềm bệnh viện (HIS, RIS-PACS, LIS, CRM, client-server applications, multimedia, databases).
   - Lưu lượng định mức:
     - **Download (DL):** 500 MB/ngày/workstation.
     - **Upload (UL):** 100 MB/ngày/workstation.
2. **Server Farm (Hệ thống máy chủ ứng dụng y tế và hạ tầng):**
   - Dịch vụ: HIS, RIS-PACS, LIS, CRM, Web Portal, Cơ sở dữ liệu Oracle/SQL, Cập nhật phần mềm hệ thống (OS/Patch Update).
   - Lưu lượng tổng định mức theo đề bài:
     - **Download (DL):** 1.000 MB/ngày (tải các bản vá, cập nhật phần mềm, dữ liệu từ ngoài).
     - **Upload (UL):** 2.000 MB/ngày (phục vụ web truy cập, portal, báo cáo cho đối tác/bảo hiểm).
3. **Thiết bị Khách hàng / Bệnh nhân (Guest Wi-Fi):**
   - Đề bài ghi nhận: Thiết bị Wi-Fi của khách tải xuống khoảng 500 MB/ngày.
   - *Ghi chú giả định:* Được phân tích ở 2 mức độ (Mức cơ sở theo số liệu tối thiểu đề bài 500 MB/ngày và Mức mở rộng khi có hàng trăm bệnh nhân/thân nhân truy cập đồng thời).
4. **Phân bổ thời gian và hệ số tập trung tải cao điểm:**
   - Hoạt động khám chữa bệnh diễn ra mạnh nhất trong giờ hành chính.
   - Tỷ lệ lưu lượng trong giờ cao điểm: **80% tổng lưu lượng cả ngày**.
   - Khung giờ cao điểm:
     - Buổi sáng: **09:00 – 11:00** (2 giờ).
     - Buổi chiều: **15:00 – 16:00** (1 giờ).
     - **Tổng thời gian cao điểm (T_peak):** 2 + 1 = 3 giờ (tương đương 10.800 giây).
   - Khung giờ thấp điểm: 24 - 3 = 21 giờ (75.600 giây), chiếm **20% tổng lưu lượng**.
5. **Dự báo tăng trưởng quy mô (Growth Rate):**
   - Tỷ lệ tăng trưởng: **20% trong 5 năm** (hệ số tăng trưởng G = 1,20) tính trên số lượng user, tải mạng và quy mô mở rộng.

---

## 2. Quy chuẩn đơn vị đo và nguyên tắc chuyển đổi

Để đảm bảo tính toán kỹ thuật chính xác, tuyệt đối không nhầm lẫn giữa **dung lượng dữ liệu lưu trữ** và **tốc độ truyền dẫn trên kênh truyền**:

| Đại lượng | Đơn vị đo | Ý nghĩa vật lý | Quy đổi sang bit |
| :--- | :--- | :--- | :--- |
| **Dung lượng (Data Volume)** | Megabyte (MB), Gigabyte (GB) | Tổng khối lượng dữ liệu tích lũy được truyền nhận trong 1 ngày (24h). | 1 Byte = 8 bits<br>1 MB = 8 × 10^6 bits (chuẩn SI) |
| **Thông lượng / Tốc độ (Throughput / Bandwidth)** | Megabits per second (Mbps) | Tốc độ truyền dữ liệu tức thời qua cổng mạng hoặc đường truyền. | 1 Mbps = 10^6 bits/giây = 1.000.000 bps |

### Công thức quy đổi toán học chuẩn:

1. **Chuyển đổi MB sang Megabit (Mb):**
```
Dung lượng (Mb) = Dung lượng (MB) × 8
```

2. **Thông lượng trung bình 24 giờ (R_avg):**
```
                Tổng dung lượng trong ngày (MB) × 8
R_avg = ---------------------------------------------------  (Mbps)
                         86.400 giây
```

3. **Thông lượng trong khung giờ cao điểm (R_peak):**
```
                80% × Tổng dung lượng trong ngày (MB) × 8
R_peak = -------------------------------------------------------  (Mbps)
                           10.800 giây
```

4. **Thông lượng sau 5 năm tăng trưởng 20% (R_growth):**
```
R_growth = R_peak × (1 + 0,20) = R_peak × 1,20  (Mbps)
```

---

## 3. Tính toán chi tiết lưu lượng và thông lượng

### 3.1 Tổng lưu lượng dữ liệu mỗi ngày (Daily Volume)

#### A. Main Site (600 Workstations + 10 Servers + Guest Wi-Fi):
- DL_WS-Main = 600 × 500 MB = 300.000 MB/ngày ≈ 292,97 GB/ngày.
- UL_WS-Main = 600 × 100 MB = 60.000 MB/ngày ≈ 58,59 GB/ngày.
- DL_Server = 1.000 MB/ngày ≈ 0,98 GB/ngày.
- UL_Server = 2.000 MB/ngày ≈ 1,95 GB/ngày.
- DL_Guest = 500 MB/ngày ≈ 0,49 GB/ngày.
- **Tổng Main Site:**
  - DL_Main = 300.000 + 1.000 + 500 = **301.500 MB/ngày** (294,43 GB/ngày).
  - UL_Main = 60.000 + 2.000 = **62.000 MB/ngày** (60,55 GB/ngày).
  - Tổng cộng 2 chiều Main Site = 363.500 MB/ngày (354,98 GB/ngày).

#### B. DBP Street (260 Workstations + 2 Servers):
- DL_WS-DBP = 260 × 500 MB = 130.000 MB/ngày ≈ 126,95 GB/ngày.
- UL_WS-DBP = 260 × 100 MB = 26.000 MB/ngày ≈ 25,39 GB/ngày.
- Giả định phần Server nội bộ xử lý tác vụ đồng bộ nhẹ: 200 MB DL, 400 MB UL.
- **Tổng DBP Site:**
  - DL_DBP = 130.000 + 200 = **130.200 MB/ngày** (127,15 GB/ngày).
  - UL_DBP = 26.000 + 400 = **26.400 MB/ngày** (25,78 GB/ngày).
  - Tổng cộng 2 chiều DBP = 156.600 MB/ngày (152,93 GB/ngày).

#### C. BHTQ Street (260 Workstations + 2 Servers):
- Quy mô đối xứng với DBP:
  - DL_BHTQ = **130.200 MB/ngày** (127,15 GB/ngày).
  - UL_BHTQ = **26.400 MB/ngày** (25,78 GB/ngày).
  - Tổng cộng 2 chiều BHTQ = 156.600 MB/ngày (152,93 GB/ngày).

#### D. Toàn Bệnh Viện (Aggregate Hospital Network):
- **Tổng Download (DL):**
  - Total DL = 301.500 + 130.200 + 130.200 = **561.900 MB/ngày** (548,73 GB/ngày)
- **Tổng Upload (UL):**
  - Total UL = 62.000 + 26.400 + 26.400 = **114.800 MB/ngày** (112,11 GB/ngày)
- **Tổng lưu lượng toàn viện cả 2 chiều:**
  - Total Traffic = 561.900 + 114.800 = **676.700 MB/ngày** (660,84 GB/ngày)

---

### 3.2 Thông lượng trung bình 24 giờ (R_avg)

Áp dụng công thức: `R_avg = (Lưu lượng MB × 8) / 86.400 giây`

| Khu vực | Download (Mbps) | Upload (Mbps) | Tổng 2 chiều (Mbps) |
| :--- | :---: | :---: | :---: |
| **Main Site** | 27,92 Mbps | 5,74 Mbps | 33,66 Mbps |
| **DBP Street** | 12,06 Mbps | 2,44 Mbps | 14,50 Mbps |
| **BHTQ Street** | 12,06 Mbps | 2,44 Mbps | 14,50 Mbps |
| **Toàn Viện (Aggregate)** | **52,03 Mbps** | **10,63 Mbps** | **62,66 Mbps** |

---

### 3.3 Thông lượng trong khung giờ cao điểm (R_peak)

Áp dụng công thức: `R_peak = (0,80 × Lưu lượng MB × 8) / 10.800 giây`

| Khu vực | Peak Download (Mbps) | Peak Upload (Mbps) | Peak Tổng (Mbps) |
| :--- | :---: | :---: | :---: |
| **Main Site** | 178,67 Mbps | 36,74 Mbps | 215,41 Mbps |
| **DBP Street** | 77,16 Mbps | 15,64 Mbps | 92,80 Mbps |
| **BHTQ Street** | 77,16 Mbps | 15,64 Mbps | 92,80 Mbps |
| **Toàn Viện (Aggregate)** | **332,98 Mbps** | **68,03 Mbps** | **401,01 Mbps** |

> **Nhận xét quan trọng:**
> - Trong 3 giờ cao điểm (09:00–11:00 và 15:00–16:00), thông lượng tăng vọt gấp **6,4 lần** so với mức trung bình cả ngày:  
>   `(80% / 3h) / (100% / 24h) = (0,8 / 3) / (1 / 24) = 6,4 lần`.
> - Điều này chứng minh rằng nếu thiết kế băng thông theo mức trung bình 24h (~63 Mbps) thì hệ thống mạng bệnh viện sẽ bị nghẽn nghiêm trọng (congested) và tê liệt vào giờ làm việc cao điểm. Băng thông cung cấp bắt buộc phải đáp ứng tối thiểu mức Peak.

---

### 3.4 Nhu cầu sau 5 năm tăng trưởng 20% (R_growth)

Theo đề bài, hệ thống tăng trưởng 20% về quy mô người dùng và tải mạng:

| Khu vực | 5-Year Peak Download | 5-Year Peak Upload | 5-Year Peak Tổng |
| :--- | :---: | :---: | :---: |
| **Main Site** | 214,40 Mbps | 44,09 Mbps | 258,49 Mbps |
| **DBP Street** | 92,59 Mbps | 18,77 Mbps | 111,36 Mbps |
| **BHTQ Street** | 92,59 Mbps | 18,77 Mbps | 111,36 Mbps |
| **Toàn Viện (Aggregate)** | **399,57 Mbps** | **81,64 Mbps** | **481,21 Mbps** |

---

## 4. Đề xuất băng thông Internet và WAN kết nối liên Site

### 4.1 Băng thông Leased Line kết nối Main – DBP và Main – BHTQ
- **Bản chất luồng truyền:** 
  - Toàn bộ cơ sở dữ liệu y tế (HIS, PACS, LIS, CRM) đặt tập trung tại Data Center của Main Site.
  - Toàn bộ lưu lượng Internet của DBP và BHTQ cũng phải đi xuyên qua Leased Line về Main Site theo yêu cầu Network Contract v1.1.
  - Do đó, đường WAN nối Main – DBP và Main – BHTQ phải gánh trọn vẹn lưu lượng Peak của mỗi site phụ.
- **Nhu cầu tính toán:**
  - Hiện tại: Peak tổng là 92,80 Mbps (Peak DL: 77,16 Mbps, Peak UL: 15,64 Mbps).
  - Sau 5 năm: Peak tổng là 111,36 Mbps (Peak DL: 92,59 Mbps, Peak UL: 18,77 Mbps).
- **Khuyến nghị đề xuất:**
  - **Băng thông cam kết cho mỗi kênh Leased Line:** **100 Mbps - 150 Mbps đối xứng (Full-duplex)**.
  - Tỷ lệ cam kết băng thông (CIR) = 100%, độ khả dụng (SLA) >= 99,9%, độ trễ <= 5 ms.
  - Kèm tính năng Burst linh hoạt lên đến 200 Mbps khi có ca mổ khẩn cấp cần truyền file ảnh DICOM/CT Scan dung lượng lớn.

### 4.2 Băng thông 2 đường Internet DSL tại Main Site
- **Bản chất luồng truyền:**
  - Main Site đóng vai trò Gateway Internet duy nhất cho cả 3 cơ sở (1.120 workstation + Server Web + Guest Wi-Fi).
  - Tải Peak Internet tổng hiện tại: ~ 333 Mbps DL và ~ 68 Mbps UL.
  - Tải Peak Internet sau 5 năm: ~ 400 Mbps DL và ~ 82 Mbps UL.
- **Yêu cầu dự phòng và cân bằng tải:**
  - Hệ thống sử dụng 2 đường Internet doanh nghiệp (FTTH/DSL chuyên dụng) chạy cơ chế **Active/Active Load Balancing**.
  - **Yêu cầu Failover:** Nếu 1 đường cáp bị đứt hoặc sự cố nhà mạng, 1 đường còn lại phải gánh được tối thiểu 80-100% tải thiết yếu của toàn viện.
- **Khuyến nghị đề xuất:**
  - Sử dụng **2 đường truyền cáp quang doanh nghiệp từ 2 ISP độc lập** (ví dụ: ISP 1: VNPT, ISP 2: Viettel để chống single point of failure về hạ tầng vật lý ngầm).
  - **Băng thông đề xuất mỗi đường:** **300 Mbps - 500 Mbps**.
  - **Tổng băng thông kết hợp:** **600 Mbps - 1.000 Mbps (1 Gbps)**.
  - Mức băng thông này đảm bảo tải đỉnh 400 Mbps hoạt động ổn định ở mức 40–50% công suất danh định của kênh (headroom 50% chống jitter/burst).

---

## 5. Dữ liệu chưa xác định và khuyến nghị khảo sát thực tế

1. **Lưu lượng Guest Wi-Fi:** Đề bài chỉ ghi 500 MB/ngày. Trong thực tế bệnh viện quy mô 1.000 giường, mỗi ngày có thể có 500 – 1.000 lượt bệnh nhân và thân nhân kết nối Wi-Fi. Cần khảo sát mật độ truy cập để áp dụng giới hạn băng thông per-user (bandwidth shaping: ví dụ 2 Mbps/user) trên Controller.
2. **Tỷ lệ lưu lượng Nội bộ (Intranet) vs. Internet:** Đề bài gộp chung các hoạt động web, tài liệu và giao dịch khách hàng. Cần khảo sát lưu lượng ảnh y tế PACS (ảnh X-quang, MRI dung lượng 50–500MB/file) truyền giữa các máy trạm phòng ban để cấu hình QoS hàng đợi ưu tiên (Priority Queue).
3. **Lưu lượng Camera giám sát (VLAN 70) và Thiết bị Y tế (VLAN 30):** Đề bài chưa cho lưu lượng camera IP (H.264/H.265) và máy monitor y tế. Giả định camera ghi hình về NVR cục bộ tại từng site, chỉ truyền liên site khi có trích xuất an ninh.
