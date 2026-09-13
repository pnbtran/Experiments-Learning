# 005 TIẾN TRÌNH LÀ GÌ? PROGRAM VÀ PROCESS
## 1. Phân biệt PROGRAM và PROCESS
- Khi tải Google Chrome về, máy bạn có một file `chrome.exe`  (Windows) hoặc Google Chrome.app (macOS) trên ổ cứng. *File đó là program*: một tập hợp lệnh máy lưu sẵn, nằm im, không tiêu tốn CPU/RAM (ngoài chỗ lưu trên đĩa).

- Khi bạn nhấp đúp để mở Chrome, hệ điều hành sẽ: 
    (1) đọc file chương trình từ đĩa, 
    
    (2) nạp các lệnh vào RAM, 
    
    (3) cấp phát tài nguyên (bộ nhớ, thanh ghi, stack), 
    
    (4) cho CPU thực thi lệnh đầu tiên. 
    
    Lúc này Chrome trở thành một process: một thực thể đang sống, đang chạy, đang dùng CPU và RAM thật.

- Phân biệt: 

| Tiêu chí | Program | Process | 
|----------|---------|---------|
| Bản chất | thụ động (passive) | chủ động (active) |
| Ở đâu | file lưu ở ổ đĩa | trong RAM khi đang thực hiện | 
| Tài nguyên | Không tốn RAM/CPU | được cấp RAM, CPU và thanh ghi | 
| Thời gian | rồn tại lâu dài | có hạn: chạy -> kết thúc | 
| Định danh | trên file | PID | 
| Ví dụ | file `chrome.exe` trên đĩa | cửa sổ chrome đang mở | 

## 2. Một program và nhiều process
- Một program có thể sinh ra nhiều process cùng chạy song song: 
    - Mở cùng lúc 3 cửa sổ Notepad, dù đều chạy từ file `notepad.exe`

    - Mở 50+ tab trình duyệt chrome --> nhiều process đang chạy cùng lúc

    - Hai người cùng chạy lệnh Python trên cùng server --> hai process Python riêng biệt được cùng thực hiện

- Mỗi process có PID khác nhau và vùng nhớ riêng.  

## 3. Không gian địa chỉ của một tiến trình
- Khi một process chạy, hệ điều hành cấp cho nó một không gian địa chỉ (address space) trong RAM và chỉ process đó được dùng, chia thành các vùng (segment), mỗi vùng nhỏ lại thực hiện 1 nhiệm vụ: 

| Vùng | Chứa gì | Nhiệm vụ | 
|------|---------|----------|
| Text (code) | các lệnh máy của chương trình | chỉ đọc (read-only), kích thước cố định | 
| Data | biến toàn cục (global), static | khởi tạo sẵn, kích thước cố định |
| Heap | Bộ nhớ cấp phát động (malloc/new) | Lớn lên, xin thêm trong lúc chạy | 
| Stack | lời gọi hàm, biến cục bộ (local), tham số | Phình/co theo lời gọi của hàm |

> Heap và Stack nằm ở 2 đầu đối diện, mọc về phía nhau: Heap lớn từ dưới lên và Stack từ trên xuống, cùng chia sẻ cùng trống ở giữa. 

```txt
   Địa chỉ CAO
   ┌────────────────────────────┐
   │   STACK   │  (mọc XUỐNG ↓) │    lời gọi hàm, biến local
   │      ▼                     │
   │   (vùng trống dùng chung)  │    ← Heap & Stack mọc về phía nhau
   │      ▲                     │
   │   HEAP    │  (mọc LÊN ↑)   │     malloc/new - cấp phát động
   ├────────────────────────────┤
   │   DATA                     │     biến global, static
   ├────────────────────────────┤
   │   TEXT (CODE)              │     lệnh chương trình (read-only)
   └────────────────────────────┘
   Địa chỉ THẤP

```
> ⚠️ Gotcha: Khi Stack đè vào Heap (hoặc ngược lại), ta gặp lỗi nổi tiếng stack overflow (thường do đệ quy vô hạn) hoặc out of memory. Tên trang web hỏi đáp lập trình Stack Overflow chính là lấy từ lỗi này!

## 4. Các thành phần của process
- Một tiến trình (process) gồm: 
    - PID (Process ID): số định danh duy nhất cho mỗi tiến trình

    - Code (text): các lệnh chương trình đang thực thi

    - Dữ liệu (Data + Heap): biến toàn cục và bộ nhớ động

    - Program Counter (PC): Con trở chỉ tới lệnh kế tiếp sẽ chạy

    - Thanh ghi (Registers): Vùng lưu tạm tốc độ cao trong CPU

    - Stack: Ngăn xếp lời gọi hàm và biến cục bộ 

> Trong đó, Program Counter và Registers có thể xem "ảnh chụp" trạng thái thực thi. Khi OS dừng một process để thực hiện process khác (chuyển ngữ cảnh, context switch), nó lưu lại PC và thanh ghi process cũ, rồi khôi phục process mới. 

## 5. Hệ điều hành tạo ra tiến trình như nào? 
- Một tiến trình được tạo ra từ một tiến trình khác, do đó tiến trình tạo ta gọi là cha (parent) và tiến trình mới được gọi là con (child). 

### 5.1. Trên Unix/Linux: fork() + exec()
- Mô hình điển hình: cha gọi `fork()` tạo con, con gọi `exec()` để biến thành chương trình mong muốn. Ví dụ khi bạn gõ `ls` trong terminal:

```txt
   shell (bash) ── fork() ──► bản sao shell ── exec("ls") ──► chạy "ls"
        │                                                         │
        └──────────────── chờ con xong (wait) ◄───────────────────┘
        ▼
   shell sẵn sàng nhận lệnh kế
```
Khi đó: 

- `fork()`: tạo bản sao y hệt tiến trình cha (cùng code, cùng dữ liệu), sinh ra tiến trình con với PID mới

- `exec()`: nạp đè chương trình mới lên tiến trình mới, thay code và dữ liệu cũ. 

## 5.2. Trên Windows: CreateProcess 
- Windows dùng một lời gọi duy nhất CreateProcess(): vừa tạo tiến trình mới vừa nạp chương trình trong một bước. 

- Khác Unix nhưng kết quả tương tự: một tiến trình mới ra đời với PID riêng.

## 5.3. Cây tiến trình và init (PID1) 
- Mỗi tiến trình đều có cha, toàn bộ tiến trình trên máy tạo thành một cây tiến trình (process tree). Với gốc đầu tiên kernel tạo lúc khởi động - init (hoặc `systemd`), luôn mang PID1

```txt
   init (PID 1)
   ├── sshd
   │   └── bash (PID 1500)
   │       └── python (PID 1733)
   └── gnome-shell
       └── chrome (PID 2048)
           ├── chrome (tab 1)
           └── chrome (tab 2)

```

## 6. Tổng hợp và lưu ý
- *Lý do: mỗi tab Chrome là một process riêng?* 

    - Cô lập lỗi: Nếu một trang web nào đó chết, sập hoặc lỗi thì chỉ có process tab đó ảnh hưởng, các tab khác vẫn sống. Vì, mỗi tab có process được phân vùng nhớ riêng. 

    - Bảo mật: Khi một trang web độc hại được nhốt trong một process riêng và có quyền hạn chế (sandbox), sẽ khó "nhòm ngó" dữ liệu các tab khác. Vì vậy, cô lập bộ nhớ của các process khác nhau là bức tường bảo mật. 

    - Đa nhân CPU: Nhiều process chạy song song trên CPU, giúp trình duyệt mượt hơn. 

- Các bước hệ điều hành biến một program thành process: Đọc file chương trình từ đĩa → Nạp lệnh vào RAM → Cấp phát tài nguyên (bộ nhớ, thanh ghi) → CPU thực thi lệnh đầu tiên.