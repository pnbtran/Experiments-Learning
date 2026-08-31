# GITFLOW - CÁCH TRIỂN KHAI 

## 1. Các nhánh trong Gitflow

- Trong repo trên Git có thể có các nhánh: 

| Nhánh | Vai trò | Tồn tại | 
|-------|---------|---------|
| `main` | Bản đã phát hành (ổn định nhất) | Vĩnh viễn | 
| `deverlop` | Nơi tích hợp mọi tính năng (bản kế tiếp) | Vĩnh viễn | 
| `feature/*` | Phát triển 1 tính năng | Tạm (xóa sau khi gộp vào deverlop) | 
| `release/*` | Chuẩn bị + kiểm tra một phiên bản trước phát hành | Tạm | 
| `hotfix/*` | Sửa lỗi KHẨN trên bản phát hành | Tạm | 

- Luồng đi của một tính năng: tính năng luôn tách và gộp vào `deverlop` - không đụng vào `main`, `main` chỉ nhận khi phát hành qua `release` 

```bash
# Tính năng bắt đầu từ deverlop 
    git switch deverlop
    git switch -c feature/gio-hang

    # Làm việc ..., commit ... 
    git switch deverlop 
    git merge feature/gio-hang          # Gộp về nhánh deverlop 
    git branch -d feature/gio-hang
``` 
- Release và Hitfix, khác gì nhau? 
    - Release branch - khi `deverlop` đủ tính năng cho 1 phiên bản --> tạo nhánh `release/1.0.0` --> "đóng băng" phiên bản + test + sửa lỗi  --> gộp `release/1.0.0`  vào `deverlop` 

    - Hotfix branch - có lỗi nghiêm trọng trên bản phát hành (`main` ?) - không thể chờ chu trình deverlop-release, cần sửa gấp --> tạo thẳng `hitfix/*` từ main để sửa, xong thì gộp vào main (vá lỗi) lẫn deverlop  

```bash 
# RELEASE-BRANCH
    git switch -c release/1.0 deverlop
        
    # ... test, sửa lỗi nhỏ, cập nhật phiên bản, ... 
    git switch main && git merge release/1.0.0 && git tag v1.0.0    #phát hành

    git switch deverlop && git merge release/1.0.0                  # gộp release vào deverlop


    # HOTFIX-BRANCH
    git switch -c hotfix/loi-thanh-toan main 

    # ... sửa lỗi ... 
    git switch main && git merge hotfix/loi-thanh-toan && git tag v1.0.1

    git switch deverlop && git merge hotfix/loi-thanh-toan
```

- Git-flow và GitHub-flow: 

| -- | Git-flow | GitHub-flow | 
|----|----------|-------------|
| Số nhánh | 5 (main/deverlop/feature/release/hotfix) | 2 (main/feature) | 
| Phù hợp | Sản phẩm phiên bản cao (app cài đặt, thư viện) | web/SaaS deploy liên tục |
| Độ phức tạp | Cao | Thấp | 

- Lệnh Git được dùng để tạo nhánh mới và chuyển sang nhánh đó `git checkout -b`

## 2. Rebase 
- So sánh: 
    - **Merge** - tạo merge + commit nối hàng 

    - **Rebase** - dời commit feature lên main thành đường thẳng

    --> kết quả giống nhưng lịch sử khác, merge giữa nhánh đan xen + merge commit; rebase làm thẳng như chưa từng tách nhánh 

    ```txt
        ### Merge
        o---o---o (main)
            \       \
            o---o---M (feature, có merge commit M)

        ### Rebase
        o---o---o (main)
                \
                o'---o' (feature - các commit được "dời" lên, lịch sử thẳng)
    ```

- Thực hiện rebase 
```bash
    # đang ở feature, muốn cập nhật code vào main 
    git switch feature/tinh-nang-moi
    git rebase main 

    git switch main 
    git merge feature/tinh-nang-moi     #fast-forward, không tạo merge commit
```
- Lịch sử commit khác 

| -- | Merge | Rebase | 
|----|-------|--------|
| Lịch sử | đầy đủ, có nhánh, có merge commit | thẳng, sạch |
| Ưu điểm | Rõ ràng, minh mạch | Dễ đọc, gọn gàng | 
| Nhược điểm | Rối, khó đọc khi nhiều nhánh | Viết lại lịch sử (hash đổi) | 

- Chỉ rebase nhánh trên máy mình, chưa push hoặc nhánh riêng chưa ai dùng. Nếu rebase nhánh đã push (người khác đã pull) -> thảm họa conflict (hash đã đổi), hỏng repo 

- Conflict khi rebase: rebase cũng có thể gặp conflict như merge; nhưng khác, rebase đắp từng commit 1 nên conflict xảy ra theo commit 

```bash 
git rebase --continue   # sau khi giải conflict thì tiếp tục 
git rebase --abort      # rút lui an toàn (gần giống merge --abort)

git rebase main               # rebase lên main
git push --force-with-lease   # push sau rebase (nhánh riêng)
```

- Thứ tự rebase đúng từ feature vào main 
    - [1] Cập nhật nhánh main với các thay đổi mới nhất từ remote.
    
    - [2] Chuyển sang nhánh feature bằng git checkout.

    - [3] Chạy lệnh rebase để di chuyển commit của feature lên main.

    - [4] Giải quyết xung đột nếu có trong quá trình rebase.

    - [5] Kiểm tra lại code và lịch sử sau khi rebase hoàn tất.

## 3. Làm Gọn Lịch Sử với Squash Commits và Interactive Rebase
- Mở interactive rebase, nhánh feature có 4 commit cần gộp 

```bash
    git log --oneline
    # d4e5f6 fix typo
    # c3d4e5 quên thêm file css
    # b2c3d4 sửa lỗi validate
    # a1b2c3 thêm form đăng nhập

    git rebase -i HEAD~4      # biên tập 4 commit gần nhất
```

- Squash - gộp commit: Git gộp cả 4 commit thành một. 
    - `fixup` như squash nhưng vứt luôn message (hữu ích cho commit "fix typo" không cần message) 

    - `reword` chỉ sửa message của commit cũ (không sửa được bằng `--amend` do không phải commit gần nhất) 

    - `drop` xóa hẳn commit khỏi lịch sử

| Lệnh | Viết tắt | tác dụng | 
|------|----------|----------|
| `pick` | `p` | giữ commint nguyên | 
| `squash` | `s` | Gộp vào commit trên (giữ cả 2 message để sửa) | 
| `fixup` | `f` | Gộp vào commit trên (bỏ message) | 
| `reword` | `r` | Giữ commit nhưng sửa message | 
| `edit` | `e` | Đừng để sửa nội dung commit | 
| `drop` | `d | Xóa commit này | 

- Interactive rebase còn cho sắp xếp lại commit (đổi thứ tự các dòng) và xóa (đổi thành drop hoặc xóa dòng). Cho phép biên tập lại commit trước khi xuất bản, nhưng đây là thao tác viết lại lịch sủ (hash đổi). 

> Nếu đã push nhánh feature (riêng của bạn, chưa ai dùng) rồi squash, cần `git push --force-with-lease` để cập nhật. Chấp nhận được nếu chắc chắn không ai khác dùng nhánh đó. Nhưng với nhánh chung nhiều người → tuyệt đối không.

## 4. Tạo và Sử Dụng Tags 
- Tạo Tag - nhãn `v1.0` vào commit hiện tại (HEAD) 

```bash 
    # Annotated tag (khuyên dùng) - có message, tác giả, ngày
    git tag -a v1.0 -m "Phát hành phiên bản 1.0"

    # Xem các tag
    git tag
    # v1.0

    # Xem chi tiết một tag
    git show v1.0
```
- Annotated và Lightweight tag 

| -- | Annotated `-a` |  Lightweight | 
|----|----------------|--------------|
| Cú pháp | `git tag -a v1.0 -m"..." ` | `git tag -a v1.0` |
| Chứa | Message + tác giả + ngày | chỉ là con trỏ đến commit | 
| Dành cho | phát hành chính thức | nhãn tạm/ cá nhân | 

- Gắn tag cho commit cũ: qua hash, gắn bù khi version đã phát hành nhưng quên tag

```bash
    git log --oneline
    # c3d4e5 ...
    # a1b2c3 (commit muốn gắn tag)

    git tag -a v0.9 a1b2c3 -m "Bản beta 0.9"    # gắn tag cho commit cũ
```

- Đẩy tag lên remote: **[FACT]** `git push` không đẩy tag, mà cần đẩy riêng 
    - Tạo tag ở local, git push như bình thường, nhưng tag không lên GitHub. 
    
    - Phải `git push origin <tag>` hoặc `--tags`. Sau khi đẩy, tag xuất hiện trong mục Releases/Tags trên GitHub.

```bash
    git push origin v1.0          # đẩy một tag cụ thể
    git push origin --tags        # đẩy TẤT CẢ tag
```

- Semantic Versioning — chuẩn đặt số phiên bản
    - PATCH (v1.0.0→v1.0.1): chỉ sửa lỗi, không thêm gì mới.

    - MINOR (v1.0.1→v1.1.0): thêm tính năng mới, code cũ vẫn chạy.

    - MAJOR (v1.1.0→v2.0.0): thay đổi phá vỡ tương thích (code cũ có thể hỏng).

```txt
        v  2  .  3  .  1
        │     │     │
        │     │     └─ PATCH: sửa lỗi (không đổi tính năng) → v2.3.2
        │     └─────── MINOR: thêm tính năng (tương thích ngược) → v2.4.0
        └───────────── MAJOR: thay đổi lớn (phá tương thích) → v3.0.0
``` 

- Dùng tag để quay về/ tạo release 

```bash
# Quay về xem code tại phiên bản v1.0
    git checkout v1.0        # (chế độ "detached HEAD" - chỉ xem)

    # Tạo nhánh từ một tag (vd để hotfix bản cũ)
    git switch -c hotfix/v1.0.1 v1.0

    # Xóa tag
    git tag -d v1.0                    # xóa local
    git push origin --delete v1.0      # xóa trên remote
``` 

- [FACT] Branch di chuyển nhưng tag cố định 

```txt
        Branch:  main ──●──●──●──►  (con trỏ tiến khi commit mới)
        Tag:     v1.0 ──●           (gắn cứng vào commit này, không đổi)
```

| Lệnh | Nhiệm vụ | 
|------|----------|
| `git tag -a v1.0 -m "..."` | tạo tag | 
| `git tag / git show v1.0`  | xem tag | 
| `git tag -a v0.9 <hash> -m "..."` | tag cho commit cũ | 
| `git push origin --tags` | đẩy tag | 
| `git tag -d v1.0 + push --delete` | xóa tag | 

## 5. Pre-commit Hook basic
- Mỗi repo có sẵn thư mục hooks: để kích hoạt hook, tạo file không có đuôi

``` bash
    ls .git/hooks/
    # pre-commit.sample  commit-msg.sample  pre-push.sample ...
``` 
- Tạo Pre-commit Hook đơn giản: Tạo file .git/hooks/pre-commit:

```bash
    #!/bin/sh
    # pre-commit hook: chặn commit nếu code còn "console.log"

    echo "Đang kiểm tra code trước khi commit..."

    # Tìm console.log trong các file sắp commit
    if git diff --cached --name-only | xargs grep -n "console.log" 2>/dev/null; then

        echo "Phát hiện console.log! Vui lòng xóa trước khi commit."
        exit 1        # exit ≠ 0 -> CHẶN commit
    fi

    echo "Kiểm tra OK"
    exit 0            # exit 0 -> CHO PHÉP commit

    # Cho quyền thực thi
    chmod +x .git/hooks/pre-commit
```

- Các hook phổ biến: 

| Hook | Chạy khi | Dùng để | 
|------|----------|---------|
| `pre-commit` | trước khi commit | lint, format, test nhanh | 
| `commit-msg` | sau khi nhập message | kiểm định dạng message |
| `pre-push` | trước khi push | chạy test đầy đủ trước khi push | 
| `post-merge` | sau khi merge | cài lại dêpndencies (`npm install`) | 

- Git hooks = script tự chạy; `pre-commit` chạy trước commit, exit ≠ 0 CHẶN. 

```txt
        git commit
            │
            ▼
        Git chuẩn bị tạo commit
            │
            ▼
        pre-commit hook
            │
            ├── kiểm tra code
            ├── chạy formatter
            ├── chạy test
            ├── kiểm tra secret/password
            └── nếu lỗi → chặn commit
            │
            ▼
        Tạo commit
```

- Đặt trong `.git/hooks/pre-commit` + `chmod +x`; hook không theo repo → dùng Husky để chia sẻ.

- Hook = lưới đầu (tại máy, `--no-verify` bỏ qua được); CI/CD = lưới cuối bắt buộc.

- Pre-commit hook chạy ngay trước khi commit được tạo, cho phép kiểm tra hoặc thay đổi trước khi lưu commit.

## 6. Code Review và Pull Request Comments 

- Lí do cần code review: 
    - *Bắt lỗi sớm* - bug, edge code, lỗ hỏng logic,... 

    - *Chia sẻ kiến thức* 

    - *Nâng chuẩn* - thống nhất cách làm việc

    - *Nhất quán* - giữ codebase cùng quy ước 

- Phân loại mức độ góp ý comment 

| Mức | Ý nghĩa | Ví dụ | 
|-----|---------|-------| 
| Blocking | phải sửa trước khi merge | bug, lỗ hỏng bảo mật | 
| Suggestion | nên cân nhắc, không bắt buộc | có thể tách hàm, gọn code | 
| Nit (nitpick) | nhỏ nhặt, tùy tác giả | thừa dòng trống | 
| Praise | khen điều tốt | -- | 

- GitHub review for workflow 

```txt
    1. Xem tab "Files changed"
    2. Comment trên các dòng (bấm "Start a review" để gom lại)
    3. Bấm "Review changes" → chọn kết luận:
    - Comment: góp ý chung, không phán quyết
    - Approve: duyệt, cho merge ✅
    - Request changes: yêu cầu sửa trước khi merge ❌
    4. Submit review (mọi comment gửi cùng lúc)
``` 
## Tổng kết 
- Vòng đời 1 dự án được quản lí bằng Git và GitHub 
    - [1] Khởi tạo: git init → .gitignore → commit đầu → tạo repo GitHub → push

    - [2] Phát triển: Mỗi tính năng: nhánh feature → commnit → push → PR → review → merge

    - [3] Kiểm tra, rà soát: Xử lí conflict → dọn lịch sử (rebase/squash), pre-commit hook 

    - [4] Phát hành: Gắn tag phiên bản (SemVer) → tạo release trên GitHub

### Giai đoạn 1 - Khởi tạo 
- Dự án đã lên GitHub 

``` bash 
    mkdir todo-app && cd todo-app
    git init
    echo "node_modules/" > .gitignore     # ignore ngay từ đầu 
    echo ".env" >> .gitignore
    echo "# Todo App" > README.md
    git add .
    git commit -m "Khởi tạo dự án với README và .gitignore"

    # Tạo repo trên GitHub, rồi:
    git remote add origin <URL>
    git push -u origin main
```

### Giai đoạn 2: Phát triển 
- Mỗi nhánh một feature + quy trình (commnit → push → PR → review → merge)

```bash
    # Tính năng 1: thêm todo
    git switch -c feature/them-todo
    # ... code, commit nhiều lần ...
    git rebase -i HEAD~3          # squash commit vụn thành 1 
    git push -u origin feature/them-todo
    # → Tạo PR trên GitHub → đồng đội review → merge (Squash and merge)

    # Tính năng 2: xóa todo (làm song song)
    git switch main && git pull
    git switch -c feature/xoa-todo
    # ... tương tự ...
```

### Giai đoạn 3: Kiểm tra và rà soát + Giai đoạn 4: Phát hành  
- Dự án có lịch sử sạch, phiên bản tag rõ ràng, hook đảm bảo chất lượng, release chuẩn 

```bash
# Thêm pre-commit hook kiểm tra 
# .git/hooks/pre-commit chạy lint trước mỗi commit

# Khi đủ tính năng cho phiên bản 1.0, phát hành:
git switch main && git pull
git tag -a v1.0.0 -m "Phát hành phiên bản 1.0 - todo cơ bản"  
git push origin --tags
# → Tạo Release trên GitHub với changelog

# Sau này sửa lỗi:
git tag -a v1.0.1 -m "Sửa lỗi hiển thị"   # PATCH tăng (SemVer)
git push origin --tags
```

### Lưu ý
- Commit thường xuyên, mỗi commit là một việc rõ 

- `.gitignore` ngay từ đầu (không commit file bí mật) 

- Một tính năng = một nhánh, không code thẳng lên main 

- PR + review cho mọi thay đổi vào main 

- Pull thường xuyên để giảm conflict 

- Dọn lịch sử (squash) trước merge → main sạch 

- Tag các phiên bản theo SemVer

- Chỉ viết lại lịch sử (amend/release) khi chưa chia sẻ 

- Hook/CI tự động kiểm tra chất lượng 