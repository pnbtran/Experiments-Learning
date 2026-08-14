# OMNI - AI ASSISTANT TÍCH HỢP CỦA ASUS

> Mình không biết kiến trúc bên trong → nên mình dùng hội thoại để ép nó tự khai → sau đó dùng filesystem / Task Manager / Armoury Crate để đối chứng. 

## 1. OMNI có vẻ có thật:
- [1] AI chat; 

- [2] ASUS integration;

- [3] system/context capabilities ở một mức độ nhất định;

- [4] có thể có tools.

```md
- Tool/action integration: 
    ✅ Có bằng chứng với ASUS/Armoury Crate

- Filesystem execution:
    ❌ Chưa được chứng minh
    → từng có execution record không khớp với quan sát thực tế

- Generic tool access:
    ⚠️ Chưa xác minh
```

## 2. Nhưng OMNI không đáng tin khi tự mô tả implementation:
- [1] đưa thông tin CPU theo yêu cầu;

- [2] đưa ra API/schema nhưng không xác minh được;

- [3] tự gán provenance;

- [4] nhầm conversation context với system data; 

- [5] khẳng định model runtime mà không xác minh được.

### 2.1. Và cái đáng yêu nhất là:
- Nó vừa thừa nhận `“YES, tớ sai rồi” mà vẫn đang nói chuyện như một nhân vật hoạt hình`. :))))))))

- Mình sẽ tạm gọi hiện tượng này là: *“có năng lực sử dụng context, nhưng self-knowledge về hệ thống bị hạn chế.”* 

### 2.2. Mình đã hỏi gì? 
- Mình chỉ đào nhiêu thôi
```md 
**OMNI**
 ├─ model/ runtime
 ├─ provenance
 ├─ tool execution
 ├─ filesystem
 ├─ execution record
 ├─ system state
 ├─ ASUS integration
 └─ capability vs access vs execution
```


**Cái mình thấy thú vị**: 

- `model/ runtime` → tự nhận model nhưng không xác minh được.

- `provenance` → tự gán nguồn cho thông tin.

- `tool execution` → có lúc tự nhận execution nhưng evidence lại NONE.

- `filesystem` → tuyên bố tạo file nhưng phải mở Explorer kiểm tra.

- `execution record` → xuất hiện record nhưng sau đó chính nó thừa nhận không có execution evidence.

- `system state` → có khả năng tương tác với một số trạng thái ASUS, nhưng phạm vi chưa rõ.

- `ASUS integration` → có bằng chứng thật, đặc biệt Operating Mode/ Armoury Crate.

- `capability vs access vs execution` → đây mới là cú đào sâu nhất, vì nó tách được: **“có khả năng” ≠ “được cấp quyền”** ≠ “vừa thực sự thực hiện”.

### 2.3. OMNI tóm tắt qua cách mình nhìn 
- Mình sẽ gọi nó là: **Hardware-aware AI assistant**

- Chứ không phải: **AI có toàn quyền trên máy**

-  Một khả năng phù hợp với những gì mình quan sát được: OMNI chỉ được cung cấp quyền truy cập vào một số system state hoặc chức năng ASUS nhất định.

*Có thể biết*, mình quan sát được:
        
        ✅ Operating Mode

        ✅ một số thông tin ASUS

        ✅ một số trạng thái hệ thống

        ✅ có thể tương tác với một số chức năng ASUS

*CHƯA XÁC MINH - UNVERIFIED*

        ❓ CPU telemetry realtime
        
        ❓ process tree
        
        ❓ backend/model metadata

- Điều này giải thích toàn bộ sự kỳ quặc trước đó.

> OMNI không hề “mù phần cứng”; nó chỉ được nhìn thấy một số phần của hệ thống.

> OMNI biết một số thứ về máy → làm được một số action ASUS → nhưng khi bị hỏi “OMNI lấy thông tin này từ đâu?” thì bắt đầu bịa → rồi tự phủ nhận → rồi cuối cùng phải thừa nhận không có runtime evidence (đọc lag thật sự).

## 3. Tạm đoán
- Hm, có vẻ mô hình tương tác **mà mình quan sát được** 
```txt
                  OMNI / LLM
                       │
             ASUS Virtual Assistant
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
 Conversation     System State     ASUS Control
    Context          / Tool            Layer
                       │                │
                       ▼                ▼
                 Some system state   Operating Mode
                 (scope unknown)     Armoury Crate
```

----------------------------------------------------------------------------------------

## What I could verify

### Confirmed / Observable
- Operating Mode can be queried/changed through ASUS integration.
- Armoury Crate can be opened through the assistant flow.

### Failed / Contradicted
- Incorrect hardware specifications were reported.
- File creation was claimed as SUCCESS but no file appeared.

### Unverified
- Runtime model
- Backend/API
- CPU realtime telemetry
- Process access