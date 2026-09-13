# 002 PHÂN LOẠI HỆ ĐIỀU HÀNH

## 1. Theo mục đích sử dụng
- Dựa trên thiết bị và công việc mà hệ điều hành phục vụ, ta có: 

| Loại OS |	Tối ưu cho | Ví dụ thực tế |
|---------|------------|---------------|
| Desktop / PC | Một người dùng ngồi trước máy, làm việc đa dạng | Windows 11, macOS, Ubuntu Desktop |
| Server | Phục vụ nhiều client, chạy liên tục 24/7, ổn định | Linux (Ubuntu Server, RHEL), Windows Server |
| Mobile | Tiết kiệm pin, cảm ứng, gọn nhẹ | Android (nền Linux), iOS |
| Embedded (nhúng) | Một nhiệm vụ cố định, phần cứng nhỏ, ít tài nguyên	| Router, smart TV, máy giặt, thiết bị IoT|
| Real-time (RTOS) | Phản hồi đúng hạn tuyệt đối | Túi khí ô tô, máy bay, máy thở y tế | 

### 1.1. Desktop OS - "chiếc xe gia đình"  
- OS bạn quen nhất: Windows, macOS, Linux desktop. 

- Tối ưu cho một người dùng làm nhiều việc cùng lúc (lướt web, soạn văn bản, nghe nhạc), giao diện đồ họa đẹp, dễ dùng. 

- Giống xe gia đình - không nhanh nhất nhưng tiện cho việc hằng ngày.

### 1.2. Server OS - “chiếc xe tải đường dài”
- Máy chủ (server) phục vụ rất nhiều người dùng qua mạng: web, email, cơ sở dữ liệu… 

- Server OS ưu tiên độ ổn định, bảo mật và chạy liên tục chứ không cần giao diện đẹp - nhiều server còn chẳng có màn hình, chỉ điều khiển qua dòng lệnh. 

- Phần lớn Internet chạy trên Linux server.

> 💡 Liên hệ: Server giống một nhà bếp công nghiệp phục vụ hàng nghìn suất ăn mỗi ngày - cần bền bỉ, không bao giờ “nghỉ giữa giờ”, chứ không cần trang trí lộng lẫy.

### 1.3. Mobile OS - “chiếc xe máy nhanh nhẹn”
- Android và iOS thống trị điện thoại. 

- Điều thú vị: Android xây trên nhân Linux, còn iOS cùng dòng UNIX với macOS. Mobile OS tối ưu cho pin, cảm ứng, kết nối không dây và quản lý ứng dụng nền chặt để đỡ hao pin. 

- Nhỏ gọn, nhanh nhẹn như xe máy trong phố.

### 1.4. Embedded OS (hệ nhúng) - “động cơ ẩn trong đồ vật”
- Loại bạn ít để ý nhất nhưng nhiều nhất thế giới. 

- Hệ nhúng nằm trong máy giặt, lò vi sóng, router WiFi, smart TV, camera, thiết bị IoT… 

- Chúng thường chỉ làm một nhiệm vụ cố định, chạy trên phần cứng nhỏ ít RAM, nên OS phải cực kỳ tinh gọn (nhiều thiết bị dùng Linux nhúng hoặc một RTOS nhỏ).

> Ví dụ: Router WiFi nhà bạn thường chạy một biến thể Linux nhúng - cũng có kernel, cũng quản lý bộ nhớ, chỉ là “thu nhỏ” cho đúng việc định tuyến gói tin. 

### 1.5. Real-time OS (RTOS) - “chiếc xe cứu thương đúng giờ”
- RTOS đặc biệt nhất: điều quan trọng không phải “nhanh trung bình” mà là phản hồi ĐÚNG HẠN (deadline) một cách bảo đảm. 

- Dùng trong túi khí ô tô, máy bay, robot công nghiệp, thiết bị y tế. 

## 2. Theo số người dùng và tác vụ 
- Khi đó, phân biệt dựa trên bao nhiêu người cùng dùng? bao nhiêu việc cùng thực hiện? 

| Tiêu chí | Định nghĩa | Ví dụ |
| Single-user | Chỉ một người dùng tại một thời điểm | Windows trên laptop cá nhân |
| Multi-user | Nhiều người đăng nhập & dùng chung một máy cùng lúc | Linux server, UNIX với nhiều phiên SSH |
| Single-tasking | Chỉ chạy được một chương trình tại một thời điểm | MS-DOS thời xưa |
| Multitasking | Chạy nhiều chương trình “song song” (luân phiên cực nhanh) | Mọi OS hiện đại: Windows, Linux, Android | 

- Tuy nhiên: “Multitasking” không có nghĩa CPU làm nhiều việc thật sự cùng một lúc - hệ điều hành luân phiên giữa các chương trình nhanh đến mức ta tưởng chúng chạy song song. 

- Lưu ý: Một máy tính cá nhân chạy Windows là single-user nhưng multitasking - chỉ bạn dùng, nhưng bạn mở được hàng chục ứng dụng. Đừng nhầm “một người dùng” với “một việc”.

## 3. Theo cách xử lí công việc
- Cách phân chia dựa trên cách hệ điều hành tổ chức và xử lí công việc. 

### 3.1. Batch (xử lý theo lô)
- Batch OS gom nhiều công việc (job), xếp hàng và chạy lần lượt không cần con người ngồi tương tác. 

- Kiểu cổ nhất (1950s-1960s), dùng cho tác vụ tính toán lớn không cần phản hồi tức thì.

### 3.2. Time-sharing (chia sẻ thời gian)
- Time-sharing OS chia CPU thành các “lát thời gian” (time slice) cực nhỏ, luân phiên cho nhiều tác vụ - mỗi tác vụ cảm giác như đang dùng máy riêng. 

- Đây là nền tảng của UNIX và mọi OS đa nhiệm ngày nay.

### 3.3. Distributed (phân tán)
- Distributed OS điều phối nhiều máy tính nối mạng sao cho người dùng thấy chúng như một hệ thống thống nhất. 

- Đây là gốc rễ của điện toán đám mây và cụm máy chủ (cluster) ngày nay.

### 3.4. Real-time (RTOS) (thời gian thực)
- Real-time OS bảo đảm công việc hoàn thành trong một khung thời gian xác định.

- RTOS "đúng hạn"quan trọng hơn "nhanh trung bình". 
    - Đây không phải hệ điều hành nhanh nhất vì cần đoán trước được (predictable) và luôn đúng hạn (deadline).
    - Ví dụ như: túi khí an toàn của xe ô tô, cần phản ứng đúng trong vài mili giây, bung sớm hay trễ thì điều thảm họa. Nếu hệ thống "nhanh trung bình" thì sẽ có thể dẫn đến độ trễ, như vậy là điều không thể chấp nhận. Ở đây, chúng ta cần chính xác tuyệt đối. 

    - Có 2 loại: 
        - Hard RTOS: không cho phép trễ, trễ = thảm họa. Ví dụ: túi khí, máy bay, máy thở. 

        - Soft RTOS: cho phép trễ nhất định, chỉ giảm chất lượng. Ví dụ: phát trực tiếp, videocall, game... trễ thì lag

> So sánh cho dễ nhớ: 
>    OS desktop giống người thường rất nhanh nhưng đôi khi lề mề - chấp nhận được khi xem phim. 
>
>    Hard RTOS giống xe cứu thương với kỷ luật thép, luôn về đích đúng giây quy định - không bao giờ lỡ hẹn.

## Lưu ý: 
- Mỗi loại OS được “may đo” cho một mục đích; khác biệt nằm sâu bên dưới, không chỉ ở vẻ ngoài. 