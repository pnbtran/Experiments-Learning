# 009 ĐỒNG BỘ TIẾN TRÌNH 
## 1. Race condition 
- Ví dụ: 
```txt
  balance ban đầu = 100
  A: ĐỌC balance = 100          --- bị ngắt, nhường CPU cho B ---
  B: ĐỌC 100 → TÍNH 150 → GHI balance = 150
  A: chạy tiếp, vẫn xài 100 đã đọc → TÍNH 150 → GHI balance = 150
  Kết quả: 150  (đáng lẽ phải là 200!)
```

- Một lần cộng đã mất (lost update) --> Race condition (mâu thuẫn dữ liệu)

- Vì sao race condition đáng sợ? nó không phải lúc nào cũng xảy ra. 
>Chạy 1000 lần có thể đúng 999 lần, sai 1 lần - bug “thỉnh thoảng mới hiện” cực kỳ khó debug, vì khi bạn thêm lệnh in để soi thì timing thay đổi và lỗi biến mất.  

## 2. Critical section và Mutual Exclusion 
- Đoạn code đụng tới tài nguyên chung (đọc, ghi, tăng `balance`) gọi là critical section. 

- Quy tắc: mỗi thời điểm chỉ một thread được phép chạm vùng tới hạn --> mutual exclusion (loại trừ lẫn nhau)

- Một giải pháp đồng bộ tốt, cần 3 yếu tố: 
    - Mutual exclusion - không 2 luồng cùng trong vùng giới hạn 

    - Progress - không cản vô cớ thread muốn vào khi vùng đang trống

    - Bounded waiting - không có thread nào chờ vô hạn, tránh đói/starvation 

    --> vùng tới hạn nên càng ngắn. Khóa cả đoạn code khiến các thread khác chờ lâu -> chậm cả chương trình. Chỉ khóa phần đụng dữ liệu chung. 

## 3. Công cụ đồng bộ 
- Hai công cụ phổ biến: Mytex và Semaphone 
### 3.1. Mutex/ Lock (khóa)
- Mutex (viết tắt MUTual EXclusion) là ổ khóa đơn giản: chỉ có 2 trạng thái khóa/mở --> chí có 1 thread được giữ được khóa

- Thread muốn vào Critical section thì cần `lock()`, xong cần `unlock()`, nhường cho Thread khác 

### 3.2. Semaphone (đèn tín hiệu có đếm)
- Semaphone là bộ đếm

- Semaphone cho phép tối đa N Thread cùng truy cập. Khi N = 1 thành binary semaphone, hành xử gần giống mutex. 

> Bãi giữ xe có N chỗ với bảng đếm số chỗ trống: 
>
>   mỗi xe vào bảng giảm 1 (thao tác wait/P), 
>
>   mỗi xe ra bảng tăng 1 (signal/V); 
>
>   khi bảng về 0, xe tới sau phải chờ 

### 3.3. So sánh

| Tiêu chí | Mutex/Lock | Semaphone | 
|----------|------------|-----------|
| Số Thread cùng lúc | đúng 1 | tối đa N (đếm) | 
| Bản chất | khóa sở hữu (ownership) | bộ đếm tính hiệu | 
| Ai mở | thường Thread đã khóa | Luồng nào cũng có thể signal | 
| Dùng khi | Bảo vệ 1 vùng tới hạn (1 chỗ) | Giới hạn N tài nguyên/ báo hiệu giữa luồng | 
| Ví dụ | Khóa biến `balance` | Bãi xe N chỗ, hồ bơi kết nối DB | 

### 3.4. Bài toán Producer - Consumer 
- Một ứng dụng kinh điểm của semaphone: 
    - Một bên sản xuất dữ liệu bỏ vào hàng đợi (thread tải video về)

    - Một bên tiêu thụ lấy ra dùng (Thread phát video)

    --> Semaphone giúp người tiêu thụ không lấy hàng đợi rỗng và người sản xuất không nhét hàng khi hàng đợi đầy.

## 4. Deadlock - khi tất cả đều kẹt cứng
- Khóa giúp tránh race condition, nhưng khóa ẩu sinh ra deadlock (tắt nghẽn) = ác mộng mới  --> kẹt cứng vĩnh viễn (?) 

- Mỗi Thread đang giữa 1 Data và chờ Data mà Thread khác chung process đang giữ --> vòng chờ không lối thoát. 

### 4.1. Bốn điều kiện Coffman dẫn đến Deadlock 
- 4 điều kiện dẫn đến Deadlock: 
    - Mutual Exclusion (loại trừ nhau): tài nguyên không chia sẻ được, một thời điểm chỉ có 1 tiến trình giữ

    - Hold and Wait (giữ và chờ): tiến trình đang giữ ít nhất và chờ thêm tài nguyên khác

    - No preemption (không được tước đoạt): không thể giành giật tài nguyên, mà tiến trình cần tự nhả 

    - Cricular Wait (chờ vòng tròn): có 1 tiến trình P1 chờ P2, P2 chờ P3 và ..., Pn chờ P1 

## 4.2. Sơ đồ vòng tài nguyên 
```txt
   P1 ──giữ──► R1        P1 muốn xin thêm R2 (đang bị P2 giữ)
   P2 ──giữ──► R2        P2 muốn xin thêm R1 (đang bị P1 giữ)

   → Vòng chờ: P1 → R2 → P2 → R1 → P1  (Circular Wait!) → kẹt cứng
```

### 4.3. Dining Phiosophers (bữa ăn triết gia?) 
- 5 triết gia ngồi quanh bàn tròn

 - giữa mỗi cặp có 1 chiếc đũa (tổng 5)

 - Mỗi người cần 2 chiếc đũa để ăn 

 --> Nếu tất cả cùng cần đũa bên trái và chờ đũa tay phải --> cả 5 đều CHỜ MÃI --> ĐÓI mãi

 --> Deadlock xảy ra 

## 5. Bốn trường phái xử lí Deadlock 
- 04 trường phái để xử lí Deadlock 
    - [1] Prevention (ngăn ngừa) : thiết kế để phá bỏ ít nhất 1 trong 4 điều kiện Coffman ngày từ đầu ???

    - [2] Avoidance (né tránh): trước cấp tài nguyêm, kiểm tra có dẫn tới trạng thái nguy hiểm (Banker's algorithm)

    - [3] Dêtction & Recovery: cho deadlock xảy ra, định kì dò vòng chờ rồi xử lí (kill tiến trình và tước tài nguyên) ???

    - [4] Ignore ('thuật toán đà điều'?): coi như không có, reboot lỡ kẹt - đơn gian, gần như miễn phí 

> 💡 Thực tế bất ngờ: Linux và Windows phần lớn chọn “Ignore” (đà điểu) cho deadlock cấp ứng dụng - vì deadlock hiếm, chi phí phòng chống đầy đủ quá đắt. Lập trình viên phải tự cẩn thận với khóa của mình. 

## 6. Nhưng: chỉ cần phá 1 trong 4 điều kiện? 
- Để Deadlock cần 4 Coffman --> phá được 1 thì Deadlock không thể hình thành = Tư duy Prevention: 
    - Mutual Exclusion: cho tài nguyên dùng chung nếu được (vd: file read only)

    - Hold and Wait: bắt xin tất cả tài nguyên một lần, hoặc nhả hết trước khi xin thêm

    - No Preemption: Cho phép trước tài nguyên của tiến trình đang chờ và cấp lại sau 

    - Circular Wait: đánh số tài nguyên, bắt xin theo thứ tự tăng dần. 

    --> Circular wait dễ áp dụng nhất, quy định một thứ tự khóa cố định --> vòng chờ không xảy ra. 

- **Tóm tắt** 
```txt 
   Nhiều luồng + dữ liệu chung
       ├─ KHÔNG đồng bộ ──► RACE CONDITION (mất cập nhật, sai ngẫu nhiên)
       └─ Dùng khóa (mutex/semaphore)
              ├─ đúng cách ──► an toàn ✅
              └─ khóa sai thứ tự ──► DEADLOCK (cần đủ 4 đk Coffman;
                                      phá 1 điều kiện là thoát)
```
