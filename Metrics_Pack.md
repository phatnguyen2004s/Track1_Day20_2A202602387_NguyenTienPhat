# Metrics Pack — Day 20

**Họ tên:** Nguyễn Tiến Phát · **MHV:** 2A202602387 · **Dự án:** AI Customer Support Agent

> Bài làm cá nhân. Chuỗi quyết định:
> Core action → Nhịp tự nhiên (nature) → Bộ metric + retention → Product loop (giả thuyết) → Tracking (phép thử)
>
> Việc dùng AI trong bài được khai báo tại [ai-support-log.md](./ai-support-log.md).

## Giả định của bài

Các giả định dưới đây thay cho dữ liệu thật mà bài chưa có. Cần kiểm khi có dữ liệu ticket thật.

- **A1. Nhịp ticket:** Ticket phát sinh ở nhiều ngày làm việc và một nhân viên thường xử lý nhiều ticket trong một ngày. Đây là giả định có thể kiểm tra lại bằng dữ liệu `ticket_opened` thực tế.
- **A2. Effort:** khoảng 3–5 phút mỗi ticket.
- **A3. Chất lượng:** khách quay lại nếu vấn đề chưa xong thường trong 7 ngày (N = 7).
- **A4. Activation:** nhân viên mới chạm first value trong 7 ngày kể từ lúc bật copilot.
- **A5. Retention:** mốc tuần 1, 2, 4 sau tuần vào cohort; ngưỡng dùng đều đặn là ≥ 3 ngày làm việc trong tuần.
- **A6. Hypothesis:** khung 4 tuần đầu đủ để thấy nhân viên quen cách dùng gợi ý.
- **A7. Tracking:** hệ thống ghi được các event `ticket_escalated`, `csat_submitted` và chi phí mỗi gợi ý.
- **A8. Lợi ích so với tự viết:** Dùng gợi ý giúp soạn và gửi phản hồi nhanh hơn so với tự viết từ đầu (reason to return, mục 05). Bài chưa có metric nào kiểm giả định này vì Depth chỉ tính trên ticket có dùng gợi ý; cần kiểm khi có dữ liệu so sánh.


---

## 00 — Phạm vi (Phase 0)

1. **Dự án:** AI Customer Support Agent
2. **Persona (một persona thôi):** Nhân viên CSKH (support agent) — người dùng AI để xử lý ticket.
3. **Core job** (lời người dùng, không phải tính năng):
   > “Tôi phải trả lời hàng chục ticket lặp lại mỗi ngày mà vẫn phải đúng chính sách và không làm khách bực thêm.”

---

## 01 — Core Action (Phase 1)

### 1.1 Phân biệt bốn khái niệm

| Khái niệm | Câu hỏi | Nội dung |
|---|---|---|
| Core job | User đang cố hoàn thành việc gì? | Xử lý nhiều ticket lặp lại mỗi ngày mà vẫn đúng chính sách, không làm khách bực thêm |
| Core action | User làm gì trong sản phẩm để tiến tới giá trị? | Giải quyết một ticket bằng câu trả lời có dùng gợi ý của copilot |
| Core value | User nhận được lợi ích gì? | Xử lý xong ticket nhanh hơn mà vẫn đúng chính sách; khách hết vấn đề |
| Core value event | Sự kiện nào chứng minh value đã xảy ra? | `ticket_resolved` (và không có `ticket_reopened` trong 7 ngày) |

### 1.2 Core Action Card

| Thành phần | Nội dung |
|---|---|
| Target user (Ai thực hiện hành vi?) | Nhân viên CSKH trực tiếp xử lý ticket với khách |
| Core job (Họ đang cố hoàn thành việc gì?) | Như mục 00 |
| Core action (Hành vi được chọn là gì?) | Giải quyết ticket: nhân viên gửi câu trả lời có dùng gợi ý của copilot (có thể đã sửa) rồi xác nhận ticket đã giải quyết |
| Object (Hành vi tác động lên đối tượng nào?) | Ticket (một yêu cầu hỗ trợ của khách) |
| Preconditions (Điều gì phải có trước khi hành vi xảy ra?) | Ticket đã được giao cho nhân viên; copilot đã tạo ít nhất một gợi ý cho ticket đó |
| Completion rule (Khi nào action được xem là hoàn tất?) | Nhân viên chuyển ticket sang trạng thái "resolved" (không tính tự đóng do hết hạn) **và** ít nhất một câu trả lời đã gửi cho khách mang cờ `suggestion_id` (hệ thống lưu khi nhân viên chèn gợi ý vào bản nháp; giữ nguyên dù sau đó sửa bao nhiêu) |
| Core value (Người dùng nhận được lợi ích gì?) | Như bảng 1.1 |
| Evidence of value (Dấu hiệu nào chứng minh value đã xảy ra?) | Ticket không bị mở lại trong 7 ngày và không phải chuyển cấp sau khi gửi |
| Candidate event (Event nào có thể dùng để tracking?) | `ticket_resolved` (kèm thuộc tính `ai_suggestion_used`); `ticket_reopened` để loại trừ |

### 1.3 Tự kiểm 5 tiêu chí (trượt ≥ 2 → phải chọn lại)

| # | Tiêu chí | Câu hỏi | Qua? (✓/✗) | Lý do |
|---|---|---|---|---|
| 1 | Gần core value | Hành vi xảy ra là user đã tiến gần rõ rệt tới value chưa? | ✓ | Nhân viên đã xử lý xong ticket (value chính của core job). "Đúng chính sách / khách hết bực" không do event đóng ticket chứng minh mà dựa vào evidence 7 ngày. |
| 2 | Có thể lặp lại | Hành vi có xuất hiện lại khi nhu cầu quay lại không? | ✓ | Mỗi ticket mới là một lý do xuất hiện lại; core job là xử lý hàng chục ticket mỗi ngày, không cần ai nhắc. |
| 3 | Có thể quan sát | Bạn biết chính xác khi nào nó hoàn tất không? | ✓ | "Resolved" quan sát được từ trạng thái ticket. "Có dùng gợi ý" nhận biết bằng cờ `suggestion_id` lưu lúc nhân viên chèn gợi ý vào bản nháp, không phụ thuộc mức sửa sau đó. |
| 4 | Có ý nghĩa | Hành vi tăng có thật sự nghĩa là sản phẩm tốt hơn không? | ✓ (có điều kiện) | Điều kiện "không mở lại 7 ngày, không chuyển cấp" lọc được kiểu đóng ticket vội. Chưa lọc được khách bỏ đi lặng lẽ mà không mở lại → cần counter-metric ở Phase 3. |
| 5 | Có thể tác động | Team có thể cải thiện khả năng nó xảy ra không? | ✓ | Team chỉnh được chất lượng gợi ý, vị trí hiển thị, độ phủ loại ticket, độ trễ. |

**Kết quả:** 5/5

### ✅ GATE 1 — Core action đứng vững
- [x] Core action có actor, object, completion rule
- [x] Qua ít nhất 4/5 tiêu chí tự kiểm (5/5)
- [x] Giải thích được vì sao nó **không** phải "mở app" hay "hỏi AI":

  > Core action của mình không phải "mở app" hay "hỏi AI" vì đó chỉ là thao tác giao diện: nhân viên có thể mở app hoặc bấm xem gợi ý cả ngày mà không giải quyết được ticket nào. Core action chỉ hoàn tất khi ticket được chuyển sang "resolved" và câu trả lời đã gửi cho khách mang cờ `suggestion_id` — tức là việc của nhân viên (xử lý xong ticket cho khách) đã thật sự xảy ra. Gợi ý được tạo ra hay được xem chỉ là output hệ thống, không chứng minh ai nhận được value.

---

## 02 — Nature & cadence (Phase 2)

### 2.1 Action Nature Card

| Thành phần | Câu hỏi | Nội dung |
|---|---|---|
| Actor | User, account, team hay object nào thực hiện? | Từng nhân viên CSKH (cấp cá nhân), làm việc trên hàng đợi ticket của team |
| Intent | Hành vi bắt đầu từ nhu cầu gì? | Xử lý xong ticket được giao, để khách hết vấn đề mà vẫn đúng chính sách |
| Trigger | Do user chủ động, sự kiện bên ngoài, người khác hay hệ thống kích hoạt? | Sự kiện bên ngoài: khách gửi yêu cầu → ticket mới được giao vào hàng đợi của nhân viên. Nhân viên chủ động mở ticket để xử lý. Không do notification của copilot |
| Effort | Mất bao nhiêu thời gian, suy nghĩ, dữ liệu? | Thấp đến trung bình: đọc ticket, đọc gợi ý, chỉnh, gửi. Cỡ 3–5 phút mỗi ticket (giả định A2) |
| Value timing | Value xuất hiện ngay, trễ, tích lũy, hay phụ thuộc người khác? | Với nhân viên: ngay khi ticket resolved. Với khách: trễ và phụ thuộc người khác, chỉ chắc chắn khi không bị mở lại sau 7 ngày |
| State | Sau action, dữ liệu/trạng thái nào được giữ lại? | Trạng thái ticket = resolved; lịch sử hội thoại; `suggestion_id` gắn với câu trả lời đã gửi |
| Dependency | Có phụ thuộc nguồn cung, thành viên khác, approval, thời điểm? | Khách phản hồi (ticket có thể cần nhiều lượt); kho kiến thức/chính sách để gợi ý đúng; approval hoặc chuyển cấp với một số loại ticket (vd hoàn tiền); ca làm việc của nhân viên |
| Repeat condition | Điều kiện nào khiến action có lý do xuất hiện lại? | Có ticket mới được giao; ticket cũ bị khách mở lại. Lý do đến từ công việc, không cần nhắc |

### 2.2 Kết luận cadence

**Dạng hành vi:** phản ứng theo sự kiện (đã cân nhắc: thói quen thường xuyên · tiến trình tích lũy · theo dự án · giao dịch · workflow của team · theo chu kỳ)

**Dữ kiện tôi xác nhận là đúng với sản phẩm** (nguyên liệu cho câu "vì"):
- Ticket mới được tạo khi khách gửi yêu cầu hỗ trợ.
- Một nhân viên xử lý nhiều ticket trong cùng một ngày làm việc.
- Core action được lặp lại khi có ticket mới được giao hoặc ticket cũ được mở lại.
- Retention được định nghĩa theo số ngày làm việc có dùng core action trong từng tuần: ≥ 3 ngày làm việc/tuần (giả định A5).
- North Star được tổng hợp ở cấp nhân viên theo tuần; quality threshold của NSM cần theo dõi trong 7 ngày sau khi ticket được resolved.

**Kết luận cadence** (theo template “Đối với …, core action … thường xuất hiện … vì …. Do đó, nhịp đo phù hợp là … ở cấp …”):

> Đối với nhân viên CSKH, core action "giải quyết ticket" thường xuất hiện nhiều lần trong mỗi ngày làm việc vì mỗi ticket mới từ khách là một sự kiện kích hoạt và một nhân viên có thể xử lý nhiều ticket trong ngày. Do đó, nhịp đo phù hợp là hằng tuần ở cấp nhân viên, vì retention được đánh giá theo mức độ quay lại trong một tuần (≥ 3 ngày làm việc) và North Star cũng được tổng hợp theo tuần, trong khi chất lượng của ticket được kiểm tra qua cửa sổ 7 ngày.

**Kết luận ngắn:** Dạng hành vi là phản ứng theo sự kiện; tần suất tự nhiên là nhiều lần trong ngày; nhịp đo là hằng tuần ở cấp nhân viên.

### ✅ GATE 2 — Cadence từ nature, không từ dashboard
- [x] Kết luận cadence theo đúng template
- [ ] Có lý do "vì" đứng được — chưa đạt: nhịp tuần là lựa chọn cách đo để thống nhất với retention và cách tổng hợp metric, không phải kết luận về tần suất hành vi; đây là quyết định đo lường.
- [x] Nhịp đo không mâu thuẫn với dạng hành vi đã chọn (core action lặp lại nhiều lần trong ngày; đo gộp theo tuần ở cấp nhân viên để so sánh ổn định)

---

## 03 — Metric System (Phase 3)

### 3.1 Activation metric

- **Start event** (khi nào user bắt đầu?): `copilot_enabled` — copilot được bật cho nhân viên lần đầu (tài khoản của nhân viên có thể dùng gợi ý)
- **Activation event** (event nào xác nhận core action đầu tiên / first value?): `ticket_resolved` đầu tiên của nhân viên có cờ `suggestion_id` (đúng completion rule ở mục 01; không chờ 7 ngày "không mở lại", phần đó thuộc chất lượng)
- **Time window** (core action cần xảy ra trong bao lâu kể từ start event?): trong 7 ngày kể từ `copilot_enabled` (giả định A4; độc lập với khoảng 7 ngày "không mở lại" ở mục 01, chỉ trùng số)

### 3.2 Engagement metric — tối đa hai góc đo

Chọn từ: frequency (core action xảy ra bao nhiêu lần trong cadence tự nhiên) · depth (mỗi lần action tạo bao nhiêu value) · breadth (bao nhiêu workflow/use case được dùng).

1. **Depth** — median thời gian xử lý chủ động mỗi ticket: tính từ `ticket_opened` đến `reply_sent` đầu tiên có `suggestion_id`, chỉ trên các ticket có sử dụng gợi ý. Không tính phần chờ khách phản hồi sau đó (ô Dependency ở 2.1), để đo đúng thời gian nhân viên bỏ ra. Dùng median thay vì mean để giảm ảnh hưởng của một số ticket có thời gian xử lý bất thường.

2. **Breadth** — số loại ticket (`category`) khác nhau mà nhân viên giải quyết có dùng gợi ý trong một khoảng đo. Vì copilot hữu ích khi dùng được ở nhiều loại ticket, không chỉ một loại lặp.

**Không chọn frequency** vì cadence đã được xác định là phản ứng theo sự kiện với nhiều lần xử lý trong ngày; retention đã đo mức độ quay lại theo số ngày làm việc trong tuần.

### 3.3 North Star + leading + counter

**North Star Metric** = unit of value + quality threshold + frequency

| Thành phần | Nội dung |
|---|---|
| Unit of value | Một ticket được giải quyết có dùng gợi ý (`ticket_resolved` có `suggestion_id`) |
| Quality threshold | Không bị mở lại và không bị chuyển cấp trong 7 ngày (từ evidence of value ở mục 01) |
| Frequency | Trên mỗi nhân viên hoạt động (có ≥ 1 `ticket_opened` trong tuần), mỗi tuần (khớp nhịp đo hằng tuần ở mục 02) |
| **NSM (viết đầy đủ)** | Số ticket được giải quyết có dùng gợi ý, không bị mở lại và không bị chuyển cấp trong 7 ngày, trên mỗi nhân viên hoạt động trong mỗi tuần. _Rủi ro: số ticket trong tuần phụ thuộc lượng ticket đến chứ không chỉ do sản phẩm (nhất là các tuần tăng mạnh). Có thể chuẩn hoá theo số ticket nhân viên đã mở xử lý (`ticket_opened`), và kiểm bằng counter-metric. Vì quality threshold cần 7 ngày, NSM của tuần N chỉ chốt hẳn sau khi hết 7 ngày kể từ ngày resolved cuối cùng của tuần đó._ |

**Leading indicators** (tối đa 3, mỗi cái kèm "vì sao tin nó dự báo được core action lặp lại"):

| # | Indicator | Vì sao dự báo core action lặp lại |
|---|---|---|
| 1 | Tỉ lệ nhân viên đạt activation trong 7 ngày kể từ `copilot_enabled` | Nhân viên chạm được value sớm thường có lý do dùng lại ở lần sau |
| 2 | Tỉ lệ ticket được mở xử lý mà có `suggestion_id` trên câu trả lời đã gửi (`ticket_opened` và `reply_sent`) | Dùng gợi ý như một phần thói quen xử lý ticket, không chỉ dùng một lần |
| 3 | Số loại ticket (category) được giải quyết bằng gợi ý (breadth, mục 3.2) | Dùng cho nhiều loại ticket nghĩa là copilot hữu ích rộng, nên có nhiều dịp để dùng lại |

**Counter-metric** (ít nhất 1: core action tăng nhưng cái gì không được xấu đi?):

| Counter-metric | Bảo vệ điều gì |
|---|---|
| CSAT của các ticket giải quyết có `suggestion_id`, không thấp hơn nhóm ticket không dùng gợi ý | Trải nghiệm của khách. Bắt được trường hợp khách bỏ đi lặng lẽ (không mở lại nhưng không hài lòng), điểm yếu đã nêu ở tiêu chí 4 Phase 1 |
| Chi phí AI mỗi ticket được giải quyết | Kinh tế của sản phẩm. Ngăn việc core action tăng bằng cách gọi AI nhiều hơn mức cần |

_Đã cân nhắc nhưng bỏ khỏi bảng: tỉ lệ câu trả lời sai chính sách (QA lấy mẫu), vì không có event nào tính được. Có thể giữ làm kiểm tra ngoài bảng._

---

## 04 — Retention Definition (Phase 3)

| Thành phần | Câu hỏi | Nội dung |
|---|---|---|
| Unit | User, account, team, organization hay object? | Nhân viên CSKH (cùng cấp đo với kết luận cadence ở mục 02) |
| Cohort entry | Event nào đưa unit vào cohort? | `ticket_resolved` đầu tiên có `suggestion_id` (chính là activation event), chia cohort theo tuần nhân viên đạt activation |
| Return event | Core action / value event nào phải lặp lại? | `ticket_resolved` có `suggestion_id` (đúng completion rule ở mục 01). Chất lượng (không mở lại / không chuyển cấp) tính ở North Star, không chờ 7 ngày trong retention |
| Window | Daily, weekly, monthly, project-based hay custom bracket? | Hằng tuần, tính ở tuần thứ 1, 2 và 4 sau tuần vào cohort, khớp nhịp đo hằng tuần ở mục 02 (giả định A5) |
| Threshold | Một lần hay nhiều lần trong window? | Có ít nhất một ticket được giải quyết kèm `suggestion_id` ở ≥ 3 ngày làm việc trong tuần. Nhân viên xử lý nhiều ticket mỗi ngày nên "một ticket" quá lỏng, đếm theo ngày phản ánh dùng đều đặn (giả định A5) |
| Segment | Retention đang áp dụng cho ai? | Nhân viên đã activated (đạt activation trong 7 ngày kể từ `copilot_enabled`); tách riêng nhân viên mới và nhân viên có kinh nghiệm, không trộn (thâm niên là thuộc tính hồ sơ nhân viên, không phải event) |

**Retention (viết thành một câu đầy đủ):** Tỉ lệ nhân viên CSKH đã activated (cohort theo tuần đạt activation) có ticket được giải quyết kèm `suggestion_id` ở ít nhất 3 ngày làm việc trong tuần thứ 1, 2 và 4 sau tuần vào cohort.

**So với ba mốc** (không so với con số cứng):
- Natural cycle (khớp cadence ở mục 02?): Khớp: nhịp đo hằng tuần, cấp nhân viên, làm mượt dao động giữa các ngày; không dùng D7/D30 chung.
- Cohort đúng segment: So retention giữa các cohort cùng loại (nhân viên mới với mới, có kinh nghiệm với có kinh nghiệm).
- Benchmark category: So với benchmark retention của category công cụ nội bộ/B2B hỗ trợ nhân viên (cohort theo tuần), không dùng benchmark của app tiêu dùng và không so với con số cứng.

### ✅ GATE 3 — Metric tính được, retention đủ nghĩa
- [x] Activation có start/activation event + time window
- [x] Retention đủ 6 thành phần và khớp cadence
- [x] NSM đúng công thức 3 thành phần
- [x] Có ít nhất 1 counter-metric (giữ cả CSAT và chi phí AI)

---

## 05 — Product Loop (Phase 4)

### Loop hai chu kỳ

| Bước | Chu kỳ 1 | Chu kỳ 2 |
|---|---|---|
| Natural trigger | Khách gửi yêu cầu → ticket mới được giao vào hàng đợi của nhân viên | Khách khác gửi yêu cầu (hoặc ticket cũ bị mở lại) → ticket tiếp theo vào hàng đợi |
| Core action | Nhân viên giải quyết ticket, câu trả lời gửi đi có `suggestion_id` | Nhân viên giải quyết ticket tiếp theo, lại dùng gợi ý trong cùng màn hình ticket |
| Immediate value | Ticket xong nhanh, đúng chính sách; hàng đợi giảm một ticket | Như chu kỳ 1; nhanh hơn nếu gợi ý khớp loại ticket (giả thuyết, được kiểm bằng metric hypothesis bên dưới) |
| Saved state / investment | Ticket = resolved; lịch sử hội thoại; câu trả lời gắn `suggestion_id`. | Thêm một ticket đã xử lý bằng gợi ý; nhân viên đã có trải nghiệm trực tiếp về chất lượng gợi ý ở chu kỳ trước, là cơ sở để dùng lại hoặc bỏ |
| Next natural trigger | Ticket kế tiếp được giao, không do notification của copilot | Ticket kế tiếp hoặc đầu ca làm việc tiếp theo |

**Loại loop chính:** event-response (khớp với dạng hành vi "phản ứng theo sự kiện" ở mục 02; đã cân nhắc habit · progress · project · workflow · transaction · account-level)

**Reason to return nếu bỏ notification đi là gì?**

Nhân viên quay lại dùng copilot vì sử dụng gợi ý giúp họ soạn và gửi phản hồi nhanh hơn so với tự viết từ đầu. Đây là lợi ích trực tiếp của core action, không phụ thuộc vào notification.

**Metric hypothesis:**

Nếu loop hoạt động, trong 4 tuần đầu sau khi nhân viên bật copilot, **median thời gian xử lý chủ động mỗi ticket** có sử dụng gợi ý ở tuần 4 sẽ thấp hơn tuần 1 ít nhất 10%.

**Cách đo:** với mỗi tuần, tính median thời gian từ `ticket_opened` đến `reply_sent` đầu tiên có `suggestion_id`, chỉ trên các ticket đủ điều kiện đo (ticket có `ticket_opened` và có `reply_sent` đầu tiên kèm `suggestion_id` trong cùng ticket). So sánh tuần 1 với tuần 4 ở cùng cấp nhân viên.

**Vì sao:** sau nhiều lần sử dụng, nhân viên có thể quen với cách đọc, chỉnh và sử dụng gợi ý, nên cần ít thời gian hơn để tạo phản hồi.

**Điều kiện diễn giải:** hypothesis không yêu cầu median phải giảm đều qua cả 4 tuần; kết luận chính là so sánh tuần 4 với tuần 1. Nếu volume ticket hoặc cơ cấu `category` thay đổi đáng kể giữa hai tuần, kết quả cần được đọc cùng với breadth và volume ticket.

---

## 06 — Tracking nhanh (Phase 4)

### Core events (4–8 event; event đại diện điều **đã xảy ra**, không phải ý định)

| Tên event (`object_action`) | Ý nghĩa | Thời điểm ghi nhận | Metric sử dụng (Phase 3) |
|---|---|---|---|
| `copilot_enabled` | Copilot được bật cho nhân viên lần đầu | Khi tài khoản nhân viên được cấp quyền dùng copilot (phía server, một lần mỗi nhân viên) | Activation (start event) |
| `ticket_opened` | Nhân viên bắt đầu xử lý một ticket | Lần đầu nhân viên mở ticket sau khi được giao (không ghi lại khi tải lại trang) | Engagement – depth (mốc bắt đầu tính thời gian xử lý) |
| `suggestion_generated` | Copilot đã tạo một gợi ý cho ticket (kèm `suggestion_id`, chi phí) | Khi hệ thống tạo xong gợi ý, không phải lúc nhân viên yêu cầu | Counter-metric – chi phí AI mỗi ticket |
| `reply_sent` | Một câu trả lời đã gửi thành công cho khách (kèm `suggestion_id` nếu có chèn gợi ý) | Khi hệ thống xác nhận gửi xong, không phải lúc bấm Gửi | Activation, core action (cờ `suggestion_id` của completion rule), depth (mốc kết thúc) |
| `ticket_resolved` | Nhân viên giải quyết xong ticket (kèm `category`, `ai_suggestion_used`) | Khi trạng thái ticket thật sự chuyển sang resolved do nhân viên; không tính tự đóng do hết hạn | Activation (activation event), core action, breadth (theo `category`) |
| `ticket_reopened` | Ticket đã resolved bị mở lại | Khi trạng thái chuyển từ resolved về mở, trong 7 ngày kể từ lúc resolved | Evidence of value (bộ lọc chất lượng của core action); NSM – quality threshold |
| `ticket_escalated` | Ticket bị chuyển cấp sau khi nhân viên đã gửi câu trả lời | Khi ticket thật sự được chuyển sang cấp xử lý khác, trong 7 ngày kể từ lúc resolved | Evidence of value (bộ lọc chất lượng của core action); NSM – quality threshold |
| `csat_submitted` | Khách gửi điểm hài lòng cho ticket | Khi khách gửi điểm thành công | Counter-metric – CSAT |

_Quy ước thuộc tính: `ai_suggestion_used` trên `ticket_resolved` = true khi ít nhất một `reply_sent` của ticket đó có `suggestion_id`. Ở các mục trên, “`ticket_resolved` có / kèm `suggestion_id`” nghĩa là `ticket_resolved` với `ai_suggestion_used` = true._

### Kiểm tra hai chiều: metric nào có event để tính

| Metric | Event dùng để tính |
|---|---|
| Activation | `copilot_enabled`, `reply_sent`, `ticket_resolved` |
| Engagement – depth | `ticket_opened`, `reply_sent` |
| Engagement – breadth | `ticket_resolved` (thuộc tính `category`) |
| Counter-metric – CSAT | `csat_submitted` |
| Counter-metric – chi phí AI | `suggestion_generated`, `ticket_resolved` |
| Retention | `copilot_enabled`, `ticket_resolved` (kèm `suggestion_id`) |
| North Star | `ticket_resolved`, `ticket_reopened`, `ticket_escalated`, `ticket_opened` (chuẩn hoá, xác định nhân viên hoạt động) |
| Leading indicators | `copilot_enabled`, `ticket_opened`, `reply_sent`, `ticket_resolved` |

_Lưu ý: nhịp đo hằng tuần chỉ cần dấu thời gian của các event, không cần dữ liệu ngoài. Hai counter-metric (CSAT, chi phí AI) cần giữ cả `csat_submitted` và `suggestion_generated`._

### Tiêu chí nghiệm thu (ít nhất 2; ưu tiên hai bẫy phổ biến)

1. Event chỉ xuất hiện khi hành vi thật sự hoàn tất (không bắn khi mới bấm nút): `ticket_resolved` chỉ ghi khi trạng thái ticket đã chuyển sang resolved thành công (bấm "Resolve" nhưng lưu thất bại thì không ghi, tự đóng do hết hạn thì không ghi). `reply_sent` chỉ ghi khi hệ thống xác nhận câu trả lời đã gửi cho khách.
2. Reload / retry / autosave không ghi trùng cùng một hành vi: với mỗi cặp `user_id` và `ticket_id`, `ticket_resolved` chỉ ghi một lần cho mỗi lần chuyển trạng thái từ chưa resolved sang resolved. Tải lại trang, retry hay autosave không tạo thêm event cho cùng một lần chuyển. Nếu ticket bị mở lại rồi resolved lần nữa thì là một lần chuyển mới, và activation chỉ tính lần đầu tiên.

### ✅ GATE 4 — Loop nối metric, event nối loop
- [x] Loop ≥ 2 chu kỳ và có metric hypothesis trỏ về một metric ở Phase 3 _(trỏ về depth, mục 3.2)_
- [x] Mọi event trong bảng đều map được về ít nhất một metric

---

## Tự soi lỗi trước khi nộp (Phase 5)

- [x] Core action không phải thao tác giao diện hay output hệ thống? — Đạt: là việc giải quyết ticket (trạng thái `resolved` + cờ `suggestion_id`), không phải mở app hay hỏi AI.
- [x] Activation không phải "xem hết hướng dẫn" hay "đăng nhập"? — Đạt: activation là `ticket_resolved` đầu tiên có `suggestion_id`; `copilot_enabled` chỉ là start event.
- [x] Frequency không cao hơn nhu cầu thật? — Đạt: core action là phản ứng theo sự kiện, được kích hoạt khi có ticket mới và có thể xảy ra nhiều lần trong ngày. Retention ≥ 3 ngày làm việc/tuần là tiêu chí đo mức độ quay lại trong window tuần, không phải yêu cầu nhân viên phải thực hiện core action với một tần suất nhân tạo mỗi ngày.
- [x] Loop có reason to return ngoài notification? — Đạt: nhân viên có lý do quay lại vì dùng gợi ý giúp soạn và gửi phản hồi nhanh hơn so với tự viết từ đầu. Ticket mới là trigger của loop, không phải reason to return.
- [x] Retention không dùng chung một window cho mọi cadence? — Đạt: retention được tính theo tuần 1, 2 và 4 sau tuần vào cohort, khớp với nhịp đo hằng tuần; không dùng D7/D30 chung cho mọi hành vi.
- [x] Mọi event đều map về một metric? — Đạt: cả 8 event đều có metric ở bảng mục 06, gồm `csat_submitted` và `suggestion_generated` cho hai counter-metric.
- [x] Metric nào cũng có event để tính nó? — Đạt sau khi sửa: bỏ ứng viên counter-metric QA lấy mẫu (không có event); NSM chuẩn hoá bằng `ticket_opened` có sẵn nên không cần thêm event; Depth dùng `ticket_opened` và `reply_sent`; nhịp đo hằng tuần chỉ cần dấu thời gian của event.

> **Phát hiện thêm ngoài 7 câu (đã sửa):** Depth từng đo đến `ticket_resolved` nên cộng cả thời gian chờ khách. Đã đổi thành từ `ticket_opened` đến `reply_sent` đầu tiên có `suggestion_id`.

### Revision log

- Thêm cờ `suggestion_id` vào completion rule (mục 01): tiêu chí 3 "có thể quan sát" ban đầu ✗ vì không nhận biết được "có dùng gợi ý" khi nhân viên sửa nhiều câu trả lời.
- Đổi dạng hành vi ở mục 02 từ "thói quen thường xuyên" sang "phản ứng theo sự kiện": core action được kích hoạt bởi ticket do khách gửi và lặp lại khi có ticket tiếp theo.
- Đổi nhịp đo ở mục 02 sang hằng tuần ở cấp nhân viên: tần suất tự nhiên của core action vẫn là nhiều lần trong ngày, nhưng retention được đánh giá theo số ngày làm việc có dùng core action trong tuần và North Star được tổng hợp theo tuần.
- Đổi định nghĩa Depth (mục 3.2) thành median thời gian từ `ticket_opened` đến `reply_sent` đầu tiên có `suggestion_id`: không tính thời gian chờ khách phản hồi và không sử dụng nhóm không dùng gợi ý làm mẫu so sánh vì metric hiện tại không tính được nhóm đó.
- Bỏ ứng viên counter-metric "sai chính sách (QA lấy mẫu)" khỏi mục 3.3: không có event nào tính được nó.
- Đổi metric hypothesis thành so sánh median Depth giữa tuần 1 và tuần 4 sau khi nhân viên bật copilot; không yêu cầu metric phải giảm đều qua từng tuần.
- Thống nhất cohort retention theo tuần đạt activation: cohort entry là activation event nên cohort được tính theo tuần nhân viên đạt activation.
- Thêm quy ước thuộc tính ở mục 06 (`ai_suggestion_used` trên `ticket_resolved` = có `reply_sent` mang `suggestion_id`): các mục trước dùng hai cách gọi cho cùng một điều kiện.

### ✅ GATE 5 — Bài sạch lỗi kinh điển
- [x] Đối chiếu xong 7 câu tự soi và sửa mọi chỗ mắc — hoặc ghi rõ vì sao giữ nguyên một lựa chọn "phá rule".

---

## AI Support Log

Xem [ai-support-log.md](./ai-support-log.md).
