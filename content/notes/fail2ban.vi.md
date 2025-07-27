---
title: "Fail2ban Docker: bảo mật tăng cường"
linkTitle: Fail2ban Docker
weight: 7
type: docs
prev: cache
next: cloudflare
---

Bảo mật luôn làm các bạn lo lắng khi tự triển khai web server. Với EasyEngine, đây là một việc nhẹ nhõm hơn nhiều vì bản thân Docker cách ly rất tốt, phần mềm luôn mới vì dễ cập nhật, nên bản thân nó đã là một hệ thống an toàn.

EasyEngine cũng đã thiết lập một số bảo mật, như thiết lập `time-request-limit` với `wp-login.php` để tránh tấn công brute-force.

Nhưng có nhiều trường hợp bạn muốn bảo vệ hoặc giảm tải server, bạn sẽ cần dùng Fail2Ban, phần mềm "mặc định" cho việc bảo vệ server. Và qua đây, bạn sẽ học cách tự triển khai phần mềm trên hệ thống server Docker hóa.

Hướng dẫn này mặc định là bạn chạy Debian 12, với tường lửa mặc định `nftables`.

## Fail2Ban là gì

Fail2Ban là một công cụ bảo mật giúp bảo vệ hệ thống Linux khỏi các cuộc tấn công brute-force hoặc truy cập trái phép thông qua việc phân tích các log file và chặn địa chỉ IP đáng ngờ. Khi được sử dụng cùng Docker, Fail2Ban có thể bảo vệ cả server (Debian) và các container Docker đang chạy trên nó.

Fail2Ban hoạt động dựa trên ba bước chính:

1. **Giám sát nhật ký**: Công cụ quét các tệp nhật ký của hệ thống hoặc dịch vụ (ví dụ: nhật ký SSH, web server) để tìm các mẫu (patterns) (dựa theo filter) cho thấy hành vi bất thường, chẳng hạn như nhiều lần đăng nhập thất bại từ một địa chỉ IP.
2. **Phát hiện vi phạm**: Khi số lần thất bại vượt quá ngưỡng được định nghĩa trước trong một khoảng thời gian cụ thể (ví dụ: 5 lần thất bại trong 10 phút), Fail2Ban xác định đó là hành vi đáng ngờ.
3. **Áp dụng quy tắc cấm**: Fail2Ban sau đó thêm quy tắc vào nftables để chặn địa chỉ IP đó trong một khoảng thời gian nhất định. Đối với container Docker, quy tắc này được thêm vào chuỗi `DOCKER-USER` của nftables, đảm bảo rằng lưu lượng từ IP bị chặn không thể đến container.

![Fail2ban Flow](/images/fail2ban-docker.svg)

Hoạt động của Fail2Ban dựa trên ba thành phần chính: **Jail**, **Filter** và **Action**:

- **Jail:** Jail trong Fail2Ban là nơi định nghĩa dịch vụ cần bảo vệ. Mỗi jail sẽ liên kết với một filter để xác định hành vi đáng ngờ và một action để quyết định cách xử lý. Ngoài ra, jail cũng chỉ định logfile mà Fail2Ban cần giám sát để phát hiện các hành vi vi phạm.
- **Filter:** Filter chứa các biểu thức chính quy (regex) giúp phân tích logfile và phát hiện các yêu cầu hoặc hành động bất thường. Khi một mục nhập trong logfile khớp với regex trong filter, Fail2Ban sẽ xem đó là một hành vi vi phạm và báo cho jail xử lý.
- **Action:** Action xác định cách Fail2Ban phản ứng khi phát hiện vi phạm. Các hành động phổ biến bao gồm chặn IP bằng iptables hoặc nftables, ghi log cảnh báo hoặc gửi email thông báo. Action giúp ngăn chặn các cuộc tấn công tiếp diễn và bảo vệ hệ thống hiệu quả.

## Cài đặt Fail2Ban với Docker

Nguyên tắc Docker hóa là can thiệp cực kỳ tối thiểu vào server host, tất cả đều được chạy trong container. Và ở đây, chúng ta cũng làm như vậy khi cài đặt Fail2Ban trong container, và image được chọn là `crazymax/fail2ban`.

Các bước tiến hành:

1. Chuẩn bị file `docker-compose.yml`, `.env`: chỉ thị cách Docker vận hành.
2. Thiết lập jail, action, filter.
3. Chạy Fail2Ban với Docker.


{{% steps %}}

### Tạo thư mục và file xây dựng docker fail2ban
Lập thư mục chứa Fail2Ban Docker, tạo file `docker-compose.yml`, file chỉ thị cho Docker xây dựng Fail2Ban container:

```bash
# Tạo các thư mục cura fail2ban
mkdir -p /opt/fail2ban/data/{action.d,filter.d,jail.d,db}

# Downaload docker-compose.yml, .env
curl -o /opt/fail2ban/docker-compose.yml -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/docker-compose.yml
curl -o /opt/fail2ban/.env -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/.env
```
### Áp dụng Jail, filter, action

Triển khai bảo vệ 2 vị trí dễ tổng thương nhất của webserver: **ssh** và **wp-login** 
```bash
curl -o /opt/fail2ban/data/filter.d/sshd.local -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/filter.d/sshd.local
curl -o /opt/fail2ban/data/filter.d/wp-login-fail.conf -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/filter.d/wp-login-fail.conf

curl -o /opt/fail2ban/data/jail.d/jail.local -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/jail.d/jail.local
```

> [!TIP]
> Đặc biệt chú ý cài đặt `chain = DOCKER-USER`, Fail2Ban sẽ chèn lệnh cấm vào chain này để có tác dụng trong hệ thống Docker hóa.  

Nếu có sử dụng Cloudflare, và muốn cấm ngay tại Cloudflare WAF, cần bổ xung thêm action cloudflare:
```bash
curl -o /opt/fail2ban/data/jail.d/jail-cloudflare.local -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/jail.d/jail-cloudflare.local

# Hãy thay đổi chính xác cfzone và cftoken của bạn
vi /opt/fail2ban/data/jail.d/jail-cloudflare.local
```

{{% /steps %}}

## Vận hành Fail2Ban Docker

Như vậy, với các file đã chuẩn bị, chúng ta đã sẵn sàng vận hành Fail2Ban Docker.

```bash
/opt/fail2ban/
├── docker-compose.yml
├── .env
└── data/
    ├── jail.d/
    │   └── jail.local
    └── filter.d/
        ├── sshd.local
        └── wp-login-fail.conf
```

Chạy Fail2Ban Docker
Đảm bảo bạn đang ở trong thư mục `/opt/fail2ban` để chạy các lệnh `docker compose`:

```bash
cd /opt/fail2ban
```

Chạy Fail2Ban Docker:
```bash
# Chạy nền Fail2Ban Docker
sudo docker compose up -d 
```

Xem nhật ký hoạt động Fail2Ban
```bash
sudo docker compose logs -f
```

Xem các IP đang bị chặn trên tất cả các jail
```bash
sudo docker compose exec fail2ban fail2ban-client status --all
```

Unban một IP
Đôi khi Fail2Ban chặn nhầm, ví dụ: unban IP `123.123.123.123` trong jail `wp-login-fail`:

```bash
sudo docker compose exec fail2ban fail2ban-client set nginx-errors unbanip 123.123.123.123
```


## Whitelist

Đôi khi, các jail khắt khe từ Fail2Ban sẽ chặn các truy xuất quan trọng từ bot của Google, ChatGPT và Facebook. Giải pháp của chúng ta là đưa các IP được công bố của Google, ChatGPT, Facebook… và tất cả các dịch vụ cần thiết cho website của bạn vào whitelist của Fail2Ban.  

Whitelist Fail2Ban được tạo bằng cách đưa các IP vào `ignoreip` trong phần `[DEFAULT]` của file `jail.local`. Đownload file sẵn có của tôi, và chỉnh sửa theo nhu cầu

```bash
curl -o ./data/jail.d/jail-whitelist.local -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/jail.d/jail-whitelist.local

```


Dưới đây là danh sách các IP mà tôi khuyến khích thêm vào whitelist:

1. **IP Home**: 123.123.123.123  
2. **IP Server Internal**: 127.0.0.1/8 ::1  
3. **IP Services Global-Nginx-Proxy_1**: 10.1.0.3  
4. [IP Cloudflare](https://www.cloudflare.com/ips/)  
5. [IP Google Bot](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
6. [IP Google Special-Crawlers](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
7. [IP Google User-Triggered-Fetchers](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
8. [IP Google Other](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
9. [IP Commoncrawl.org](https://commoncrawl.org/faq)  
10. [IP Bing Bot](https://searchengineland.com/microsoft-list-of-bingbot-ip-addresses-released-376039)  
11. [IP ChatGPT Bot](https://platform.openai.com/docs/bots)  

### Tham khảo:
[GitHub: Crazy-Max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban)  
