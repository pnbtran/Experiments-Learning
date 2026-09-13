# 006 TRẠNG THÁI TIẾN TRÌNH VÀ PCB
## 1. Tiến trình hoạt động như nào? 
- Process có chạy liên tục trên CPU đến khi chương trình kết thúc phiên làm việc? Dĩ nhiên là không. 

- CPU chỉ có vài lõi nhưng máy thì có hàng tá process cần chạy nên mỗi process sẽ chạy trong 1 lát thời gian ngắn, sau đó nhường CPU cho process khác. 

- Vì vậy, process luôn phải đổi giữa các trạng thái. 

## 2. Các trạng thái process
- Một process có 5 trạng thái: 

| Trạng thái | Ý nghĩa | Ví dụ (diễn giải) | 
|------------|---------|-------------------|
| New (mới) | vừa được tạo, OS đang chuẩn bị (cấp PCB, vùng nhớ), chưa sẵn sàng chạy | Vừa được nhận, đang làm thủ tục | 
| Ready (sẵn sàng) | đã sẵn sàng hoạt động, chờ được cấp CPU | ngồi vào bàn, chờ sếp giao task |
| Running (đang thực thi) | đang thực thi lệnh trên CPU ngay lúc này | đang ngồi chạy dl của task | 
| Waiting/Blocked (chờ) | đang chờ một sự kiện (I/O xong, dữ liệu tới) KHÔNG dùng CPU | gửi sơ bộ kết quả, chờ phản hồi | 
| Terminated (kết thúc) | đã chạy xong (hoặc kill), OS thu hồi tài nguyên | task xong, tan làm |

- Lưu ý: 
    - Trên một lõi CPU, tại một thời điểm chỉ có đúng 1 process running, số còn lại sẵn sàng thì đang ready.

    - Ready khác Running, vì Ready đủ điều kiện chạy nhưng thiều quyền dùng CPU từ OS

    - Waiting thì không cần CPU, dù CPU cũng đang rảnh

    - **Tóm lại**, Ready = muốn CPU nhưng chưa tới lượt, Waiting = đừng đưa CPU, đang chờ thứ khác. Cả hai điều không chạy nhưng đang có nhiệm vụ khác nhau 

## 3. Sơ đồ trạng thái 
- Các trạng thái được chuyển cần được nối bằng các đường truyền có quy luật 

```txt
        admit            dispatch (được chọn)
  NEW --------> READY -------------------> RUNNING ------> TERMINATED
                  |                           |           (exit: chạy xong)
                  |       timeout/preempt/    |
                  |     <---------------------/
                  |   (hết lượt, bị giành CPU)
                  |                           |
                  |                           |     I/O hoặc chờ sự kiện
       I/O xong /  \                          v
       sự kiện tới  \---------------------- WAITING
        (wake up)        (Blocked)
``` 

- Diễn giải: 
    - New -> Ready (admit): OS đã chuẩn bị xong, đưa tiến trình vào hàng chờ CPU

    - Ready -> Running (dispatch): process được chọn và trao quyền CPU

    - Running -> Ready (timeout/ preempt): hết lượt (time slice) hoặc process được ưu tiên hơn đã giành CPU

    - Running -> Waiting (block): tiến trình xin chừo I/O hoặc sự kiện (qua system call) 

    - Waiting -> Ready (wake up): sự kiện đã tới (i/O xong, dữ liệu về), phần cứng báo ngắt 

    - Running -> Terminated (exit): tiến trình chạy xong hoặc bị kill

    - **Nhầm lẫn dễ gặp**: Không có mũi tên đi thẳng từ Waiting sang Running, vì khi process chạy xong sẽ quay về Ready chờ được lập lịch chứ không chạy lại ngay. 

## 4. PCB là gì? 
- PCB (Process Control Block) là cấu trúc dữ liệu OS tạo và lưu cho mỗi tiến trình. PCB như hồ sơ nhân viên đầy đủ: ghi mọi thứ cần thiết cho OS quản lí tiến trình và thể có thể "đóng băng" và "rã băng" mà không làm mất dấu. 

| Mục trong PCB | Lưu gì? | Để làm gì? | 
|---------------|---------|------------|
| PID | Mã định danh tiến trình (số duy nhất) | Phân biệt các tiến trình |
| Trạng thái | New/ REady/ Running/ Waiting/ Terminated | OS tiến trình đang ở đâu |
| Program Counter (PC) | địa chỉ lệnh kế tiếp sẽ thực thi | để chạy đúng chỗ dang dỡ | 
| Thanh ghi CPU (registers) | giá trị các thanh ghi lúc bị dừng | Khôi phục y nguyên "ngữ cảnh tính toán" |
| Thông tin bộ nhớ | Con trở tới page table, giới hạn vùng nhớ  OS biết tiến trình dùng vùng RAM nào? | 
| Danh sách file đang mở | Bảng các file/ đường ống đang dùng | quản lí ra vào tiến trình | 
| Thông tin lập lịch | Độ ưu tiên (priority), thời gian đã chạy... | độ lập lịch quyết định tiến trình nào được chạy tiếp | 
| Accounting | thời gian CPU đã dùng, giới hạn tài nguyên, ID người dùng | Thống kê, tính tài nguyên, phân quyền | 

> PCB như quyển sổ giao ban. Khi công nhân trực đổi ca, sẽ ghi lại tất cả trạng thái máy móc, thiết bị, công việc để người sau có thể tiếp tục thực hiện và tiếp nối công việc chính xác. 
> 
> PCB cho phép OS bàn giao CPU giữa các process mà không bị nhầm lẫn. 

## 5. Context switch - chuyển ngữ cảnh
- Khi process A và B cần chuyển cho nhau thì cần context switch để chuyển đổi, có thể tạm hình dung:

```txt
   Đang chạy tiến trình A
            │
            v
   ┌─────────────────────────────────┐
   │ 1. LƯU ngữ cảnh của A           │       PC, thanh ghi... -> ghi vào PCB của A
   ├─────────────────────────────────┤
   │ 2. Cập nhật trạng thái A        │      Running -> Ready (hoặc Waiting)
   ├─────────────────────────────────┤
   │ 3. Chọn tiến trình B (lập lịch) │
   ├─────────────────────────────────┤
   │ 4. NẠP ngữ cảnh của B           │      Đọc PCB của B -> nạp lại PC, thanh ghi
   ├─────────────────────────────────┤
   │ 5. B chuyển Ready -> Running    │
   └─────────────────────────────────┘
            │
            v
   CPU bắt đầu chạy tiến trình B (đúng chỗ B đang dở)
```

- Quá trình lưu PCB của process đang chạy rồi nạp PCB của process kế = context switch

- Context switch là overhead thuần - trong lúc CPU bận lưu/nạp thanh ghi và đổi bảng nhớ, nó không làm việc hữu ích nào cho chương trình. Giống đang làm task A mà sếp bảo chuyển sang task B: phải cất tài liệu A, lấy tài liệu B, lấy lại mạch suy nghĩ… mất vài phút “không sản xuất” gì. 

> ⚠️ Một context switch chỉ cỡ vài microsecond, nghe nhỏ. 
>
> Nhưng nếu hệ thống chuyển hàng chục nghìn lần mỗi giây, tổng chi phí “vô ích” này tích lại rất đáng kể. 

## 6. Hàng đợi: Ready queue và waiting queue
- Chúng ta có hàng trăm process nhưng chỉ vài lõi CPU, OS quản lí bằng hàng đợi (queue) chứa con trỏ đến PCB

| Hàng đợi | Chứa ai | Khi nào rời hàng |
|----------|---------|------------------|
| Ready queue | Tiến trình ở trạng thái Ready (chờ CPU) | Được lập lịch chọn -> Running |
| Waiting/Device queue | Tiến trình chờ một thiết bị/sự kiện cụ thể (vd chờ ổ đĩa) | Sự kiện xảy ra -> về Ready queue |

- Mỗi thiết bị I/O (đĩa, mạng, bàn phím…) thường có hàng đợi riêng. 

- Khi thiết bị hoàn thành (báo ngắt), OS lấy tiến trình tương ứng ra khỏi waiting queue và đẩy về ready queue.

> Vòng đời điển hình: tiến trình lượn vòng Ready -> Running -> Waiting -> Ready… rất nhiều lần trước khi tới Terminated. Nó hiếm khi chạy “một mạch”

## 7. Tóm lại và lưu ý
- Tóm tắt: 

```txt
   NEW ──admit──> READY ──dispatch──> RUNNING ──exit──> TERMINATED
                   ^  ^                  │  │
       wake up ────┘  └──timeout/preempt─┘  │ I/O / chờ sự kiện
       (I/O xong)                           v
                              WAITING (Blocked)

   Mỗi tiến trình <-> 1 PCB (PID, trạng thái, PC, thanh ghi, bộ nhớ, file, ưu tiên)
   Đổi tiến trình = CONTEXT SWITCH (lưu PCB cũ, nạp PCB mới) = overhead
```
