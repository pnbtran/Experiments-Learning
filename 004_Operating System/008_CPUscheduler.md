# 008 LẬP LỊCH CPU - FCFS, SJF, ROUND ROBIN, PRIORITY
## 1. Lí do CPU cần lập lịch? 
- Lúc nào máy cũng có hàng trăm process, nhưng lõi CPU thì chỉ có thể chạy đúng 1 process tại 1 thời điểm.  Số còn lại xếp hàng trong hàng đợi sẵn sàng (ready queue), chờ tới lượt. 

- Bộ lập lịch ngắn hạn (short-term scheduler / CPU scheduler) thuộc kernel: mỗi khi CPU rảnh, nó chọn một process từ ready queue để trao CPU. 

- Việc trao CPU do bộ điều phối (dispatcher) thực hiện, gồm thao tác chuyển ngữ cảnh (context switch)

```txt
   Ready Queue (hàng chờ)            CPU (1 chỗ)
   [ P4 | P3 | P2 | P1 ] ──scheduler chọn?──▶ [ đang chạy ]
```
> Scheduler = cái talon (số thứ tự), CPU = dekanat đang xử lí tá hs của đám sinh viên chờ (ready queue). 

## 2. Tiêu chí đánh giá thuật toán lập lịch 
- 5 thước đo

| Tiêu chí | Ý nghĩa | Muốn |
|----------|---------|------|
| CPU utilization (mức tận dụng CPU) | % thời gian CPU bận làm việc | Càng **cao** càng tốt |
| Throughput (thông lượng) |Số process hoàn thành trong 1 đơn vị thời gian | Càng **cao** càng tốt |
| Turnaround time (thời gian hoàn thành) | Tổng thời gian từ lúc process đến đến lúc nó xong hẳn | Càng **thấp** càng tốt |
| Waiting time (thời gian chờ) | Tổng thời gian process nằm chờ trong ready queue (không tính lúc chạy) | Càng **thấp** càng tốt |
| Response time (thời gian phản hồi) | Từ lúc gửi yêu cầu đến lúc có phản hồi ĐẦU TIÊN (chưa cần xong) | Càng **thấp** càng tốt |

- 2 công thức cần nhớ: 
```txt 
Turnaround = Completion (hoàn thành) - Arrival (đến)
Waiting    = Turnaround - Burst time (thời gian cần chạy)
```

> Đừng nhầm response và turnaround.
>   Khi mở máy, response time là lúc màn hình bắt đầu thực hiện
>   turnaround time là lúc app load xõng hẳn 
>   -> hệ tương tác cần response thấp, batch cần turnaround thấp

## 3. Preemptive và Non-preemptive
- có 2 trường phái 

    - **Non-preemptive** (không tước quyền): process có CPU thì thực hiện đến khi xong hoặc tự động nhường (chờ I/O). 

    - **Preemptive** (tước quyền): kernel có thể "giật" CPU lại giữa chừng (hết quantum, hoặc có process được ưu tiên cao hơn) 

- Lí do preemptive quan trọng? Nếu 1 process lập vô hạn `while(true)` mà còn non-preemptive thì treo máy. Khi đó, preemptive ch phép kernel ngắt đồng hồ (timer interrupt) để giành CPU lại, nền tảng để máy chạy đa nhiệm. 

## 4. FCFS - First Come First Serve (đến trước được phục vụ trước)
- Hiểu đơn giản, process nào ready queue trước thì running trước, non-preemptive, running xong, về xếp hàng tiếp. 

- Sơ đồ Gantt 3 process và burst time
```txt 
|      P1       |  P2  |  P3  |
0               24     27     30
```
    Waiting time trung bình = (0 + 24 + 27) / 3 = 17

    Turnaround trung bình = (24 + 27 + 30) / 3 = 27

>   P2 và P3 chỉ có 3 đơn vị nhưng chờ đến 24 và 27, vì: đang xếp sau P1 --> hiệu ứug đoàn xe (convoy effect) 

## 5. SJF - Shortest Job First (việc ngắn nhất trước)
- Ý tưởng: cho process có CPU burst time ngắn chạy trước 

- Vẫn 3 process phía trên, nhưng khi này sơ đồ sẽ là: 
```txt 
|  P2  |  P3  |      P1       |
0      3      6               30
```

    Waiting time trung bình = (0 + 3 + 6) / 3 = 3 (so với FCFS là 17 - giảm cực mạnh!)

    Turnaround trung bình = (3 + 6 + 30) / 3 = 13

- 2 nhược điểm của SJF: 
    - [1] Starvation: nếu liên tục có process ngắn đến, các process dài như P1 sẽ được chờ mãi mãi. 

    - [2] Khó biết trước burst time: OS thực tế không biết process nào sắp chạy bao lâu, phải đoán từ lịch sử (trung bình mũ - exponential averaging) --> SJF "thuần" hiếm khi được dùng nhưng ý tưởng thì được ứng dụng khá nhiều 

## 6. Round Robin (RR) - xoay vòng công bằng 
- Mỗi process được chạy trong 1 khoảng thời gian nhất định - time quantum (lượng tử thời gian), thường 10-100 mili-giây

- Hết lượt, chưa xong --> CPU bị giật lại, nhường process khác --> xuống xếp hàng chờ tiếp 

- vẫn 3 process nhưng bây giờ time quantum = 4, thì
```txt
| P1 | P2 | P3 | P1 | P1 | P1 | P1 | P1 |
0    4    7   10   14   18   22   26   30
```

    Waiting time trung bình = (6 + 4 + 7) / 3 ≈ 5.67

    Turnaround trung bình = (30 + 7 + 10) / 3 ≈ 15.67

> Khi đóđó:
>    P1 chạy 4 xong còn 20 - xuống xếp hàng chờ

>   P2 chạy 3, xong tại 7

>   P3 chạy 3, xong tại 10

>   P1 chạy từng đợt, đến 30

- Vậy, RR không cho turnaround tốt nhất, SJF vẫn thắng, nhưng ở đây nó công bằng (không process nào phải chờ quá lâu để được đụng CPU lần đầu) nên response vẫn ổn. 

>> RR = FCFS + time quantum + preemptive. Nếu time quantum vô hạn thì RR biến thành FCFS. 

## 7. Priority Scheduling (lập lịch theo chế độ ưu tiên)
- Mỗi process sẽ có 1 số ưu tiên (priority) 

- CPU sẽ ưu tiên process cao nhất (quy ước phổ biến: số nhỏ = ưu tiên cao)

- FCFS và SJF là trường hợp đặc biệt của priority 

- Cho 4 process, cùng t=0, non-preemptive

| process | burst | priority |
|---------|-------|----------|
| P1 | 4 | 2 |
| P2 | 3 | 1 (cao nhất) | 
| P3 | 2 | 4 (thấp nhất) |
| P4 | 1 | 3 |

- Khi đó thứ tự ưu tiên: P2 -> P1 -> P4 -> P3
```txt
|  P2  |  P1  | P4 |  P3 |
0      3      7    8    10
```
    Wating time trung bình = (0 + 3 + 7 + 8)/4 = 4.5

- Nhược điểm: starvation, process ưu tiên thấp như P3 có thể bị process cao hơn chen ngang, giành quyền --> không bao giờ được chạy

- Cách khắc phục: aging (lão hóa), cứ sau khoảng chờ, tăng dần độ ưu tiên của process đang chờ

## 8. Time quantum và MLFQ
- Time quantum bao nhiêu là vừa? Bài toán RR
    - Quá nhỏ (vd 1ms): quá nhiều context sưicth (lưu/khôi phục thanh ghi, tốn chi phí). CPU bận dọn khu làm việc > làm việc --> overhead lớn

    - Quá lớn (vd 10 000ms): process chạy gần hết mới nhường --> RR chuyển hóa thành FCFS, chả công bằng

    - Vừa phải (10 - 100ms): cân bằng, đảm bảo phản hồi nhanh mà overhead vẫn nhỏ. 

    --> quantum đủ lớn để phần lớn CPU burst xong trong 1 quantum, nhưng đủ nhỏ để hệ phản hồi mượt

- **MLFQ (Multilevel Feedback Queue - hàng đợi phản hồi nhiều mức)**: cách các OS thật dung hòa mọi thứ. 
    - Có nhiều hàng đợi mức ưu tiên khác nhau, mỗi hàng dùng RR. 
    
    - Process mới vào hàng ưu tiên cao; nếu dùng hết quantum (tỏ ra “ngốn CPU”) thì bị hạ xuống hàng thấp, nếu nhường CPU sớm (việc ngắn, tương tác) thì được giữ ở hàng cao; định kỳ đẩy tất cả lên (một dạng aging) để chống starvation.

- MLFQ tự “đoán” việc nào ngắn/tương tác để ưu ái mà không cần biết trước burst time - khắc phục đúng nhược điểm của SJF  

## 9. Tóm tắt
### 9.1. Lưu ý
- [x] hiểu lầm: FCFS luôn công bằng nên thuật toán rất oke 

  [v] Đúng: FCFS đơn giản, không starvation, nhưng wating time thường đợi, không hợp hệ tương tác. Vì FCFS công bằng theo thứ tự đến nên dễ convoy effect và đẩy watting time lên tb cao. 

- [x] hiểu lầm: SJF đã đủ tối ưu, cứ SJF thôi là ngon

  [v] Đúng: SJF tối ưu waiting time trên lí thuyết, thực tế chỉ dùng dạng dự đoán. SJF cần biết trước burst time, OS phải đoán, nguy cơ starvation cho process dài. 

- SJF (job dài bị job ngắn chen mãi) và Priority (job ưu tiên thấp bị job ưu tiên cao chen mãi) đều có thể starvation. FCFS và Round Robin thì không, vì mọi tiến trình chắc chắn tới lượt.