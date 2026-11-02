# DANH SÁCH GIẢ ĐỊNH, DỮ LIỆU CẦN XÁC MINH VÀ GIỚI HẠN MÔ PHỎNG
**Dự án:** Thiết kế hệ thống mạng Bệnh viện Đa khoa Chuyên sâu  
**Phần việc:** Phân tích băng thông, thiết bị và công nghệ WAN (Thành viên: Bảo)  
**Phối hợp:** Phước (Kiến trúc), Toàn (Bảo mật), Trung (Mô phỏng)  
**Tài liệu chuẩn:** Network Contract v1.1  

---

## 1. Danh sách các Giả định Kỹ thuật (Key Assumptions)

| STT | Tham số / Yếu tố | Giả định áp dụng | Căn cứ và Giải trình |
| :---: | :--- | :--- | :--- |
| **1** | **Quy chuẩn đơn vị** | 1 Byte = 8 bits; 1 MB = 8 Mbits; 1 Mbps = 1.000.000 bps. | Tuân thủ tiêu chuẩn viễn thông và mạng quốc tế (SI standard), tránh tuyệt đối việc nhầm lẫn giữa dung lượng tĩnh (MB/ngày) và tốc độ dòng (Mbps). |
| **2** | **Phân bổ tải cao điểm** | 80% lưu lượng cả ngày diễn ra trong 3 giờ cao điểm (09:00–11:00 và 15:00–16:00, tương đương 10.800 giây). | Đúng nguyên văn đề bài. Tải cao điểm tập trung gấp 6,4 lần tải trung bình 24h. 20% lưu lượng còn lại phân bổ trong 21 giờ thấp điểm (75.600 giây). |
| **3** | **Vị trí Server & Luồng WAN** | Toàn bộ 10 máy chủ nghiệp vụ cốt lõi (HIS, PACS, LIS, CRM, DB) đặt tập trung tại Data Center của Main Site. | Phù hợp với thực tế bệnh viện đa cơ sở (quản lý tập trung CSDL) và yêu cầu đề bài. Do đó, lưu lượng nghiệp vụ của 260 máy tính tại DBP và 260 máy tính tại BHTQ bắt buộc phải chạy qua kênh Leased Line WAN về Main Site. |
| **4** | **Lưu lượng Internet tập trung** | Toàn bộ lưu lượng ra Internet của cả 3 cơ sở đều phải đi qua Edge Firewall tại Main Site. | Yêu cầu bắt buộc từ Network Contract v1.1 (Mục 1) và đề bài. Do đó, kênh WAN của các site phụ phải gánh cả lưu lượng Internet của site đó. |
| **5** | **Khách hàng Wi-Fi (Guest)** | Baseline: 500 MB/ngày tải xuống. Kịch bản Stress: 200–500 khách đồng thời với giới hạn rate-limit 1 Mbps DL / 256 kbps UL. | Đề bài nêu 500 MB/ngày. Để dự phòng rủi ro quá tải, nhóm đưa thêm kịch bản mở rộng khi bệnh viện đông khách vào sáng sớm. |
| **6** | **Tăng trưởng 5 năm** | Mức tăng trưởng tích lũy là **20% sau 5 năm** (hệ số tăng trưởng G = 1,20) trên mọi mặt: số máy trạm, số người dùng, lưu lượng ứng dụng. | Đúng yêu cầu đề bài: "growth rate of 20% in 5 years". Đảm bảo headroom cho hạ tầng mạng không bị lỗi thời sau 5 năm. |
| **7** | **Mật độ thiết bị và cổng Switch** | Tỷ lệ sử dụng cổng thực tế khuyến nghị <= 75%, để dành tối thiểu 25% cổng trống (headroom) cho mở rộng và dự phòng cổng lỗi. | Với mỗi tầng 60 PC + thiết bị phụ trợ (~72-75 kết nối), sử dụng 2 switch 48-port (tổng 96 ports) là cấu hình chuẩn công nghiệp. |

---

## 2. Danh sách Dữ liệu Cần Xác minh Thực tế (Data Requiring Verification)

Các thông số dưới đây chưa được cung cấp cụ thể trong đề bài hoặc cần đo đạc thực địa trong giai đoạn khảo sát chi tiết (Site Survey):

1. **Khảo sát Lưu lượng và Số lượng Camera giám sát (VLAN 70):**
   - *Hiện trạng:* Đề bài yêu cầu đề xuất hệ thống camera giám sát nhưng chưa nêu số lượng camera và độ phân giải.
   - *Cần xác minh:* Bệnh viện dự kiến lắp bao nhiêu camera IP tại mỗi tầng (ví dụ: 4–6 camera/tầng)? Chuẩn nén là H.264 hay H.265? Đầu ghi hình NVR đặt tại từng cơ sở hay đặt tập trung tại Main Site?
2. **Khảo sát Danh mục và Giao thức Thiết bị Y tế (VLAN 30):**
   - *Hiện trạng:* Có các thiết bị y tế kết nối mạng tại 2 tòa nhà và các site phụ.
   - *Cần xác minh:* Danh mục cụ thể các máy y tế có cổng mạng (máy điện tim, monitor đầu giường, máy xét nghiệm sinh hóa, máy chụp X-quang di động)? Yêu cầu địa chỉ IP tĩnh hay DHCP reservation? Có yêu cầu giao thức HL7 / DICOM đặc thù không?
3. **Mặt bằng Kiến trúc và Chiều dài Kéo cáp (Cabling Distance):**
   - *Hiện trạng:* Biết tòa nhà 5 tầng (Main) và 2 tầng (phụ), phòng Data Center cách 50m.
   - *Cần xác minh:* Chiều dài từ tủ rack IDF các tầng đến các phòng xa nhất có vượt quá giới hạn 90 mét của cáp đồng Cat6 không? Nếu vượt quá, cần đặt thêm tủ mạng trung gian hoặc dùng bộ mở rộng cáp quang.
4. **Ngân sách Đầu tư (CAPEX) và Chi phí Vận hành Hàng tháng (OPEX):**
   - *Hiện trạng:* Đề bài yêu cầu đề xuất phương án WAN theo chi phí giải pháp.
   - *Cần xác minh:* Ngân sách dự kiến cho việc thuê kênh viễn thông hàng tháng (Leased Line thuần có chi phí khoảng 10–20 triệu VNĐ/tháng cho mỗi đường 100 Mbps; trong khi SD-WAN qua FTTH chỉ tốn 2–3 triệu VNĐ/tháng).

---

## 3. Giới hạn Mô phỏng (Simulation Limitations) và Giải pháp Thay thế

Khi hiện thực hóa đồ án trên môi trường mô phỏng (Cisco Packet Tracer / GNS3), có sự khác biệt rõ rệt giữa **Thiết kế Kỹ thuật Thực tế** và **Khả năng của Phần mềm Mô phỏng**:

| Công nghệ trong Thiết kế | Giới hạn trên Cisco Packet Tracer (CPT) | Phương án Mô phỏng Khả thi (Alternative Evidence) |
| :--- | :--- | :--- |
| **Quy mô 1.120 Máy tính (Workstations)** | CPT sẽ bị treo, giật lag nặng và crash file nếu cắm đủ 1.120 icon PC và hàng chục switch. | Dùng **PC đại diện (Representative PCs)** cho từng VLAN tại mỗi site (theo Network Contract v1.1). Đi kèm bảng tính dung lượng (Capacity Table) và báo cáo để chứng minh khả năng đáp ứng. |
| **Công nghệ SD-WAN thông minh** | CPT hoàn toàn **không hỗ trợ** các bộ điều khiển SD-WAN (như Cisco vManage, vSmart, vEdge) và tính năng Application-Aware Routing. | Mô phỏng lớp truyền dẫn WAN bằng **Point-to-Point Routed Links với giao thức OSPFv2**. Trong báo cáo thuyết minh chi tiết giải pháp SD-WAN Hybrid WAN thực tế. |
| **Công nghệ GPON (Mạng quang thụ động)** | CPT không hỗ trợ OLT/ONT quang thụ động chia chùm quang (Splitter 1:32/1:64) đúng chuẩn ITU-T G.984. | Mô phỏng đường trục quang (Optical Backbone) bằng các kết nối Gigabit Ethernet / Fiber SFP giữa Switch Core và Distribution/Access. |
| **Next-Generation Firewall (NGFW) & IDS/IPS** | CPT không có tính năng quét mã độc thực tế (AV), lọc Web theo danh mục động, hoặc cơ sở dữ liệu chữ ký Snort IDS/IPS cập nhật. | Sử dụng **Cisco ASA 5506-X** hoặc **Cisco Router tích hợp Extended Access Control Lists (ACL)** để mô phỏng chính sách lọc gói, phân vùng Security Zones và NAT/PAT. |
| **Cân bằng tải 2 đường Internet (Load Balancing)** | CPT hạn chế trong việc mô phỏng PBR/ECMP đa đường ra ngoài Internet qua các ISP ảo. | Cấu hình **Default Static Route kết hợp Floating Static Route (Administrative Distance)** hoặc cấu hình 2 default route song song để kiểm tra chuyển hướng lưu lượng. |
| **Wi-Fi Doanh nghiệp & Captive Portal** | CPT hỗ trợ WLC cơ bản (Cisco 3504 / 2504) và AP nhẹ (Lightweight AP), nhưng trang chào Captive Portal cho khách còn hạn chế. | Cấu hình WLC với 2 WLAN/SSID riêng biệt: SSID `Hospital_Staff` (WPA2-Enterprise gán VLAN 10/20) và SSID `Hospital_Guest` (gán VLAN 60 tách biệt, không cấp quyền vào mạng nội bộ). |

---

## 4. Bàn giao và Phối hợp Liên thành viên

1. **Gửi cho Phước (Kiến trúc & IP):**
   - Xác nhận số lượng switch, router, firewall đã khớp hoàn toàn với kiến trúc 3 tầng (Core - Distribution - Access) và bảng phân chia VLAN/IP của Contract v1.1.
   - Bố trí cổng Uplink 10GbE từ Distribution Switch về Core Switch để đảm bảo không bị thắt cổ chai nội bộ.
2. **Gửi cho Toàn (Bảo mật):**
   - Cung cấp thông số lưu lượng WAN và Internet Edge để Toàn chọn cấu hình phần cứng Firewall có thông lượng kiểm tra bảo mật (Threat Protection / IPS Throughput) đạt tối thiểu 1 Gbps.
   - Chuyển giao các giả định về dải mạng và cổng ứng dụng (HIS, PACS port 104/DICOM, Web 80/443, OSPF IP 89) để Toàn xây dựng Ma trận Access Control.
3. **Gửi cho Trung (Mô phỏng):**
   - Cung cấp danh sách thiết bị đại diện trong file Packet Tracer `HospitalNetwork_v1.pkt`.
   - Cung cấp danh mục các bài test kiểm tra throughput, độ trễ ping và hội tụ OSPF khi ngắt thử nghiệm 1 đường WAN để Trung ghi nhận kết quả và chụp màn hình bằng chứng.
