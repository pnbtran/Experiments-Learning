# 003 KIẾN TRÚC BÊN TRONG - KERNEL, SHELL, SYSTEM CALL

## 1. Kernel - khối điều khiển trung tâm
### 1.1. Kernel là gì? 
- **Kernel** (nhân) là phần lõi của hệ điều hành, luôn nằm trong bộ nhớ, là phần code **duy nhất** có toàn quyền điều khiển phần cứng. 

- Còn lại là **shell**, đều phải đi qua kernel để chạm tới CPU, RAM hay thiết bị. 

### 1.2. Nhiệm vụ
- Kernel có 4 nhiệm vụ cốt lõi
    - Quản lí CPU (lịch nhiệm): Quyết định tiến trình nào chạy trên CPU, khi nào, bao lâu

    - Quản lí bộ nhớ: cấp phát/ thu hồi RAM, cô lập cùng nhớ từng tiến trình

    - Quản lí thiết bị (I/O): điều khiển ổ cứng, bàn phím, mạng qua driver

    - Quản lí tiến trình: tạo, dừng, hủy tiến trình, chuyển ngữ cảnh giữa các tiến trình 

### 1.3. Kiến trúc Kernel
- Vậy Kernel gồm những phần nào? Kiến trúc của kernel được xây dựng dựa trên việc, trả lời câu hỏi: có bao nhiêu thứ nên chạy trong khu đặc quyền (kernel mode)? 

- Có 3 dạng: 
| Loại | Ý tưởng | VÍ dụ | Ưu điểm | Nhược điểm | 
|------|---------|-------|---------|------------|
| Monolithic | Gần như mọi dịch vụ (driver, file system, mạng) đều chạy CHUNG trong kernel mode | Linux | Nhanh (gọi nội bộ, ít chuyển chế độ) | Mỗi driver lỗi có thể làm sập cả kernel, kernel lớn | 
| Microkernel | Kernel chỉ giữ tối thiểu (lập lịch, bộ nhớ, giao tiếp), driver, file system chạy ở user mode | MINUX, QNX, seL 4 | Cô lập tốt, một dịch vụ chết, không gây sập hệ thống | Chậm hơn nhiều, các tiến trình qua lại (IPC) | 
| Hybrid (lai) | Lỗi giống microkernel nhưng gộp một số dịch vụ hiệu năng cao và kernel mode | Window NT, macOS (XNU) | Cân bằng tốc độ và modul hóa | Phức tạp, ranh giới mời giữa hai trường phái | 

> Có thể hình dung: monolithic như 1 bếp trưởng làm tất cả không cần phụ bếp (nhanh nhưng nếu sai thì có nguy cơ hỏng bếp cao), microkernel = nhiều đầu bếp nhưng ở mỗi khu bếp riêng tác biệt cần trao đổi qua bộ đàm (an toàn nhưng tốn công liên lạc), hybird = đa số tách phòng nhưng dụng cụ có thể để chung để thao tác lẹ.

## 2. Hai chế độ làm việc của CPU
- CPU có hai chế độ làm việc gồm: user mode và kernel mode nhầm tránh việc mỗi chương trình đều chạm thẳng đến phần cứng, mỗi app lỗi hoặc virus không thể xóa sạch ổ cứng hay đọc trộm RAM app khác. 

| Chế độ | khu vực làm việc | Quyền hạn | Tên gọi | 
|--------|------------------|-----------|---------|
| user mode | ứng dụng (chorme, game...) | Hạn chế: Không được dùng lệnh đặc quyền, không chạm thẳng phần cứng | Ring 3 |
| Kernel mode | Kernel của hệ điều hành | Toàn quyền: mọi lệnh, mọi vùng nhớ, mọi thiết bị | Ring 0| 

- User mode không phải nhà tù tuyệt đối - app vẫn cần lưu file, vẽ màn hình, gửi mạng. Nhưng nó chỉ chuyển sang kernel mode qua một cánh cửa duy nhất, có kiểm soát: đó chính là system call.  

## 3. System call
### 3.1. Khái niệm 
- **System call (lời gọi hệ thống)** là cơ chế để chương trình ở user mode yêu cầu kernel thực hiện một dịch vụ cần đặc quyền (đọc/ghi file, tạo tiến trình, gửi mạng…) - cánh cửa chính thức từ ring 3 vào ring 0. 
### 3.2. Các lệnh quan trọng (POSIX/Linux)
- `open`: mở (tạo) một file, trả về file descriptor
- `read`: độc dữ liệu từ file, thiết bị vào bộ nhớ
- `write`: ghi dữ liệu từ bộ nhớ ra file/ thiết bị
- `fork`: tạo một tiến trình con (bản sao của tiến trình hiện tại)
- `exec`: nạp một chương trình mới để dè tiếng tình hiện tại
- `exit`: kết thúc tiến trình, trả mã thoát cho OS

> Bộ ba fork + exec + exit là cách shell chạy chương trình: shell fork ra con, con exec lệnh bạn gõ, xong thì exit.

### 3.3. Cơ chế chuyển từ user mode sang kernel mode
- System call không phải lời mời gọi hàm thường. Nó dùng lệnh CPU đặc biệt, để phát **trap** mộy ngắt mền có chủ đích khiến CPU chuyển từ ring 3 sang ring 0 và nhảy vào điểm cố định của kernel 

```txt
[ring 3] app: write(...) ── lệnh `syscall` ──TRAP──► [ring 0] kernel:
   ▲                          (ring 3 → ring 0)        kiểm tra quyền,
   └──── nhận kết quả ◄──── trở về ring 3 ─────────────  ghi xuống đĩa
```

- Khi thực hiện system call: 
    (1) chuyển từ ring 3 sang ring 0

    (2) lưu ngữ cảnh (thanh ghi, ngăn xếp)

    (3) kernel kiểm tra hợp lệ và thực thi 

    (4) chuyển ngược về ring 3 

    (5) khôi phục ngữ cảnh, tốn nhiều chu kì CPU hơn lệnh call thường 

> System call như tờ phiếu yêu cầu được vào khu đặc biệt (kernel) 

## 4. Shell 
- Shell (vỏ) là chương trình nhận lệnh từ bạn rồi dịch thành system call để kernel thực thi. 

- Shell không phải một phần kernel, nó chạy ở user mode như mọi app khác. 

- Có 2 kiểu shell: 
    - **CLI (dòng lệnh)**: gõ lệnh bằng văn bản. Ví dụ: bash, zsh (Linux/macOS), Powershell, cmd (Windows)

    - **GUI (đồ họa)**: bấm chuột, kéo thả icon. ví dụ: Windows Explorer, GNOME/KDE (linux), Finder (macOS)

## 5. Tổng kết 
- `app --> thư viện --> system call --> kernel --> phần cứng`. Thông thường, ứng dụng ít khi tự gõ `system call` mà phải đi qua thư viện chuẩn, bọc lệnh system call lại dễ sử dụng và đa nền tảng (?)

```txt
  ┌─────────────────────────────────────────────┐  USER MODE (ring 3)
  │  1. ỨNG DỤNG    printf("hi") / fopen(...)   │
  │           │                                 │
  │  2. THƯ VIỆN    libc / Win32 API            │
  │     (bọc system call cho tiện)              │
  │           │  phát lệnh `syscall` (TRAP)     │
  └───────────┼─────────────────────────────────┘
              ▼   ───────── ranh giới đặc quyền ─────────
  ┌─────────────────────────────────────────────┐  KERNEL MODE (ring 0)
  │  3. KERNEL    nhận, kiểm tra quyền, thực thi│
  │           │  ra lệnh cho driver             │
  │  4. PHẦN CỨNG  CPU / RAM / ổ cứng / mạng    │
  └─────────────────────────────────────────────┘

``` 

>    Mỗi lần băng qua ranh giới user/kernel đều toán chi phí chuyển chế độ (model switch): CPU phải lưu/ khôi phục thanh ghi, đổi quyền, có thể làm "nguội" cache và đường ống lệnh (pipeline). Một system call lẻ thì nhỏ, nhưng hàng triệu thì làm chậm thấy rõ. Để tối ưu: gom nhiều thao tác để giảm số system call. 
> 
>   Thay vì `write` 1000 lần - mỗi lần 1 byte (1000 lần trap), ta gom vào bộ nhớ đệm (buffer) rồi gọi `write` một lần (1 trap). Đây là lí do vì sao thư viện I/O có buffering `printf` không ghi thẳng từng kí tự mà gom lại rồi mới đẩy xuống kernel.  