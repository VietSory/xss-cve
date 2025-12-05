# 🚀 CVE-2020-29606: Pluck CMS Stored XSS Exploitation

![Metasploit](https://img.shields.io/badge/Metasploit-Framework-333333?style=for-the-badge&logo=metasploit&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Ruby](https://img.shields.io/badge/Language-Ruby-CC342D?style=for-the-badge&logo=ruby&logoColor=white)
![Status](https://img.shields.io/badge/Status-PoC%20Verified-success?style=for-the-badge)

> **Dự án phát triển Module Metasploit khai thác lỗ hổng Stored XSS trên Pluck CMS v4.7.13.**

## 📑 Mục lục
- [Giới thiệu](#-giới-thiệu)
- [Cấu trúc thư mục](#-cấu-trúc-thư-mục)
- [Phân tích kỹ thuật](#-phân-tích-kỹ-thuật)
- [Hướng dẫn khai thác](#-hướng-dẫn-khai-thác)
---

## 📖 Giới thiệu

Module này được thiết kế để tự động hóa quá trình khai thác lỗ hổng **CVE-2020-29606**. Đây là lỗ hổng **Stored Cross-Site Scripting (XSS)** nằm trong module "Albums" của Pluck CMS.

Kẻ tấn công (đã có quyền đăng nhập) có thể chèn mã JavaScript độc hại vào tên của Album ảnh. Mã này sẽ được lưu lại vĩnh viễn trong hệ thống và thực thi mỗi khi quản trị viên truy cập vào trang quản lý Album.

* **Mục tiêu:** Pluck CMS 4.7.13
* **Loại lỗi:** Stored XSS (Authenticated)
* **Module Metasploit:** `exploit/linux/http/cve_2020_29606_xss`
---

## 📂 Cấu trúc thư mục

```text
.
├── xss_basic/              # Demo các lỗi XSS cơ bản (Reflected, DOM, Stored)
├── CMS/                    # Chứa mã nguồn và dữ liệu của CVE
├── modules/
│   ├── pluck_xss.rb        # Module 1: Stored XSS (CVE-2020-29606)
│   └── pluck_cms_theme_rce.rb # Module 2: RCE via Theme Upload (CVE-2020-29607)
├── docker-compose.yml      # File cấu hình môi trường Lab
└── README.md               # Tài liệu hướng dẫn

```

## 🔍 Phân tích kỹ thuật

Lỗ hổng xảy ra do thiếu cơ chế kiểm soát đầu vào (Input Sanitization) và mã hóa đầu ra (Output Encoding) tại file xử lý Albums.

Cụ thể, ứng dụng nhận biến `$_POST['album_name']` từ người dùng nhưng **không sử dụng** các hàm an toàn như `htmlspecialchars()` để loại bỏ các ký tự đặc biệt của HTML/JS trước khi lưu trữ và hiển thị.

**Đoạn code mô phỏng lỗ hổng (Vulnerable Logic):**

```php
// data/inc/modules/albums.php
if (isset($_POST['create_album'])) {
    $album_name = $_POST['album_name'];

    // ❌ LỖI BẢO MẬT: Biến $album_name được sử dụng trực tiếp.
    // Nếu kẻ tấn công nhập: <script>alert(1)</script>
    // Hệ thống sẽ hiểu đây là code thực thi thay vì văn bản thuần.
```
## ⚔️ Hướng dẫn Khai thác & Demo (Full Chain)

Dự án thực hiện kịch bản tấn công chuỗi (Kill-chain):
1.  **Giai đoạn 1 (XSS):** Tấn công Stored XSS để chiếm quyền điều khiển (Session Hijacking).
2.  **Giai đoạn 2 (RCE):** Sử dụng phiên làm việc đã chiếm được để upload mã độc (Reverse Shell) và chiếm quyền server.

### 📍 Giai đoạn 1: Stored XSS (CVE-2020-29606)

**Mục tiêu:** Chèn mã độc vào Album ảnh để đánh cắp Cookie của Admin.

1.  **Cài đặt module:**
    ```bash
    cp modules/pluck_xss.rb ~/.msf4/modules/auxiliary/scanner/http/plunk_xss.rb
    ```

2.  **Thực hiện tấn công:**
    ```bash
    msfconsole
    use auxiliary/scanner/http/plunk_xss
    set RHOSTS ip_cve
    RUN
    ```
    ✅ **Kết quả:** Payload XSS được lưu vào DB. Khi Admin truy cập, Cookie sẽ bị lộ.

---
### 📍 Giai đoạn 2: Remote Code Execution (RCE)

**Mục tiêu:** Sau khi có quyền truy cập (hoặc giả lập đã có quyền), sử dụng lỗ hổng trong tính năng "Theme Edit" để thực thi mã từ xa.

1.  **Cài đặt module:**
    ```bash
    cp modules/pluck_cms_theme_rce.rb ~/.msf4/modules/exploits/linux/http/pluck_cms_theme_rce.rb
    ```

2.  **Thực hiện tấn công:**
    ```bash
    msfconsole
    use exploits/linux/http/pluck_cms_theme_rce
    set RHOSTS ip_cve
    set COOKIE_SESSION
    exploit
    ```
    ✅ **Kết quả:** RCE vào được server .
