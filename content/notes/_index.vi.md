---
title: Ghi chép về EasyEngine 4
next: review
---
{{< cards >}}
  {{< card link="review" title="Đánh giá EasyEngine 4" icon="star" >}}
  {{< card link="differences" title="Chuyển qua EasyEngine" icon="switch-horizontal" >}}
  {{< card link="deploying" title="Cài đặt với WordPress" icon="cursor-click" >}}
  {{< card link="cache" title="Redis Cache" icon="trending-up" >}}
  {{< card link="fail2ban" title="Fail2ban Docker" icon="shield-check" >}}
  {{< card link="cloudflare" title="Cloudflare với Fail2Ban" icon="globe" >}}
  {{< card link="borgmatic" title="Sao lưu Borgmatic" icon="cloud-upload" >}}
  {{< card link="migrate" title="Chuyển server mới" icon="truck" >}}
  {{< card link="development-stage-production" title="Development, Stage, và Production" icon="beaker" >}}
  {{< card link="easyengine-docker" title="Esyengine-Docker" icon="cube" >}}

{{< /cards >}}

Tôi kinh ngạc trước sự thuận tiện và hiệu suất của EasyEngine 4, và nhận thấy rằng nó bị đánh giá thấp một cách bất công dù đội ngũ phát triển đã nỗ lực không nhỏ, chỉ vì nó sử dụng Docker — một lựa chọn tiến bộ. Đó là lý do tôi viết ghi chú này, giúp những người chưa quen với Docker có thể triển khai và hưởng lợi từ một dự án cộng đồng tuyệt vời. Đồng thời, điều này cũng có thể góp phần giúp đội ngũ EasyEngine tiếp tục hành trình đáng tự hào của họ.

Đây là những ghi chú tôi đã ghi lại trong quá trình lựa chọn, tìm hiểu, triển khai và khắc phục sự cố với EasyEngine 4. Tôi muốn công khai chúng với một chút chỉnh sửa phù hợp để người khác tham khảo. Tôi tin rằng chúng sẽ giúp những ai có hành trình giống tôi tìm được câu trả lời nhanh hơn nhiều lần, giúp quá trình này trở nên thật dễ dàng.

Nếu bạn đang tìm hiểu cách xây dựng webserver đầu tiên của mình, tìm kiếm một phương pháp tốt hơn để quản lý webserver hiện tại, hoặc đã nghe nói về EasyEngine 4 nhưng vẫn còn băn khoăn, thì bạn chính là người sẽ hưởng lợi nhiều nhất từ ghi chú này.

Đừng lo lắng về những cảnh báo liên quan đến Docker hay sự phức tạp của nó. Tôi cũng không biết gì về Docker trước khi sử dụng EasyEngine 4. Nhưng đội ngũ phát triển đã làm được điều phi thường: bạn không cần biết về Docker vẫn có thể vận hành webserver Docker hóa với EasyEngine.

## Bạn có câu hỏi ?

{{< callout emoji="❓" >}}
  Luôn có những vấn đề khó với người này những dễ với người khác.
  Nếu bạn có vấn đề cần hỗ trợ, hãy dùng diễn đàn chính thức của EasyEngine [Đặt câu hỏi](https://github.com/EasyEngine/easyengine/discussions)!
{{< /callout >}}

## Bắt đầu

Bước đầu tiên của khám phá

{{< cards >}}
  {{< card link="review" title="Đánh giá EasyEngine 4" icon="document-text" subtitle="Lý do bạn không nên chần chừ bắt đầu với EasyEnginee" >}}
{{< /cards >}}
