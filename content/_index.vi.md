---
title: Ghi chép của Dũng về EasyEngine 4
layout: hextra-home
---
{{< hextra/hero-badge link="https://github.com/EasyEngine/easyengine/discussions">}}
  <div class="hx-w-2 hx-h-2 hx-rounded-full hx-bg-primary-400"></div>
  <span>Đặt câu hỏi về EasyEngine</span>
  {{< icon name="arrow-circle-right" attributes="height=14" >}}
{{< /hextra/hero-badge >}}

{{< hextra/hero-container
  image="/images/easyeengine-docker-gray.svg"
  imageClass="hx-block hx-overflow-hidden"
  imageWidth="400" imageHeight="300"
  imageTitle="AXIVO"
>}}

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-headline >}}
  Ghi chép của Dũng&nbsp;<br class="sm:hx-block hx-hidden" />về EasyEngine 4
{{< /hextra/hero-headline >}}
</div>

<div class="hx-mb-12">
{{< hextra/hero-subtitle >}}
  Kinh nghiệm tôi đã xây dựng một webserver&nbsp;<br class="sm:hx-block hx-hidden" />nhanh, hiệu xuất, tin cậy và hiện đại.
{{< /hextra/hero-subtitle >}}
</div>

{{< /hextra/hero-container >}}

<div class="hx-mb-6">
{{< hextra/hero-button text="Easyengine-Docker" link="notes/easyengine-docker" >}}
{{< hextra/hero-button text="Đọc Notes" link="notes" >}}
</div>

<div class="hx-mt-6"></div>

{{< hextra/feature-grid >}}
  {{< hextra/feature-card
    title="Đánh giá về EasyEngine 4"
    subtitle="Tôi thực sự đã dựng được một webserver nhanh nhất từng có."
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-md:hx-min-h-[340px]"
    image="/images/easyengine-server.svg"
    imageClass="hx-top-[40%] hx-left-[24px] hx-w-[180%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 50% 80%,rgba(194,97,254,0.15),hsla(0,0%,100%,0));"
    link="notes/review"
  >}}
  {{< hextra/feature-card
    title="Những khác biệt khi Docker hoá"
    subtitle="So sánh những khác biệt khi bạn chuyển đến EasyEngine từ LEMPstack thông thường."
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-lg:hx-min-h-[340px]"
    image="/images/easyengine-structure.svg"
    imageClass="hx-top-[40%] hx-left-[36px] hx-w-[180%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 50% 80%,rgba(142,53,74,0.15),hsla(0,0%,100%,0));"
    link="notes/differences"
  >}}
  {{< hextra/feature-card
    title="Triển khai Wordpress trên EasyEngine"
    subtitle="Các bước đơn giản triển khai và quản lý vòng đời site Wordpress."
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-md:hx-min-h-[340px]"
    image="/images/proxy-cache-easyengine.svg"
    imageClass="hx-top-[40%] hx-left-[36px] hx-w-[110%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 50% 80%,rgba(221,210,59,0.15),hsla(0,0%,100%,0));"
    link="notes/deploying"
  >}}
  {{< hextra/feature-card
    title="Lỗi database vì binlog"
    subtitle="File dữ liệu nhật ký Binlog làm cạn bộ nhớ server, gây lỗi Database Connection, một tình huống rất phổ biến dễ nản lòng nhưng khắc phục dễ dàng."
    link="notes/binlog"
  >}}
  {{< hextra/feature-card
    title="SSL và Redirect"
    subtitle="Cài đặt SSL dễ dàng và cách thiết lập Redirect trong hệ thống nginx của EasyEngine."
    link="notes/ssl-redirect"
  >}}
  {{< hextra/feature-card
    title="Cache mọi cấp độ với Redis"
    subtitle="Giải pháp cache duy nhất nhưng cự kì hiệu quả, với tất cả cấp độ từ full-pages-cache, object-cache lên đến proxy-cache."
    link="notes/cache"
  >}}
  {{< hextra/feature-card
    title="Fail2ban Docker"
    subtitle="Tăng cường bảo mật với Fail2ban, và những đặc biệt của riêng hệ thống docker hoá."
    link="notes/fail2ban"
  >}}
  {{< hextra/feature-card
    title="Và vài điều khác..."
    icon="sparkles"
    subtitle="Backup tự động với Borgmatic, cấu hình với Cloudflare, các thủ thuật nhỏ can thiệp vào nginx tốt cho SEO."
    link="notes"
  >}}
{{< /hextra/feature-grid >}}

