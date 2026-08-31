# CHUỖI WORKFLOW CHO GIT 

## Khi 2 người làm 2 tính năng 
- Giả sử: Một thư mục được `clone` về 2 máy hoặc 2 tài khoản 

    ```txt
    [GitHub] Repo_ProjectNew

    [An]: thực hiện feature/trang-chu

    [Khuê]: thực hiện feature/trang-lien-he

    [2 người] sửa cùng file còig.txt -> sẽ có conflict 

    ```

## Giả sử 

- [1] **An làm xong và tạo PR** 

```bash 
    # [An] cập nhật + tạo nhánh 
    git pull    #main mới nhất

    git switch -c feature/trang-chu

    # [An] làm việc
    echo "<h1>Trang chủ </h1>" > home.html
    echo "title = Trang chủ" > config.txt        # [An] đã sửa config.txt 

    git add .
    git commit -m"Thêm trang chủ" 

    # [An] Đẩy nhánh + tạo PR 
    git push -u origin feature/trang-chu

    # -> [GitHub] tạo PR cho feature/trang-chu --> main

```

- [2] **Khuê làm xong và tạo PR** 

```bash 
    # [Khuê] cập nhật + tạo nhánh 
    git pull    #main mới nhất

    git switch -c feature/trang-lien-he

    # [Khuê] làm việc
    echo "<h1>Trang liên hệ </h1>" > home.html
    echo "title = Trang liên hệ" > config.txt        # [Khuê] đã sửa config.txt, CÙNG DÒNG với An

    git add .
    git commit -m"Thêm trang liên hệ" 

    # [Khuê] Đẩy nhánh + tạo PR 
    git push -u origin feature/trang-lien-he

    # -> [GitHub] tạo PR cho feature/trang-lien-he --> main
```

- [3] **2PR được tạo** đang chờ merge và đều cùng sửa config.txt cùng dòng --> mần móng conflict xuất hiện 

- [4] **Review và merge PR của An đầu tiên**
    - Khuê mở PR của An, xem Files changed --> *Approve*

    - Merge PR của An -> `main` giờ có `trang-chu` + `config.txt` của An

    - Xóa nhánh `feature/trang-lien-he` của An

    - PR đầu chẳng có chuyện gì xảy ra, vì `main` hiện chỉ có nhánh An tách

- [5] **PR thứ 2 - CONFLICT**
    - Giờ Khuê muốn merge, nhưng `main` đã đổi `config.txt`, do PR của An đã duyệt 

    - GitHub: **This branch has conflicts that must be resolved** 

- [6] Giải quyết??? 
    - Kéo main có thay đổi vào nhánh về máy: `git pull orgin main`

    - Mở `config.txt`

    ```txt
        <<<<<<< HEAD
        title = Lien He        (bản của Khue)
        =======
        title = Trang Chu      (bản của An, vừa vào main)
        >>>>>>> main
    ```

    - Giả sử: cách giải quyết ở đây kết hợp 2 nhánh 

    ```txt
    title = Trang chu va lien he
    ``` 

    - Hoàn tất: 
    
```bash
    git add config.txt
    git commnit -m"Giải quyết conflict config.txt của main"
    git push    #cập nhật PR
``` 
> PR hiện tại hết conflict -> Approve -> merge cả hai tính năng vào main an toàn 

## Tóm tắt

| Bước | Việc | 
|------|------|
| Bắt đầu | `git pull` main mới nhất | 
| Tạo nhánh | `git switch -c feature/x` | 
| Làm + đẩy | commit -> `git push -u origin feature/x` | 
| Đề nghị gộp | tạp PR trên Github | 
| Review | comment + Approve | 
| Conflict | pull nhánh vào main + giải quyết | 
| Hoàn tất | merge PR + xóa nhánh | 