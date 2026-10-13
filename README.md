# Track1_Day20_2A202602387_NguyenTienPhat

- **Họ tên:** Nguyễn Tiến Phát
- **MHV:** 2A202602387
- **Dự án chọn làm:** AI Customer Support Agent. Persona là nhân viên CSKH dùng copilot gợi ý câu trả lời để xử lý ticket.
- **Metrics Pack:** [Metrics_Pack.md](./Metrics_Pack.md)
- **AI Support Log:** [ai-support-log.md](./ai-support-log.md)

## Chuỗi quyết định trong Metrics Pack

| Bước | Quyết định |
|---|---|
| Core action | Giải quyết một ticket bằng câu trả lời có dùng gợi ý của copilot (`ticket_resolved` kèm `suggestion_id`), tự kiểm 5/5 tiêu chí |
| Cadence | Phản ứng theo sự kiện, nhiều lần trong ngày; đo hằng tuần ở cấp nhân viên |
| North Star | Số ticket giải quyết có dùng gợi ý, không bị mở lại / chuyển cấp trong 7 ngày, trên mỗi nhân viên hoạt động mỗi tuần |
| Retention | Nhân viên đã activated có ticket giải quyết kèm gợi ý ở ≥ 3 ngày làm việc trong tuần 1, 2, 4 sau tuần vào cohort |
| Loop | Hai chu kỳ event-response; ticket mới là trigger tự nhiên; reason to return là dùng gợi ý giúp soạn và gửi phản hồi nhanh hơn tự viết; hypothesis: median thời gian xử lý chủ động mỗi ticket ở tuần 4 thấp hơn tuần 1 ít nhất 10% |
| Tracking | 8 event, mỗi event map về ít nhất một metric; 2 tiêu chí nghiệm thu |

## Điều tôi mang về áp dụng cho dự án thật

Với AI Customer Support Agent, tôi sẽ không dùng "mở app" hay "hỏi AI" làm core action mà đo việc nhân viên thực sự giải quyết ticket bằng gợi ý. Tôi cũng sẽ tách trigger của hành vi khỏi reason to return: ticket mới là lý do công việc khiến loop bắt đầu, còn giá trị tiết kiệm thời gian mới là lý do nhân viên tiếp tục dùng copilot. Khi triển khai thật, tôi sẽ kiểm tra median thời gian xử lý theo cohort để xem giả thuyết về việc nhân viên quen dần với gợi ý có đúng hay không.
