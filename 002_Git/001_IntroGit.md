# GIT CHO QUẢN LÍ DỰ ÁN CÁ NHÂN

## 1. KHỞI TẠO DỰ ÁN
- Tạo repo mới
```bash  
    # Tạo thư mục dự án và vào đó
    mkdir web-mini
    cd web-mini

    #Khởi tạo repo mới, trống
    git init
```

-> Repo đã sẵn sàng

## 2. COMMIT 1: CẤU TRÚC BAN ĐẦU + .gitignore
``` bash
# Tạo .gitignore trước
echo ".env" > .gitignore
echo "*.log" >> .gitignore
echo ".DS_Store" >> .gitignore

# Tạo file đầu tiên
echo "# Web Mini - dự án học Git" > readme.md

git add .
git commit -m "Khởi tạo dự án với readme và .gitignore"
```

## 3. COMMIT 2: THEM TRANG HTML
- Thêm một tính năng rõ ràng (trang chủ)

```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Web Mini</title></head>
<body>
  <h1>Xin chào Git!</h1>
</body>
</html>
EOF

git add index.html
git commit -m "Thêm trang chủ index.html"
```

## 3. COMMIT 3: THÊM CSS
- Thêm phần giao diện

```bash
cat > style.css << 'EOF'
body { font-family: sans-serif; background: #f5f5f5; }
h1 { color: #2563eb; }
EOF

git add style.css
git commit -m "Thêm file style.css cho giao diện"
```

## 4. COMMIT 4: SỬA NỘI DUNG + COMMIT 5: HOÀN THIỆN

```bash
# Commit 4: sửa file đã có (nhớ bài "sửa và commit")
echo "  <p>Trang web đầu tiên quản lý bằng Git.</p>" >> index.html
git add index.html
git commit -m "Cập nhật nội dung trang chủ"

# Commit 5: hoàn thiện readme
echo "Dự án nhỏ để thực hành Git cơ bản." >> readme.md
git add readme.md
git commit -m "Bổ sung mô tả dự án trong readme"
```
## XÊM LẠI LỊCH SỬ 

```bash
git log --oneline
```
- Kì vọng: đọc được câu chuyện phát triển: khởi tạo → thêm trang → thêm giao diện → cập nhật nội dung → hoàn thiện.

- Quy trình đơn giản: 
    - Tạo mới: `init` tạo 1 lần ban đầu.
    - Tạo và cập nhật: `.gitignore` 
    - Cập nhật, chốt thay đổi: sửa code → `git add` → `git commit` với message rõ → thỉnh thoảng `git log` xem lại.
    - Xem lịch sử: `git log --oneline`
    - Xem một commit `git show <hash>` 