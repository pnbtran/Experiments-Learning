# 007 LUỒNG (THREAD) VÀ ĐA LUỒNG
## 1. Luồng (Thread) là gì? 
- Luồng (Thread) là đơn vị thực thi nhỏ nhất mà thể điều hành có thể lập lịch chạy trên CPU. 

- Mỗi process có ít nhất một thread (thread chính và main thread), có thể có nhiều thread để làm nhiều việc cùng một lúc. 
    - Process: chương trình đang chạy + tài nguyên riêng (bộ nhớ + file)

    - Thread: một mạch được thực hiện bên trong một tiến trình

    - Tiến trình đơn luồng - chỉ làm một việc tại một thời điểm, tiến trình đa luồng - có nhiều mạch chạy song song, chia nhau làm nhiều việc 

## 2. So sánh Process và Thread 
- Thread chia sẻ:
    - Tài sản chung của process: code, dữ liệu, heap, file đang mở

    - Tài sản riêng: stack, thanh ghi, program counter

| Thành phần | -- | Giải thích | 
|------------|----|------------|
| Code (mã lệnh) | chung | Mọi thread chạy cùng 1 chương trình | 
| Data (biến toàn cục) | chung | cùng nhìn thấy và sửa được, nguồn gốc của race condition | 
| Heap (bộ nhớ cấp phát động) | chung | vùng nhớ dùng chung, ví dụ đối tượng tạo bằng new/malloc | 
| File đang mở, thiết bị I/O| chung | cùng tiến trình thì cùng bảng file | 
| Stack (ngăn xếp) | riêng | mỗi thread có chuỗi lời gọi hàm + biến cục bộ riêng | 
| Thanh ghi (rehisters) | riêng | trạng thái CPU của riêng mỗi thread | 
| Program counters (PC) | riêng | mỗi thread đang chạy mỗi lệnh khác nhau | 

- **Thread và proprocess, nào tốn tài nguyên hơn?** 
    - ProProcess mới thì OS cần cấp không gian địa chỉ mới, đựng PCB đầy đủ nên tốn hơn

    - Tạo Thread mới trong process đã có thì chỉ cần thâm stack, thanh ghi nhỏ và dùng tài nguyên chung. 

    - Nên, tạo/ hủy thread và chuyển ngữ cảnh (context switch) giữa 2 thread trong 1 process sẽ tiết kiệm hơn

## 3. Sơ đồ liên hệ Process và Thread 
- Một process có thể chứa nhiều thread

```txt
   PROCESS ĐƠN LUỒNG               PROCESS ĐA LUỒNG (3 thread)
  ┌───────────────────┐           ┌───────────────────────────────────┐
  │  Code  Data Heap  │           │  Code   Data   Heap   File (CHUNG)│
  │  File (dùng chung)│           ├──────────┬──────────┬─────────────┤
  ├───────────────────┤           │ Thread 1 │ Thread 2 │  Thread 3   │
  │   Thread 1        │           │  Stack   │  Stack   │   Stack     │
  │   Stack           │           │  Regs    │  Regs    │   Regs      │
  │   Regs + PC       │           │  PC      │  PC      │   PC        │
  └───────────────────┘           └──────────┴──────────┴─────────────┘
       1 mạch chạy                3 mạch chạy SONG SONG, chung tài nguyên

```

- Nếu đa luồng, phần tài sản chung có 1 bản cho cả process còn phàn trạng thái thực thi sẽ nhân theo số thread. 

## 4. Lợi ích đa luồng
- Có 4 lợi ích chính 
    - Song song thật trên CPU đa nhân: mỗi Thread chạy trên một nhân (core) khác --> làm việc đồng thời. VD: Render video chia cho 8 core cùng làm

    - Giữ giao diện phản hồi: một thread lo UI, thread khác làm việc nặng --> app không đơ. VD: UI vẫn thực hiện trong khi thread mạng đang chạy

    - Chia sẻ tài nguyên nhẹ: thread chung bộ nhớ nên trao đổi dữ liệu trực tiếp. VD: thread tải về và thread hiển thị dùng chung dữ liệu, đỡ chia sẻ

    - Tiết kiệm hơn process: tạo/chuyển thread tiết kiệm hơn tạo/chuyển process. VD: Web server sẽ tạo mỗi thread cho mỗi yêu cầu xử lí

> Trên CUP chỉ có 1 nhân: khi 1 thread chờ I/O (đọc đĩa, chờ mạng), hệ điều hành cho thread khác chạy thay vì để CPU ngồi không

## 5. Rủi ro của đa luồng - race condition
- Do các Thread chung Process có phần tài nguyên chung, nếu không phối hợp sẽ gây ra *lỗi tranh chấp tài nguyên* (race condition). 

- Ví dụ, trong sổ thu ngân Thread A và B đang có 100, cùng cộng 50 cho mỗi Thread, cùng ghi đè nên kết quả 150 thay cho 200. "Một lần cộng 50 đã bốc hơi" 

```txt
  Biến đếm = 100 (chung)
  Thread A: đọc 100 ─┐                 ┌─ ghi 150
  Thread B: đọc 100 ─┴─ (cả hai +50) ──┴─ ghi 150   ← MẤT 1 lần cộng!
  Kết quả SAI: 150 (đúng phải là 200)
```
> Cái giá của việc chia sẻ bộ nhớ không cẩn thận. Để xử lí, cần đồng bộ (synchronization) để khóa (lock/mutex), semaphore... đảm bảo mỗi lần chỉ có một thread được sửa dữ liệu chung. 

## 6. Mô hình Thread
- Threaad quản lí theo mô hình 2 cấp
    - **User-level thread** (mức người dùng): thư viện không gian người dùng quản lí, kernel không biết từng thread. Tạo/chuyển rất nhanh nhưng nếu có 1 Thread bị chặn bởi system call, cả tiến trình có thể bị chặn theo, khó tận dụng nhiều nhân trong CPU. 

    - **Kernel-level thread** (mức nhân): do chính OS quản lí lập lịch. Tuy nặng nhưng kernel lập lịch được từng thread trên nhiều nhân. Một Thread chờ I/O thì không làm kẹt Thread khác. 

- Mô hình 3 ánh xạ User-level thread và Kernel-level thread 

| Mô hình | Ý nghĩa | Ưu và nhược điểm | 
|---------|---------|------------------|
| Many-to-one | Nhiều user thread -> 1 kernel thread | Nhanh nhưng 1 thread bị block là kẹt cả, không chạy đa nhân | 
| One-to-one | Mỗi user thread -> 1 kernel thread | song song thật, 1 thread block không làm kẹt các thread khác (tốn tài nguyên). (Window và Linux hiện đại dùng kiểu này) | 
| Many-to-many | Nhiều user thread -> nhiều kernel thread (<=) | linh hoạt, cần bằng nhưng cài đặt phức tạp |
 
> Đa số hệ điều hành phổ biến dùng mô hình one-to-one nên mỗi thread thường tương ứng với một thread được kernel lập lịch 

## 7. Ví dụ 
- Trình duyệt web (chrome, firefox)
    - Thread render: vẽ trang, bố cục, ảnh

    - Thread tải mạng: download HTML, CSS, ảnh

    - Thread giao diện UI: cuộn, click, gõ địa chỉ

    - Nhờ tách thread, trang vẫn cuộn muột trong khi ảnh đang tải nền

- Trình soạn thảo văn bản (Word hay VS Code)
    - Thread gõ: hiển thị kí tự theo thời gian thực

    - Thread tự lưu (Auto-save): ghi file nền

    - Thread kiểm tra chính tả: gạch chân lỗi mà không chặn việc nhập gõ

- **Điểm chung**, luôn có một thread tương tác người dùng để app sống, không đơ; việc nặng thì đẩy sang thread khác xử lí. 

## 8. Vấn đề
- Nghịch lí đẹp của Thread: rẻ vì chia sẻ chung không gian địa chỉ nhưng cũng nguy hiểm vì chia sẻ vì dễ sinh ra bug khó tái hiện - race condition. 

- Fact: không phải cứ đa luồng là nhanh. Vì mỗi lần đồng bộ (khóa/ mở khóa) đều cần thời gian (overhead đồng bộ), nếu tranh nhau cùng một khóa , các thread phải xếp hàng chờ, quy trình lại thành tuần tự, OS phải context switch liên tục 

> Đa luồng giúp ích nhất khi công việc chia nhỏ độc lập được hoặc có thể có nhiều thời gian để chờ I/O