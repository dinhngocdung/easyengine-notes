---
title: Dũng’s EasyEngine Notes
layout: hextra-home
---
{{< hextra/hero-badge link="https://github.com/EasyEngine/easyengine/discussions">}}
  <div class="hx-w-2 hx-h-2 hx-rounded-full hx-bg-primary-400"></div>
  <span>Discussion about EasyEngine</span>
  {{< icon name="arrow-circle-right" attributes="height=14" >}}
{{< /hextra/hero-badge >}}

{{< hextra/hero-container
  image="/images/easyengine-note-origin-rotate.svg"
  imageClass="hx-block hx-overflow-hidden hx-rounded-3xl"
  imageWidth="300" imageHeight="300"
  imageTitle="AXIVO"
>}}

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-headline >}}
  Dũng's &nbsp;<br class="sm:hx-block hx-hidden" />EasyEngine Notes
{{< /hextra/hero-headline >}}
</div>

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-subtitle >}}
  👋 Welcome to my notes.
{{< /hextra/hero-subtitle >}}
</div>

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-subtitle >}}
  My experience in building a web server
  that is fast, performance, reliable, and modern.
{{< /hextra/hero-subtitle >}}
</div>

{{< /hextra/hero-container >}}


<div class="hx-mb-6">
{{< hextra/hero-button text="Let's start" link="notes" >}}
</div>

<div class="hx-mt-6"></div>

{{< hextra/feature-grid >}}
  {{< hextra/feature-card
    title="Review of EasyEngine in 2025"
    subtitle="I truly built the fastest web server ever."
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-md:hx-min-h-[340px]"
    image="/images/review.png"
    imageClass="hx-top-[40%] hx-left-[24px] hx-w-[180%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 50% 80%,rgba(194,97,254,0.15),hsla(0,0%,100%,0));"
    link="notes/review"
  >}}
  {{< hextra/feature-card
    title="Differences When Dockerizing"
    subtitle="Comparing the differences when switching to EasyEngine from a traditional LEMP stack"
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-lg:hx-min-h-[340px]"
    image="/images/proxycache.png"
    imageClass="hx-top-[40%] hx-left-[36px] hx-w-[180%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 50% 80%,rgba(142,53,74,0.15),hsla(0,0%,100%,0));"
    link="notes/differences"
  >}}
  {{< hextra/feature-card
    title="Deploying WordPress on EasyEngine"
    subtitle="Simple steps to deploy and manage the WordPress site lifecycle."
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-md:hx-min-h-[340px]"
    image="/images/borgmatic-docker-easyengine.svg"
    imageClass="hx-top-[40%] hx-left-[36px] hx-w-[110%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 50% 80%,rgba(221,210,59,0.15),hsla(0,0%,100%,0));"
    link="notes/deploying"
  >}}
  {{< hextra/feature-card
    title="Database Errors Due to Binlog"
    subtitle="Binlog data files exhaust server memory, causing Database Connection errors—a very common, discouraging situation but easily fixable."
    link="notes/binlog"
  >}}
  {{< hextra/feature-card
    title="SSL and Redirect"
    subtitle="Easily set up SSL and configure redirects in EasyEngine’s nginx system."
    link="notes/ssl-redirect"
  >}}
  {{< hextra/feature-card
    title="Caching at All Levels with Redis"
    subtitle="A single yet extremely effective caching solution, covering all levels from full-page cache, object cache, to proxy cache."
    link="notes/cache"
  >}}
  {{< hextra/feature-card
    title="Fail2ban Docker"
    subtitle="Enhancing security with Fail2ban, and the unique aspects of a Dockerized system."
    link="notes/fail2ban"
  >}}
  {{< hextra/feature-card
    title="And a Few More Things..."
    icon="sparkles"
    subtitle="Automated backups with Borgmatic, configuration with Cloudflare, and small nginx tweaks for better SEO."
    link="notes"
  >}}
{{< /hextra/feature-grid >}}



