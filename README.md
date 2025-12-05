# 🚀 CVE-2020-29606: Pluck CMS Stored XSS Exploitation

![Metasploit](https://img.shields.io/badge/Metasploit-Framework-333333?style=for-the-badge&logo=metasploit&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Ruby](https://img.shields.io/badge/Language-Ruby-CC342D?style=for-the-badge&logo=ruby&logoColor=white)
![Status](https://img.shields.io/badge/Status-PoC%20Verified-success?style=for-the-badge)

> **Dự án phát triển Module Metasploit khai thác lỗ hổng Stored XSS trên Pluck CMS v4.7.13.**

## 📑 Mục lục
- [Giới thiệu](#-giới-thiệu)
- [Phân tích kỹ thuật](#-phân-tích-kỹ-thuật)
- [Cài đặt môi trường](#-cài-đặt-môi-trường)
- [Hướng dẫn khai thác](#-hướng-dẫn-khai-thác)
- [Kết quả Demo](#-kết-quả-demo)
- [Disclaimer](#-disclaimer)

---

## 📖 Giới thiệu

Module này được thiết kế để tự động hóa quá trình khai thác lỗ hổng **CVE-2020-29606**. Đây là lỗ hổng **Stored Cross-Site Scripting (XSS)** nằm trong module "Albums" của Pluck CMS.

Kẻ tấn công (đã có quyền đăng nhập) có thể chèn mã JavaScript độc hại vào tên của Album ảnh. Mã này sẽ được lưu lại vĩnh viễn trong hệ thống và thực thi mỗi khi quản trị viên truy cập vào trang quản lý Album.

* **Mục tiêu:** Pluck CMS 4.7.13
* **Loại lỗi:** Stored XSS (Authenticated)
* **Module Metasploit:** `exploit/linux/http/cve_2020_29606_xss`

---

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
    
    save_album_to_db($
