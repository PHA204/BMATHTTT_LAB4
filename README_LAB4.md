# LAB4 --- Khảo sát và đánh giá bề mặt mạng bằng Nmap

## 1. Thông tin sinh viên

-   **Họ và tên:** Phạm Hữu Ân
-   **MSSV:** 1150080084
-   **Mã lớp:** CNPM1

------------------------------------------------------------------------

## 2. Mục tiêu

-   Kiểm tra và sử dụng Nmap trong môi trường lab.
-   Thiết lập mạng Host-Only giữa Kali Linux và Metasploitable 2.
-   Thực hiện Host Discovery.
-   Khảo sát các cổng TCP bằng `-sT` và `-sS`.
-   Khảo sát UDP.
-   Xác định dịch vụ và phiên bản bằng `-sV`.
-   Thực hiện OS detection bằng `-O`.
-   Sử dụng NSE để thu thập thông tin SMB và kiểm tra dấu hiệu liên quan
    MS17-010.
-   Lưu kết quả quét thành các file evidence.
-   Thực hiện một thay đổi hardening và so sánh kết quả trước/sau.

------------------------------------------------------------------------

## 3. Môi trường thực hành

### 3.1. Máy quét

**Kali Linux**

-   VMware Workstation
-   Network: Host-Only
-   IPv4: `192.168.126.131/24`
-   Nmap: `7.99`

### 3.2. Máy mục tiêu

**Metasploitable 2**

-   VMware Workstation
-   Network: Host-Only
-   IPv4: `192.168.126.130/24`
-   Subnet: `192.168.126.0/24`
-   MAC: `00:0C:29:69:46:47`
-   Hệ điều hành/dịch vụ SMB được NSE nhận diện: Unix (Samba
    3.0.20-Debian)

### 3.3. Topology

``` text
                 Host-Only Network
                 192.168.126.0/24
                         |
             +-----------+-----------+
             |                       |
       Kali Linux               Metasploitable 2
   192.168.126.131            192.168.126.130
        Scanner                    Target
```

> Chỉ sử dụng các máy ảo thuộc môi trường lab và mạng Host-Only.

------------------------------------------------------------------------

## 4. Cấu hình và kiểm tra kết nối

### 4.1. Kiểm tra IP Kali

Lệnh:

``` bash
ip -br addr
```

Kết quả:

``` text
eth0    UP    192.168.126.131/24
```

### 4.2. Kiểm tra IP Metasploitable 2

Lệnh:

``` bash
ifconfig
```

Kết quả:

``` text
eth0    inet addr:192.168.126.130
Mask:255.255.255.0
```

### 4.3. Kiểm tra kết nối

Từ Kali:

``` bash
ping -c 4 192.168.126.130
```

Kết quả: kết nối Kali → Metasploitable 2 thành công.

------------------------------------------------------------------------

## 5. Host Discovery

Lệnh thực hiện:

``` bash
nmap -sn 192.168.126.0/24
```

Mục đích: phát hiện các host đang hoạt động trong mạng lab trước khi
thực hiện các bước quét dịch vụ.

Host mục tiêu xác định được:

``` text
192.168.126.130
```

------------------------------------------------------------------------

## 6. TCP Scan

### 6.1. TCP Connect Scan

Lệnh:

``` bash
nmap -sT 192.168.126.130
```

Kết quả:

-   `23` cổng TCP ở trạng thái `open`.
-   `977` cổng TCP ở trạng thái `closed`.

Một số dịch vụ được Nmap nhận diện:

  Port       Service
  ---------- --------------
  21/tcp     ftp
  22/tcp     ssh
  23/tcp     telnet
  25/tcp     smtp
  53/tcp     domain
  80/tcp     http
  139/tcp    netbios-ssn
  445/tcp    microsoft-ds
  3306/tcp   mysql
  5432/tcp   postgresql
  5900/tcp   vnc
  8180/tcp   unknown/http

### 6.2. TCP SYN Scan

Lệnh:

``` bash
sudo nmap -sS 192.168.126.130
```

Kết quả:

-   `23` cổng TCP ở trạng thái `open`.
-   `977` cổng TCP ở trạng thái `closed`.
-   Danh sách cổng phát hiện được tương ứng với lần `-sT`.

Thời gian quét ghi nhận trong ảnh thực hành: khoảng `4.73 seconds`.

### 6.3. So sánh

Trong lần thực hành này, `-sT` và `-sS` cho cùng số lượng cổng
open/closed và cùng danh sách cổng. Khác biệt chính nằm ở cơ chế thực
hiện scan và yêu cầu quyền.

------------------------------------------------------------------------

## 7. Service / Version Detection

Lệnh:

``` bash
nmap -sV 192.168.126.130
```

Một số kết quả quan trọng:

  Port       Service        Version / thông tin
  ---------- -------------- -------------------------------------
  80/tcp     http           Apache httpd 2.2.8 (Ubuntu) DAV/2
  139/tcp    netbios-ssn    Samba smbd 3.X - 4.X
  445/tcp    microsoft-ds   Samba smbd 3.X - 4.X
  1524/tcp   bindshell      Metasploitable root shell
  3306/tcp   mysql          MySQL 5.0.51a-3ubuntu5
  5432/tcp   postgresql     PostgreSQL 8.3.0 - 8.3.7
  5900/tcp   vnc            VNC protocol 3.3
  8009/tcp   ajp13          Apache JServ Protocol v1.3
  8180/tcp   http           Apache Tomcat/Coyote JSP engine 1.1

Thông tin hệ thống mà Nmap nhận diện:

``` text
OS: Unix
CPE: cpe:/o:linux:linux_kernel
```

Thời gian service detection ghi nhận: khoảng `57.12 seconds`.

------------------------------------------------------------------------

## 8. OS Detection

Lệnh:

``` bash
sudo nmap -O 192.168.126.130
```

Mục đích: sử dụng OS fingerprinting để ước đoán hệ điều hành của mục
tiêu.

Kết quả OS detection cần được hiểu là kết quả fingerprinting, không phải
bằng chứng tuyệt đối về hệ điều hành.

------------------------------------------------------------------------

## 9. NSE --- SMB OS Discovery

Lệnh:

``` bash
sudo nmap -p445 --script smb-os-discovery 192.168.126.130
```

Kết quả:

``` text
445/tcp open microsoft-ds

OS: Unix (Samba 3.0.20-Debian)
Computer name: metasploitable
NetBIOS computer name: metasploitable
Domain name: localdomain
FQDN: metasploitable.localdomain
```

### Kết luận

Cổng SMB `445/tcp` đang mở và NSE nhận diện dịch vụ Samba 3.0.20-Debian.

------------------------------------------------------------------------

## 10. NSE --- Kiểm tra MS17-010

Lệnh:

``` bash
sudo nmap -p445 --script smb-vuln-ms17-010 192.168.126.130
```

Kết quả thực tế không trả về dòng `VULNERABLE`.

### Kết luận

Không có đủ cơ sở để kết luận mục tiêu dễ bị ảnh hưởng bởi MS17-010 chỉ
từ kết quả lần kiểm tra này. Việc không xuất hiện `VULNERABLE` không
được diễn giải thành bằng chứng rằng hệ thống đã được vá.

------------------------------------------------------------------------

## 11. UDP Scan

Đã thực hiện UDP scan trong môi trường lab.

Lệnh sử dụng:

``` bash
sudo nmap -sU -p 53,67,68,69,123,137,138,161 192.168.126.130
```

> **Evidence:** lưu ảnh kết quả UDP scan trong thư mục `screenshots/`.
> Nếu cần ghi chi tiết từng port, bổ sung kết quả thực tế từ ảnh/log vào
> đây; không tự suy đoán trạng thái port.

UDP scan thường chậm hơn TCP và có thể xuất hiện trạng thái
`open|filtered`.

------------------------------------------------------------------------

## 12. Lưu kết quả

Thư mục evidence:

``` text
~/LAB4_Evidence/
```

Các file đã tạo:

``` text
sV.txt
sV.xml
before.txt
after.txt
```

Các file này được dùng làm bằng chứng cho quá trình quét và so sánh
trước/sau hardening.

------------------------------------------------------------------------

## 13. Before / After Hardening

### 13.1. Before

Lệnh khảo sát:

``` bash
nmap -sV 192.168.126.130 -oN ~/LAB4_Evidence/before.txt
```

Trước hardening:

``` text
445/tcp open microsoft-ds
```

### 13.2. Biện pháp hardening

Trên Metasploitable 2, áp dụng firewall rule:

``` bash
sudo iptables -A INPUT -p tcp --dport 445 -j DROP
```

Kiểm tra rule:

``` bash
sudo iptables -L INPUT -n --line-numbers
```

### 13.3. After

Lệnh kiểm tra:

``` bash
sudo nmap -sV -p445 192.168.126.130
```

Kết quả:

``` text
445/tcp filtered microsoft-ds
```

Lưu kết quả:

``` bash
nmap -sV -p445 192.168.126.130 -oN ~/LAB4_Evidence/after.txt
```

### 13.4. So sánh

  Giai đoạn   445/tcp           Ý nghĩa
  ----------- ----------------- -----------------------------------------------
  Before      `open`            SMB có thể được truy cập
  Hardening   `iptables DROP`   Firewall chặn TCP/445
  After       `filtered`        Nmap không thể xác định open/closed do bị lọc

### Kết luận

Trạng thái `445/tcp` thay đổi từ `open` sang `filtered` sau khi áp dụng
firewall. Đây là bằng chứng cho thấy biện pháp hardening đã hạn chế khả
năng truy cập tới dịch vụ SMB.

------------------------------------------------------------------------

## 14. Câu hỏi phân tích

### 1. Open, closed và filtered

-   **Open:** có dịch vụ đang lắng nghe trên cổng.
-   **Closed:** cổng có thể truy cập nhưng không có dịch vụ lắng nghe.
-   **Filtered:** firewall/bộ lọc ngăn probe khiến Nmap không xác định
    được chính xác trạng thái.

Ví dụ trong lab: `445/tcp` chuyển từ `open` sang `filtered` sau
hardening.

### 2. `-sS` và `-sT`

`-sS` sử dụng TCP SYN và thao tác trực tiếp với packet nên thường cần
quyền cao hơn. `-sT` sử dụng TCP Connect của hệ điều hành nên thường
không yêu cầu raw-packet privilege như `-sS`.

### 3. FIN/Xmas/NULL

Các kỹ thuật này phụ thuộc vào cách TCP/IP stack và firewall xử lý các
packet có cờ TCP đặc biệt. Vì vậy kết quả có thể khó diễn giải và có thể
xuất hiện `open|filtered`.

### 4. ACK scan

ACK scan chủ yếu xác định firewall có lọc traffic hay không, phân biệt
`filtered` và `unfiltered`. Nó không dùng để xác nhận cổng đang `open`
như SYN scan.

### 5. UDP scan

UDP không có TCP handshake và nhiều dịch vụ UDP không phản hồi probe.
Firewall cũng có thể âm thầm loại bỏ packet, vì vậy UDP scan thường chậm
và dễ xuất hiện `open|filtered`.

### 6. Vai trò của `-sV`

`-sV` xác định dịch vụ và phiên bản đang chạy. Chỉ biết port 80 open
chưa đủ vì chưa biết phần mềm web và phiên bản cụ thể đang cung cấp dịch
vụ.

### 7. Giới hạn của OS fingerprinting

Kết quả `-O` là fingerprinting/ước đoán và có thể bị ảnh hưởng bởi
firewall, NAT, TCP/IP stack hoặc thiết bị trung gian. Vì vậy không nên
coi kết quả là tuyệt đối.

### 8. NSE timeout

Timeout không đồng nghĩa hệ thống không có lỗ hổng. Nó chỉ cho biết
script không nhận được phản hồi cần thiết; nguyên nhân có thể là
firewall, dịch vụ không phản hồi hoặc điều kiện mạng.

### 9. Before/After hardening

Trong lab, `445/tcp` thay đổi:

``` text
open → filtered
```

sau khi áp dụng `iptables DROP`. Đây là thay đổi port state chứng minh
biện pháp phòng thủ có hiệu lực trong lần kiểm tra.

### 10. Ba biện pháp giảm bề mặt tấn công

1.  Tắt các dịch vụ không cần thiết.
2.  Dùng firewall để giới hạn phạm vi truy cập.
3.  Cập nhật/thay thế hệ điều hành và dịch vụ lỗi thời.

------------------------------------------------------------------------

## 15. PASS / FAIL

  Hạng mục                               Trạng thái
  -------------------------------------- ------------------------------------
  Thiết lập Host-Only                    PASS
  Kali và Metasploitable 2 cùng subnet   PASS
  Host Discovery                         PASS
  TCP `-sT`                              PASS
  TCP `-sS`                              PASS
  Service detection `-sV`                PASS
  OS detection `-O`                      PASS
  NSE `smb-os-discovery`                 PASS
  NSE `smb-vuln-ms17-010`                PASS --- không trả về `VULNERABLE`
  Lưu TXT/XML                            PASS
  UDP scan                               PASS
  Before/After hardening                 PASS
  Firewall 445/tcp                       PASS

------------------------------------------------------------------------

## 16. Lỗi và cách xử lý

### Lỗi 1 --- Kali ban đầu sử dụng NAT

**Vấn đề:** Kali ban đầu sử dụng NAT nên không cùng mạng lab với
Metasploitable 2.

**Cách xử lý:** Chuyển Network Adapter của Kali sang Host-only.

**Kết quả:** Kali nhận `192.168.126.131/24`, Metasploitable 2 nhận
`192.168.126.130/24`.

### Lỗi 2 --- Kiểm tra quyền khi sử dụng Nmap

Một số kỹ thuật như `-sS`, `-O` và NSE được chạy với `sudo` để có quyền
cần thiết.

### Lỗi 3 --- MS17-010 không trả về `VULNERABLE`

**Cách xử lý:** Không tự kết luận mục tiêu đã được vá hoặc không có lỗ
hổng. Ghi nhận đúng kết quả: script không trả về `VULNERABLE`.

------------------------------------------------------------------------

## 17. Evidence

Đề nghị đặt ảnh chụp trong:

``` text
screenshots/
```

Danh sách:

``` text
01_metasploitable_ifconfig.png
02_kali_ip.png
03_ping.png
04_host_discovery.png
05_tcp_sT.png
06_tcp_sS.png
07_service_sV.png
08_os_detection.png
09_smb_os_discovery.png
10_ms17_010.png
11_udp_scan.png
12_iptables_rule.png
13_after_hardening.png
```

Các ảnh phải được chụp trực tiếp từ môi trường lab.

------------------------------------------------------------------------

## 18. Lưu ý an toàn

-   Chỉ quét các máy ảo thuộc môi trường thực hành.
-   Sử dụng mạng Host-Only cho lab.
-   Không quét IP/domain bên ngoài khi chưa được phép.
-   Không thực hiện khai thác lỗ hổng trong bài NSE.
-   Không đưa mật khẩu, token, cookie hoặc thông tin cá nhân vào
    repository.
-   Không đưa installer hoặc executable không cần thiết vào repository.

------------------------------------------------------------------------

## 19. Kết quả tổng kết

LAB4 đã thực hiện quy trình khảo sát bề mặt mạng bằng Nmap trên môi
trường VMware Workstation gồm Kali Linux và Metasploitable 2. Các kỹ
thuật được thực hiện gồm Host Discovery, TCP Scan, UDP Scan,
Service/Version Detection, OS Detection và NSE. Kết quả được lưu thành
file evidence và thực hiện so sánh Before/After bằng cách áp dụng
firewall chặn `445/tcp`.

Biện pháp hardening đã làm trạng thái cổng:

``` text
445/tcp: OPEN → FILTERED
```

qua đó chứng minh khả năng hạn chế bề mặt truy cập của dịch vụ SMB trong
môi trường lab.
