# Bài tập — Mổ App AI: Vietnam Airlines NEO

**Sản phẩm chọn:** Vietnam Airlines — Chatbot NEO  
**Evidence:** Transcript + 3 screenshot chat Web VNA  
**Ngày:** 03/06/2026  
**Học viên:** Nguyễn Minh Hiếu — 2A202600705

---

## 1. Dùng thử — Promise vs Reality

### Product hứa gì?

Theo [trang NEO chính thức](https://www.vietnamairlines.com/vn/vi/support/chatbot):

> *"Thuận tiện tra cứu, giải đáp nhanh chóng (24/7) mọi thắc mắc liên quan đến thông tin hành trình, mua vé, thanh toán..."*

NEO hứa: tra cứu vé/chuyến bay, **tìm kiếm giá vé**, giải đáp hoàn/đổi vé, hành lý — chat ngắn gọn, rõ ràng.

### 3 query đã thử (chat thật)

| # | Prompt | Kết quả | Path |
|---|---|---|---|
| 1 | *"hoàn tiền chuyến bay"* | Trả FAQ chi tiết, đúng quy trình hoàn vé online/offline + thời gian hoàn tiền | **Happy** |
| 2 | *"tiền vé bao nhiêu tiền"* | Chuyển sang slot-filling: hỏi loại vé, điểm đi/đến, ngày, hành khách, hạng | Low-confidence → Failure |
| 3 | *"một chiều HN → HCM, 1 người lớn, hạng thương gia"* + *"bao nhiêu tiền vé"* (×2) | Thu thập đủ slot trừ **Ngày đi** — lặp lại cùng template 3 lần, **không trả giá** | **Breaking point** |

### Điểm gãy chính (từ transcript)

1. **Loop slot-filling cứng:** User hỏi giá 3 lần (*"bao nhiêu tiền vé"*, *"hạng thương gia thì bao nhiêu tiền"*) — NEO chỉ lặp *"Vui lòng cung cấp: Ngày đi"*, không giải thích vì sao, không đưa khoảng giá tham khảo.
2. **Intent bị hiểu sai mức độ:** User có thể chỉ muốn **ước lượng giá** (research), không phải đặt vé — NEO ép vào flow booking đầy đủ slot.
3. **Không nhận diện user đang stuck:** Cùng một block text lặp lại → cảm giác bot "đơ", không nghe câu hỏi mới.
4. **CSAT sớm:** Sau câu hoàn tiền, NEO hỏi đánh giá 1–5⭐ trong khi user chưa hoàn thành task tra giá vé.

---

## 2. Screenshot — chat thật (Web VNA)

### Ảnh 1 — Query #1: Hoàn tiền (Happy path)

![NEO trả FAQ hoàn vé chi tiết + hỏi CSAT 1–5 sao](./screenshots/neo-01-hoan-tien.png)

*User: "hoàn tiền chuyến bay" → NEO trả hướng dẫn website/app/đại lý, timeline 15 ngày, sau đó hỏi đánh giá.*

---

### Ảnh 2 — Query #2: Bắt đầu slot-filling giá vé

![NEO yêu cầu đủ slot khi user hỏi giá vé](./screenshots/neo-02-slot-filling.png)

*User: "tiền vé bao nhiêu tiền" → chưa có info. User bổ sung "một chiều HN → HCM" → NEO cập nhật HAN/SGN, vẫn thiếu ngày đi, hành khách, hạng.*

---

### Ảnh 3 — Query #3: Loop khi hỏi giá (Breaking point)

![NEO lặp template thiếu Ngày đi dù user hỏi bao nhiêu tiền vé](./screenshots/neo-03-loop-gia-ve.png)

*Đã có: một chiều, HAN→SGN, 1 NL, hạng thương gia. User: "bao nhiêu tiền vé" → NEO **lặp y hệt** block "Thiếu: Ngày đi", không trả giá.*

---

## 3. Transcript rút gọn (evidence)

```text
User: hoàn tiền chuyến bay
NEO: [FAQ hoàn vé chi tiết — OK ✅]
NEO: Đánh giá 1–5 sao?

User: tiền vé bao nhiêu tiền
NEO: Slot-filling — thiếu tất cả fields

User: một chiều từ hà nội đến hcm
NEO: Đã có HAN→SGN, một chiều — thiếu ngày đi, hành khách, hạng

User: 1 người lớn hạng thương gia
NEO: Đủ trừ Ngày đi — thiếu ngày đi

User: bao nhiêu tiền vé          ← lần 1
NEO: [Lặp lại — thiếu Ngày đi]

User: hạng thương gia thì bao nhiêu tiền  ← lần 2
NEO: [Lặp lại y hệt — thiếu Ngày đi]  ❌

User: (implicit lần 3 — cùng pattern)
NEO: [Lặp lại y hệt]  ❌
```

---

## 4. Vẽ 4 paths (Mermaid)

### Tổng quan session chat thật

```mermaid
flowchart LR
    START([Mở NEO]) --> Q1["hoàn tiền chuyến bay"]
    Q1 --> H1[✅ FAQ đầy đủ]
    H1 --> CSAT[⭐ Hỏi rating sớm]
    CSAT --> Q2["tiền vé bao nhiêu"]
    Q2 --> SLOT[Slot-filling giá vé]
    SLOT --> LOOP["❌ Loop: thiếu Ngày đi<br/>×3 lần không thoát"]

    style H1 fill:#c8e6c9
    style LOOP fill:#ffcdd2
    style CSAT fill:#fff9c4
```

---

### Path 1 — Happy (hoàn tiền)

```mermaid
flowchart TD
    A([User: hoàn tiền chuyến bay]) --> B{NEO nhận diện FAQ}
    B -->|Intent rõ| C[Trả quy trình hoàn vé<br/>Website + App + Đại lý]
    C --> D[Thêm lưu ý loại trừ + timeline 15 ngày]
    D --> E([✅ User có đủ info để hành độn])

    style E fill:#c8e6c9
```

---

### Path 2 — Low-confidence (thiếu slot — nhưng xử lý kém)

```mermaid
flowchart TD
    A([User: tiền vé bao nhiêu]) --> B[NEO vào flow Tìm giá vé]
    B --> C{Đủ slot?}
    C -->|Thiếu nhiều| D[Liệt kê fields cần bổ sung]
    D --> E[User bổ sung dần: HAN→SGN, 1 NL, TG]
    E --> F{Chỉ thiếu Ngày đi}
    F --> G["❌ NÊN: hỏi ngày + giải thích<br/>+ offer 'xem giá 7 ngày tới'"]
    F -->|Thực tế| H[Lặp template y hệt]
    H --> I([User stuck — không có path thoát])

    style G fill:#fff9c4,stroke-dasharray: 5 5
    style H fill:#ffcdd2
    style I fill:#ffcdd2
```

---

### Path 3 — Failure (loop + bỏ qua câu hỏi)

```mermaid
flowchart TD
    A["User: bao nhiêu tiền vé<br/>(đã có HAN→SGN, 1 NL, TG)"] --> B{NEO detect<br/>user hỏi lại giá?}
    B -->|Không| C[Chỉ check slot Ngày đi]
    C --> D["Trả lại block 'Thu thập được...<br/>Thiếu: Ngày đi'"]
    D --> E["User: hạng TG bao nhiêu tiền"]
    E --> F{Reframe intent?}
    F -->|Không| D
    D --> G["User hỏi lần 3"]
    G --> D
    D --> H([❌ GÃY: không bao giờ có giá<br/>User bỏ chat])

    style H fill:#ffcdd2,stroke:#c62828
```

**Failure mode:** User có **4/5 slot** nhưng không được trả **bất kỳ thông tin giá nào** — kể cả khoảng tham khảo.

---

### Path 4 — Correction (user cố sửa bằng cách nói rõ hơn)

```mermaid
flowchart TD
    A([User phát hiện NEO không trả giá]) --> B["Rephrase: 'hạng thương gia<br/>thì bao nhiêu tiền'"]
    B --> C{NEO hiểu là câu hỏi mới?}
    C -->|Không| D[Ignore rephrase → lặp slot template]
    D --> E([❌ Correction trong chat thất bại])

    B --> F["To-be: User chọn 'Chưa chọn ngày'"]
    F --> G[NEO trả khoảng giá + link search]
    G --> H([✅ User tiếp tục hoặc chọn ngày])

    style D fill:#ffcdd2
    style E fill:#ffcdd2
    style H fill:#c8e6c9,stroke-dasharray: 5 5
```

**Human handoff:** Trang VNA nói có chuyển tư vấn viên khi NEO không giải quyết được — **không xuất hiện** trong transcript này.

---

## 5. Path yếu nhất — chọn để sửa

**Path yếu:** *Tra giá vé khi user chưa có / không muốn chốt ngày bay*.

**Không phải bug lẻ:** Đây là workflow research phổ biến — user hỏi *"HN–SGN thương gia khoảng bao nhiêu"* trước khi quyết định ngày.

---

## 6. Finding → Product decision

```
Khi user đã cung cấp đủ route + hành khách + hạng vé nhưng chưa có ngày đi,
NEO lặp lại cùng template yêu cầu "Ngày đi" và bỏ qua câu hỏi "bao nhiêu tiền",
hậu quả là user bị kẹt trong loop, không nhận được khoảng giá tham khảo và bỏ chat.
Lỗi thuộc layer Intent + UX Recovery.
Nên sửa bằng: (1) nhận diện câu hỏi lặp → đổi chiến lược;
(2) offer "Xem giá rẻ nhất 7 ngày tới" hoặc khoảng giá;
(3) chip "Chưa chọn ngày — xem giá tham khảo";
(4) human handoff sau 2 lần stuck.
```

### Một câu quyết định product

> **NEO không được block hoàn toàn khi thiếu ngày bay — phải trả giá tham khảo hoặc lịch giá rút gọn trước khi ép user chốt ngày.**

---

## 7. Sketch As-is / To-be (Mermaid)

### So sánh As-is vs To-be

```mermaid
flowchart TB
    subgraph ASIS["🔴 AS-IS — từ chat thật"]
        direction TB
        A1["User: HAN→SGN, 1 NL, TG"] --> A2{Thiếu ngày?}
        A2 -->|Có| A3["Lặp: 'Thiếu Ngày đi'"]
        A3 --> A4["User hỏi giá lại"]
        A4 --> A3
        A3 --> A5["❌ Bỏ chat"]
    end

    subgraph TOBE["🟢 TO-BE — đề xuất"]
        direction TB
        B1["User: HAN→SGN, 1 NL, TG"] --> B2{Thiếu ngày?}
        B2 -->|Có| B3["'Chưa có ngày — bạn muốn:'"]
        B3 --> B4["Chip: 7 ngày tới / Tháng này / Chưa chọn ngày"]
        B4 --> B5["Trả khoảng giá hoặc bảng giá theo ngày"]
        B5 --> B6["Link 'Đặt vé với ngày X'"]
        B2 -->|User hỏi lại 2 lần| B7[Human handoff / FAQ giá]
    end

    style A5 fill:#ffcdd2
    style B5 fill:#c8e6c9
    style B6 fill:#c8e6c9
```

---

### As-is — chi tiết path giá vé

```mermaid
flowchart TD
    START([NEO chào]) --> R["✅ hoàn tiền → FAQ OK"]
    R --> CSAT["⭐ CSAT sớm"]
    CSAT --> P["tiền vé bao nhiêu tiền"]
    P --> S1[Slot-filling bắt đầu]
    S1 --> S2["User: HN → HCM một chiều"]
    S2 --> S3["User: 1 NL, hạng TG"]
    S3 --> CHECK{Ngày đi?}
    CHECK -->|Thiếu| LOOP["📋 Template lặp:<br/>'Thiếu: Ngày đi'"]
    LOOP --> U1["User: bao nhiêu tiền vé"]
    U1 --> LOOP
    LOOP --> U2["User: TG bao nhiêu tiền"]
    U2 --> LOOP
    LOOP --> FAIL["❌ GÃY #1: Không có giá<br/>❌ GÃY #2: Không detect repeat<br/>❌ GÃY #3: Không handoff"]

    style R fill:#c8e6c9
    style CSAT fill:#fff9c4
    style FAIL fill:#ffcdd2,stroke:#c62828
```

---

### To-be — path đã sửa

```mermaid
flowchart TD
    START([User hỏi giá vé]) --> PARSE[Thu thập slot]
    PARSE --> CHECK{Đủ route + hạng?}

    CHECK -->|Thiếu ngày| CLARIFY["🤖 'Giá phụ thuộc ngày bay.<br/>Bạn muốn xem theo cách nào?'"]
    CLARIFY --> CHIPS["📅 7 ngày tới · 📆 Tháng này<br/>🔍 Chưa chọn ngày — xem khoảng giá"]
    CHIPS --> C1[Chọn 7 ngày tới]
    CHIPS --> C2[Chưa chọn ngày]

    C1 --> PRICE1["Bảng giá theo ngày<br/>+ giá rẻ nhất highlight"]
    C2 --> PRICE2["Khoảng giá TG HAN-SGN:<br/>~X – Y triệu (tham khảo)"]
    PRICE1 --> CTA[Link đặt vé / chọn ngày cụ thể]
    PRICE2 --> CTA

    CHECK -->|User hỏi lại ≥2 lần| ESCALATE["Chuyển tư vấn viên<br/>hoặc mở trang Tìm vé"]
    ESCALATE --> DONE([✅ User không bị loop])

    PARSE --> REPEAT{Detect câu hỏi lặp?}
    REPEAT -->|Có| CLARIFY

    style PRICE1 fill:#c8e6c9
    style PRICE2 fill:#c8e6c9
    style DONE fill:#c8e6c9
    style ESCALATE fill:#e3f2fd
```

---

### Sequence — To-be (User ↔ NEO)

```mermaid
sequenceDiagram
    actor U as User
    participant N as NEO
    participant F as Fare API

    U->>N: HAN→SGN, 1 NL, hạng TG — bao nhiêu tiền?
    N->>N: Detect: thiếu ngày + user hỏi giá
    N->>U: Giá thay đổi theo ngày. Chọn cách xem?
    Note over N,U: Chips: 7 ngày tới | Tháng này | Khoảng tham khảo

    U->>N: Chưa chọn ngày — xem khoảng giá
    N->>F: Query fare range HAN-SGN Business
    F-->>N: ~4.5M – 8.2M VND
    N->>U: Khoảng tham khảo + "Chọn ngày để xem giá chính xác"
    N->>U: 🔗 Mở Tìm vé / Đặt chỗ

    U->>N: (Nếu hỏi lại lần 3)
    N->>U: Chuyển tư vấn viên hoặc deep link search
```

---

## 8. Tự kiểm

- [x] Evidence chat thật (transcript)
- [x] Screenshot app (3 ảnh — `screenshots/`)
- [x] Đủ 4 paths
- [x] Finding = product decision
- [x] Mermaid as-is + to-be

---

## 9. Build slice Day 06 (track VNA)

```text
Cho hành khách đang research giá vé chưa chọn ngày bay,
prototype dùng AI detect "price inquiry without date" và offer 3 lựa chọn xem giá,
tạo ra khoảng giá tham khảo hoặc bảng 7 ngày,
và xử lý loop failure bằng repeat-detection + human handoff sau 2 lần stuck.
```

