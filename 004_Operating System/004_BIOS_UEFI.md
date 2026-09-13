# 004 QUÁ TRÌNH KHỞI ĐỘNG MÁY - BIOS/UEFI VÀ BOOTLOADER
## 1. Các bước khởi động máy tính
- Khi máy đang tắt, RAM trống, hệ điều hành nằm im trong ổ cứng. 

```txt
  Nhấn nút nguồn
   │
   > [1] FIRMWARE (BIOS/UEFI)   chip bo mạch chủ tỉnh dậy đầu tiên
   > [2] POST                    kiểm tra RAM, CPU, bàn phím... còn sống không?
   > [3] Tìm THIẾT BỊ KHỞI ĐỘNG  theo boot order: ổ SSD? USB? mạng?
   > [4] Nạp BOOTLOADER          chương trình tí hon ở đầu đĩa (GRUB / Windows Boot Manager)
   > [5] Bootloader nạp KERNEL   vào RAM  rồi trao quyền cho kernel
   > [6] KERNEL khởi tạo         dò thiết bị, nạp driver, dựng bộ nhớ ảo
   > [7] INIT / SYSTEMD (PID 1)  tiến trình đầu tiên, bật các dịch vụ nền
   > [8] Màn hình ĐĂNG NHẬP      máy đã sẵn sàng phục vụ bạn!

```
> 3 bước đầu thuộc về phần cứng, từ bước 5 thuộc về hệ điều hành 

## 2. Firmware: BIOS/UEFI - người gác cổng đánh thức phần cứng
- **Firmware** là phần mền được cài sẵn vào chip nhỏ trên bo mạch chủ (mainboard) 

- Có 2 thế hệ firware là BIOS (Basic Input/Outout System) chuẩn cũ, thập niên 1980 và UEFI (Unified Firware Interface) chuẩn hóa hiện đại thay thế BIOS, phổ biến từ 2012. 

- Nhiệm vụ cốt lõi đánh thức phần cứng, chạy POST, tìm thiết bị khởi động và nạp bootloader và cung cấp màn hình setup để chỉnh giờ, boot order, bật tắt tính năng. 

### 2.1. POST 
- POST (Power-On Self-Test) có nhiệm vụ, dò RAM, CPU, card màn hình, bàn phím... còn hoạt động không

- Nếu có lỗi, máy thường kêu bíp theo mã hoặc hiện mã lỗi. 

### 2.2. Boot order
- Sau POST, firware tìm thiết bị chứa hệ điều hành theo danh sách ưu tiên, được gọi là boot order

## 3. So sánh BIOS và UEFI

| Tiêu chí | BIOS (Cũ) | UEFI (mới) |
|----------|-----------|------------|
| Thời gian | 1980-2010 | 2010 đến nay |
| Kiểu phân vùng trên đĩa | MBR (mater boot Record) | GPT (GUID Partiton Table) |
| Giới hạn đĩa khởi động/ số phân vùng | ~ 2.2TB/ 4 phân vùng chính | Lớn (~9.4 ZB)/ 128 phân vùng |
| Giao diện cấu hình | Chữ, chỉ dùng bàn phím | Đồ họa, có thể dùng chuột | 
| Tốc độ khởi động | Chậm hơn | Nhanh hơn (khởi tạo song song) | 
| Serure Boot | Không có | Có (chống mọi malware nạp sớm) |

### 3.1. Phân biệt MBR và GPT
- MBR và GPT là cách ghi mục lục phân vùng ở đầu ổ đĩa, để firware biết OS nằm ở phân vùng nào. 

- MBR (cũ): 1 bản mục lục gói trong 512 byte đầu, tối đa 4 phân vùng chính, đĩa ≤ 2.2 TB. GPT (mới): tới 128 phân vùng, đĩa rất lớn, có bản sao dự phòng ở cuối đĩa.

>  Vì sao GPT an toàn hơn? GPT lưu 2 bản bảng phân vùng (đầu và cuối đĩa) - hỏng bản này còn bản kia. MBR chỉ có 1 bản; hỏng là mất hết mục lục, dù dữ liệu thật vẫn còn. Giống photo dự phòng tờ danh bạ quan trọng phòng khi rách bản gốc. 

### 3.2. Secure boot
- Tính năng của UEFI, chỉ cho nạp những bootloader/kernel đã được ký số (digital signature) bởi nguồn tin cậy.

## 4. Bootloader - chương trình nối firware và kernel
- Sau khi firmware tìm thấy thiết bị khởi động, nó nạp một chương trình nhỏ gọi là bootloader (trình nạp khởi động). 

- Nhiệm vụ duy nhất nhưng tối quan trọng: tìm kernel, nạp nó vào RAM, rồi trao quyền điều khiển cho kernel. 

| Bootloader | Dùng cho | Ghi chú |
|------------|----------|---------|
| GRUB (GRand Unified Bootloader) | Linux (phổ biến nhất) | Hỗ trợ menu chọn nhiều OS, nhiều nhân |
| Windows Boot Manager (bootmgr) | Windows | Quản lý khởi động Windows |
| systemd-boot | Linux (UEFI, gọn nhẹ) | Đơn giản hơn GRUB | 

- **Multi-boot**: một máy, nhiều hệ điều hành
    Nếu máy bạn cài cả Windows lẫn Linux, bootloader (thường là GRUB) sẽ hiện một menu lúc khởi động để bạn chọn vào hệ điều hành nào (dùng phím mũi tên chọn, Enter để vào). Đây gọi là multi-boot (hoặc dual-boot khi có 2 OS).

## 5. Kernel khởi tạo và init/systemd lên ngôi 
- Khi *bootloader trao quyền*, *kernel bắt đầu màn trình diễn chính*: tự giải nén vào bộ nhớ (kernel thường được nén để tiết kiệm chỗ), dò tìm phần cứng và nạp driver tương ứng (đĩa, mạng, USB…), dựng bộ nhớ ảo cùng bảng tiến trình (học sâu ở Phần 2, 3), rồi cuối cùng khởi chạy tiến trình đầu tiên của không gian người dùng. 

- Tiến trình đầu tiên ấy mang số hiệu PID 1 (Process ID 1) và là điểm bắt đầu của mọi tiến trình khác trên máy. Trên Linux hiện đại, nó thường là systemd; các hệ cũ hơn dùng init (SysV init). *Nhiệm vụ*: khởi động các dịch vụ (service) nền (mạng, âm thanh, đăng nhập, máy in…) theo đúng thứ tự, rồi bật màn hình đăng nhập - máy sẵn sàng.

## 6. Tổng kết
### 6.1. Firware không phải hệ điều hành
- Vì sao? 
| -- | Firmware (BIOS/UEFI) | Hệ điều hành |
|----|----------------------|--------------|
| Chỗ lưu | Chip trên bo mạch chủ | Trên ổ cứng/ SSD | 
| Thời gian chạy | Vài giây đầu trước khi trao quyền cho OS | chạy suốt phiên làm việc | 
| Quản lí đa nhiệm, ứng dụng? | không | Có - nhiệm vụ OS | 

### 6.2. Boot từ USB và boot qua mạng (PXE)
- Boot từ USD nạp từ bootloader từ thanh USD - cách cài OS từ USB cứu hộ 
- Boot qua mạng (PXE - Preboot eXecution Enviroment): máy tải OS từ server qua mạng LAN rồi khởi động.

### 6.3. Vì sao Secure boot quan trọng? 
- Malware nguy hiểm nhất là loại nạp sớm hơn cả hệ điều hành - bootkit/rootkit. Chiếm được bootloader, nó chạy trước kernel, nên “che mắt” được cả phần mềm diệt virus (vốn chạy sau, trong OS). 

- Giống kẻ gian trà trộn vào trước khi bảo vệ tới ca trực thì rất khó phát hiện. Secure Boot chặn đúng kịch bản này: firmware từ chối nạp bootloader/kernel không có chữ ký hợp lệ - cắt đứt con đường chen vào sớm của bootkit. 