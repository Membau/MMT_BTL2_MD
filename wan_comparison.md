# BÁO CÁO SO SÁNH CÁC CÔNG NGHỆ WAN VÀ PHƯƠNG ÁN ĐỀ XUẤT
**Dự án:** Thiết kế hệ thống mạng Bệnh viện Đa khoa Chuyên sâu  
**Tác giả:** Bảo (Băng thông, Thiết bị & Công nghệ WAN)  
**Phối hợp:** Phước (Kiến trúc & IP), Toàn (Bảo mật), Trung (Mô phỏng & Cấu hình)  
**Tài liệu chuẩn:** Network Contract v1.1  

---

## 1. Bối cảnh và Yêu cầu kết nối WAN của Bệnh viện

Bệnh viện có 1 cơ sở chính (**Main Site**) và 2 cơ sở phụ (**Site DBP** và **Site BHTQ**). Đặc điểm luồng dữ liệu WAN bao gồm:
1. **Dữ liệu Y tế nhạy cảm và thời gian thực:**
   - Hệ thống thông tin bệnh viện (HIS), Xét nghiệm (LIS), Quản lý quan hệ bệnh nhân (CRM) và Cơ sở dữ liệu đặt tập trung tại Data Center Main Site.
   - Các máy trạm tại phòng ban của DBP và BHTQ phải truy vấn và cập nhật hệ thống thông tin bệnh viện (HIS, EMR) liên tục. Độ trễ yêu cầu <= 20 ms, tỷ lệ mất gói <= 0,1%.
2. **Hình ảnh Y khoa độ phân giải cao (RIS-PACS):**
   - Các file ảnh chụp cắt lớp vi tính (CT Scanner 128 lát, MRI, DSA tim mạch) có dung lượng từ 50 MB - 500 MB truyền về hội chẩn. Kênh WAN cần thông lượng lớn và khả năng burst tốt.
3. **Mọi lưu lượng Internet đi qua Main Site:**
   - Theo Network Contract v1.1, toàn bộ PC tại DBP và BHTQ không ra Internet trực tiếp mà đi xuyên kênh WAN về Main Site để đi qua Edge Firewall kiểm soát an ninh tập trung.
4. **Yêu cầu sẵn sàng cao (High Availability):**
   - Sự cố đứt cáp WAN không được phép làm tê liệt hoạt động cấp cứu và khám bệnh tại các chi nhánh.

---

## 2. So sánh toàn diện các Công nghệ WAN

Bảng so sánh 4 giải pháp WAN phổ biến hiện nay: **Leased Line truyền thống**, **MPLS VPN**, **SD-WAN (Software-Defined WAN)**, và **IPsec VPN qua Internet băng rộng (Broadband FTTH)**:

| Tiêu chí so sánh | 1. Leased Line (Point-to-Point) | 2. MPLS VPN (L3VPN / L2VPN) | 3. SD-WAN (Mạng WAN định nghĩa bằng phần mềm) | 4. IPsec VPN over Internet (FTTH) |
| :--- | :--- | :--- | :--- | :--- |
| **Bản chất công nghệ** | Kênh thuê riêng vật lý/logic dành riêng (Layer 1/2) qua mạng SDH/OTN hoặc Metro Ethernet. | Mạng riêng ảo chuyển nhãn đa giao thức do nhà mạng (ISP/SP) quản lý và gắn nhãn nhãn QoS. | Lớp mạng phủ (Overlay) thông minh trên mọi nền tảng truyền tải (Underlay: Leased Line, FTTH, 4G/5G). | Đường hầm mã hóa bảo mật (GRE/IPsec) đi trên mạng Internet công cộng không cam kết. |
| **Chi phí đầu tư (CAPEX)** | Thấp - Trung bình (chỉ cần Router có cổng GE/Serial/SFP). | Trung bình (cần router doanh nghiệp hỗ trợ định tuyến ISP). | Trung bình - Cao (cần thiết bị SD-WAN Edge hoặc bản quyền license vManage/Cloud). | Rất thấp (dùng router sẵn có hỗ trợ IPsec VPN). |
| **Chi phí vận hành (OPEX)** | **Rất cao** (cước hàng tháng đắt nhất, tính theo từng Mbps cam kết). | **Cao** (thấp hơn Leased Line nhưng cao hơn Internet nhiều lần). | **Tối ưu nhất** (kết hợp được đường FTTH giá rẻ với 1 đường leased line nhỏ). | **Rất rẻ** (chỉ tốn chi phí thuê bao Internet FTTH vài trăm nghìn/tháng). |
| **Hiệu năng & Băng thông** | Thông lượng tối đa bằng gói thuê; độ trễ cực thấp (<= 3-5 ms); không jitter. | Rất tốt, có cam kết độ trễ theo từng CoS/QoS của ISP. | Cực cao nhờ tính năng gộp băng thông (Bandwidth Aggregation) từ nhiều đường. | Phụ thuộc chất lượng Internet; có thể bị biến động trễ (jitter) và nghẽn giờ cao điểm. |
| **Độ tin cậy & SLA** | **Tối cao (99,99%)**, có bồi thường cam kết thời gian phục hồi MTTR nghiêm ngặt. | **Rất cao (99,9%)**, có SLA cam kết từ nhà mạng. | **Tuyệt đối cao**, tự động chuyển mạch sang đường dự phòng trong micro-giây khi có suy hao gói. | **Thấp - Trung bình**, Internet công cộng không có SLA cam kết dịch vụ. |
| **Bảo mật** | Tách biệt vật lý hoàn toàn, an toàn tự nhiên nhưng không có mã hóa payload mặc định. | Tách biệt theo VRF/VPN ID của nhà mạng, tin cậy ở mức nhà mạng. | Mã hóa đầu cuối (End-to-End IPsec AES-256), Zero-Trust Network Access (ZTNA) tích hợp. | Mã hóa mạnh IPsec (AES-256/SHA-2), an toàn trên môi trường Internet công cộng. |
| **Khả năng quản trị** | Thủ công từng thiết bị qua CLI; không có giám sát chất lượng ứng dụng tự động. | Cần phối hợp với ISP mỗi khi đổi routing/QoS; quản trị phức tạp. | **Giao diện tập trung trực quan (Single Pane of Glass)**; Zero-Touch Provisioning (ZTP). | Cấu hình từng router cục bộ; khó quản lý khi mở rộng nhiều chi nhánh. |
| **Khả năng định tuyến theo ứng dụng** | Không có (chỉ định tuyến theo địa chỉ IP/Subnet). | Định tuyến dựa trên DSCP/CoS cơ bản do nhà mạng map nhãn. | **Application-Aware Routing**: tự động chọn đường tối ưu cho HIS, PACS, Zoom dựa trên RTT/Jitter. | Không có hoặc cần cấu hình PBR phức tạp thủ công. |
| **Mức độ mô phỏng trên Packet Tracer** | **Rất tốt**, hỗ trợ đầy đủ Serial/Ethernet Point-to-Point và OSPF. | Hạn chế (CPT không mô phỏng đầy đủ MPLS PE-P-CE nhãn LDP/BGP). | **Không hỗ trợ** (Cisco Packet Tracer không có phần mềm vEdge/vManage SD-WAN). | Hỗ trợ mô phỏng IPsec VPN Site-to-Site cơ bản qua Router. |

---

## 3. Phân tích chi tiết và Lựa chọn phương án

### 3.1 Phương án Đề xuất cho Thiết kế Thực tế (Production Architecture)

Trong môi trường thực tế của một bệnh viện đa cơ sở, giải pháp lý tưởng nhất là **Mô hình Hybrid WAN tích hợp SD-WAN**:
- **Kênh chính (Primary WAN):** Sử dụng kênh **Metro Ethernet Leased Line 100 Mbps - 150 Mbps** kết nối trực tiếp từ Main Site đến DBP và BHTQ.
  - *Lý do:* Đảm bảo độ trễ tối thiểu tuyệt đối (< 5 ms), không suy hao gói cho dữ liệu y tế cứu mạng người (hội chẩn mổ, dữ liệu tim mạch, PACS, database HIS).
- **Kênh dự phòng (Secondary / Backup WAN):** Sử dụng **Đường truyền Internet FTTH Doanh nghiệp 300 Mbps - 500 Mbps** chạy đường hầm bảo mật **IPsec VPN** kết nối về Main Site.
  - *Lý do:* Chi phí cực kỳ rẻ (chỉ khoảng 1–2 triệu VNĐ/tháng), nhưng cung cấp băng thông dồi dào.
- **Lớp điều khiển thông minh SD-WAN Edge:**
  - Kích hoạt tính năng **Application-Aware Routing**:
    - Khi cả 2 đường hoạt động bình thường: Lưu lượng HIS, Database, PACS chạy trên Leased Line; lưu lượng Internet, Guest Wi-Fi và cập nhật phần mềm chạy qua FTTH VPN.
    - Khi đường Leased Line bị đứt: Toàn bộ lưu lượng y tế tự động bẻ ghi sang đường hầm IPsec VPN trong thời gian **< 1 giây**, người dùng không hề nhận ra sự gián đoạn.

### 3.2 Phương án Lựa chọn trong Đồ án Môn học và Mô phỏng Packet Tracer

Để đảm bảo tính khả thi cao nhất trên công cụ **Cisco Packet Tracer** mà nhóm đã thống nhất (theo Network Contract v1.1):
- **Công nghệ triển khai mô phỏng:** **Point-to-Point Leased Line (sử dụng Serial hoặc Gigabit Ethernet Routed Interfaces) kết hợp Giao thức định tuyến động OSPFv2 (Open Shortest Path First)**.
- **Phân bổ địa chỉ Subnet WAN theo Contract và Topology (khớp WAN1, WAN2 của Phước):**
  - Tuyến `MAIN - DBP`: Subnet `10.255.0.0/30`
    - IP đầu Main (RTR-MAIN-01 / WAN1): `10.255.0.1/30`
    - IP đầu DBP (RTR-DBP-01 / BR1_ROUTER): `10.255.0.2/30`
  - Tuyến `MAIN - BHTQ`: Subnet `10.255.0.4/30`
    - IP đầu Main (RTR-MAIN-02 / WAN2): `10.255.0.5/30`
    - IP đầu BHTQ (RTR-BHTQ-01 / BR2_ROUTER): `10.255.0.6/30`
- **Giao thức OSPFv2 (Process ID 1, Backbone Area 0):**
  - Tự động quảng bá dải mạng các site: Main (`10.10.0.0/16`), DBP (`10.20.0.0/16`), BHTQ (`10.30.0.0/16`).
  - Hội tụ nhanh khi có sự cố, dễ kiểm tra bằng các lệnh `show ip ospf neighbor`, `show ip route`.
- **Cơ chế dự phòng mô phỏng:**
  - Tại Main Site, mô phỏng 2 đường kết nối ra Internet qua ISP (sử dụng 2 địa chỉ IP Public khác nhau) kết nối vào 2 Firewall HA (`FW-MAIN-01`, `FW-MAIN-02`) và 2 Router WAN (`RTR-MAIN-01`, `RTR-MAIN-02`), cấu hình Default Route và Floating Static Route / IP SLA tracking hoặc cân bằng tải.

---

## 4. Sơ đồ Kiến trúc WAN Logic và Luồng Dữ liệu (Khớp 100% Topology Phước)

Dưới đây là sơ đồ kiến trúc WAN kết nối giữa 3 site và Internet Edge, hoàn toàn đồng bộ với sơ đồ logic PlantUML của Phước:

```mermaid
flowchart TD
    subgraph INTERNET_CLOUD [" Đám Mây Internet / Công Cộng "]
        ISP1["Nhà mạng ISP 1 (VNPT)"]
        ISP2["Nhà mạng ISP 2 (Viettel)"]
        PUB_WEB["Web Server Ngoài Internet"]
    end

    subgraph MAIN_SITE [" MAIN SITE (Cơ Sở Chính - TP.HCM) "]
        subgraph EDGE_LAYER ["Lớp Bảo Mật & WAN Biên (HA Redundancy)"]
            FW1["Firewall 1 Active (FW-MAIN-01)"]
            FW2["Firewall 2 Standby (FW-MAIN-02)"]
            WAN1["WAN Edge Router 1 (RTR-MAIN-01)"]
            WAN2["WAN Edge Router 2 (RTR-MAIN-02)"]
            FW1 <-->|HA State Sync| FW2
            WAN1 <-->|WAN Redundancy| WAN2
        end

        subgraph DMZ_LAYER ["Vùng DMZ (VLAN 50)"]
            DMZ_SW["DMZ Switch (SW-MAIN-DMZ-01)"]
            DMZ_WEB["Reverse Proxy / Web Server"]
            DMZ_PUB["Public DNS / Mail Gateway"]
            DMZ_SW --- DMZ_WEB
            DMZ_SW --- DMZ_PUB
        end

        subgraph CORE_LAYER ["Lớp Core Trung Tâm (Stack / MLAG)"]
            CORE1["Core L3 Switch 1 (CSW-MAIN-01)"]
            CORE2["Core L3 Switch 2 (CSW-MAIN-02)"]
            CORE1 <-->|StackWise-Virtual| CORE2
        end

        subgraph DIST_LAYER ["Lớp Phân Phối (Distribution)"]
            DIST_A["Distribution Tòa A (DSW-MAIN-A-01)"]
            DIST_B["Distribution Tòa B (DSW-MAIN-B-01)"]
        end

        subgraph DC_SERVICES ["Hệ Thống Máy Chủ & Giám Sát Data Center"]
            SRV_FARM["Server Farm (VLAN 40)<br>HIS, PACS, LIS, CRM, DB"]
            CAM_SW["Camera Switch (SW-MAIN-CAM-01)"]
            NVR_SYS["Hệ thống NVR & Quản lý Camera"]
            CAM_SW --- NVR_SYS
        end

        subgraph ACCESS_USERS ["Máy Trạm Phòng Ban & Mạng Không Dây"]
            STAFF_LAN["Máy trạm phòng ban Tòa A & B (VLAN 10/20)"]
            WIFI_SYS["Hệ thống APs (Staff Wi-Fi & Guest Wi-Fi VLAN 60)"]
        end
    end

    subgraph WAN_LINKS [" KÊNH TRUYỀN DẪN WAN CHUYÊN DỤNG "]
        LL_DBP["Leased Line 1: 10.255.0.0/30<br>Cam kết 100-150 Mbps, Độ trễ < 5ms"]
        LL_BHTQ["Leased Line 2: 10.255.0.4/30<br>Cam kết 100-150 Mbps, Độ trễ < 5ms"]
    end

    subgraph SITE_DBP [" CƠ SỞ PHỤ DBP STREET "]
        BR1_ROUTER["Branch Router 1 (RTR-DBP-01)"]
        BR1_CSW["Branch Core L3 Switch (CSW-DBP-01)"]
        BR1_SW["Branch Access Switches (7 Switch 48P)"]
        BR1_USERS["260 Máy trạm + APs + Camera"]
        BR1_SRV["2 Servers Cục bộ (RODC/Cache)"]
        BR1_ROUTER --- BR1_CSW --- BR1_SW --- BR1_USERS
        BR1_CSW --- BR1_SRV
    end

    subgraph SITE_BHTQ [" CƠ SỞ PHỤ BHTQ STREET "]
        BR2_ROUTER["Branch Router 2 (RTR-BHTQ-01)"]
        BR2_CSW["Branch Core L3 Switch (CSW-BHTQ-01)"]
        BR2_SW["Branch Access Switches (7 Switch 48P)"]
        BR2_USERS["260 Máy trạm + APs + Camera"]
        BR2_SRV["2 Servers Cục bộ (RODC/Cache)"]
        BR2_ROUTER --- BR2_CSW --- BR2_SW --- BR2_USERS
        BR2_CSW --- BR2_SRV
    end

    %% Kết nối ISP ra Firewall
    ISP1 --- FW1
    ISP2 --- FW2
    ISP1 --- PUB_WEB
    ISP2 --- PUB_WEB

    %% Kết nối Firewall đến DMZ và Core
    FW1 --- DMZ_SW
    FW2 --- DMZ_SW
    FW1 --- CORE1
    FW2 --- CORE2

    %% Kết nối Core đến WAN Routers
    WAN1 --- CORE1
    WAN2 --- CORE2

    %% Kết nối Core đến Distribution và Server/CCTV
    CORE1 --- DIST_A
    CORE2 --- DIST_A
    CORE1 --- DIST_B
    CORE2 --- DIST_B
    CORE1 --- SRV_FARM
    CORE2 --- SRV_FARM
    CORE1 --- CAM_SW
    CORE2 --- CAM_SW

    %% Kết nối Distribution đến Access
    DIST_A --- STAFF_LAN
    DIST_B --- STAFF_LAN
    DIST_A --- WIFI_SYS
    DIST_B --- WIFI_SYS

    %% Kết nối WAN Leased Lines
    WAN1 === LL_DBP === BR1_ROUTER
    WAN2 === LL_BHTQ === BR2_ROUTER

    %% Định dạng style
    classDef main fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef branch fill:#e8f5e9,stroke:#388e3c,stroke-width:2px;
    classDef wan fill:#fff3e0,stroke:#f57c00,stroke-width:2px;
    classDef net fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;

    class FW1,FW2,WAN1,WAN2,CORE1,CORE2,DIST_A,DIST_B,SRV_FARM,DMZ_SW,CAM_SW,STAFF_LAN,WIFI_SYS main;
    class BR1_ROUTER,BR1_CSW,BR1_SW,BR1_USERS,BR1_SRV,BR2_ROUTER,BR2_CSW,BR2_SW,BR2_USERS,BR2_SRV branch;
    class LL_DBP,LL_BHTQ wan;
    class ISP1,ISP2,PUB_WEB net;
```

---

## 5. Kết luận và Khuyến nghị Kỹ thuật cho Nhóm

1. **Về phía Phước (Kiến trúc & IP):**
   - Đảm bảo các cổng WAN trên 2 router `RTR-MAIN-01` (WAN1) và `RTR-MAIN-02` (WAN2) được gán chính xác IP `/30` theo bảng phân bổ Contract v1.1.
   - Thể hiện rõ trên sơ đồ các cổng kết nối Uplink từ Core Switch Main sang Router biên WAN và sang Firewall.
   - Bổ sung khối Core Switch L3 (`CSW-DBP-01`, `CSW-BHTQ-01`) và 2 server chi nhánh tại DBP và BHTQ trên sơ đồ kiến trúc để đồng bộ hoàn toàn với Contract v1.1.
2. **Về phía Toàn (Bảo mật):**
   - Xây dựng chính sách Firewall tại Main Site cho phép luồng định tuyến OSPF (IP Protocol 89) qua 2 đường Leased Line giữa Main và DBP/BHTQ.
   - Áp dụng cấu hình NAT Overload (PAT) cho cả 3 dải IP `10.10.0.0/16`, `10.20.0.0/16`, `10.30.0.0/16` khi truy cập Internet thông qua 2 cổng ra của Edge Firewall.
3. **Về phía Trung (Mô phỏng Cisco Packet Tracer):**
   - Cấu hình OSPF đơn vùng (Single-Area OSPF Area 0) trên RTR-MAIN-01, RTR-MAIN-02, RTR-DBP-01 và RTR-BHTQ-01 để đảm bảo các PC tại DBP và BHTQ ping thông suốt đến Server Farm tại Main Site và ngược lại.
   - Kiểm tra định tuyến default route `0.0.0.0 0.0.0.0` được router biên Main quảng bá qua OSPF (`default-information originate`) để toàn bộ các site phụ tự động nhận diện lối thoát ra Internet.
