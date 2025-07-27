---
title: "Fail2ban Docker: Enhancing Security"
linkTitle: Fail2ban Docker 
weight: 7
type: docs
prev: cache
next: cloudflare
---

Security is always a concern when deploying a web server. With EasyEngine, this process becomes much simpler since Docker provides strong isolation, software updates are easy to apply, and the system remains inherently secure.

EasyEngine also includes some built-in security measures, such as setting `time-request-limit` for `wp-login.php` to prevent brute-force attacks.

However, in cases where you want to enhance protection or reduce server load, you’ll need Fail2Ban, the "default" software for server protection. This guide will teach you how to deploy Fail2Ban on a Docker-based server.

This guide assumes you are running **Debian 12** with the default firewall **nftables**.

## What is Fail2Ban?

Fail2Ban is a security tool that protects Linux systems from brute-force attacks and unauthorized access by analyzing log files and blocking suspicious IP addresses. When used with Docker, Fail2Ban can safeguard both the **host server (Debian)** and the **Docker containers** running on it.

Fail2Ban operates in three main steps:

1. **Monitoring logs**: It scans system or service log files (e.g., SSH logs, web server logs) to detect suspicious activity patterns (defined by filters), such as multiple failed login attempts from the same IP.
2. **Detecting violations**: When the number of failed attempts exceeds a predefined threshold within a specific time (e.g., 5 failed logins in 10 minutes), Fail2Ban flags the IP as suspicious.
3. **Applying ban rules**: Fail2Ban then adds rules to **nftables** to block the IP for a set period. In a Docker environment, these rules are applied to the **DOCKER-USER** chain in nftables, ensuring that traffic from banned IPs is blocked before reaching the container.

![Fail2ban Flow](/images/fail2ban-docker.svg)

Fail2Ban operates using three key components: **Jail, Filter, and Action**.

- **Jail:** Defines the service that needs protection. Each jail is associated with a filter to detect suspicious behavior and an action to determine how to respond. It also specifies which log files Fail2Ban should monitor.
- **Filter:** Contains regular expressions (regex) used to analyze logs and detect suspicious requests or activities. If a log entry matches the regex, Fail2Ban considers it a violation and alerts the jail.
- **Action:** Determines how Fail2Ban responds to a detected violation. Common actions include blocking the IP using **iptables** or **nftables**, logging warnings, or sending email notifications. Actions help prevent ongoing attacks and protect the system efficiently.

## Installing Fail2Ban with Docker  

The principle of Dockerization is to minimize intervention on the host server—everything runs within a container. Here, we follow the same approach by installing Fail2Ban inside a container using the `crazymax/fail2ban` image.  

**Steps to implement:**

1. Prepare the `docker-compose.yml` and `.env` files, which dictate how Docker operates.  
2. Configure jails, actions, and filters.  
3. Run Fail2Ban with Docker.  

Here's the English translation of your Markdown content, keeping the original Markdown formatting:

{{% steps %}}

### Create Directory and Docker Fail2Ban Build Files

Set up the Fail2Ban Docker directory and create the `docker-compose.yml` file, which instructs Docker to build the Fail2Ban container:

```bash
# Create directories for Fail2Ban
mkdir -p /opt/fail2ban/data/{action.d,filter.d,jail.d,db}

# Download docker-compose.yml, .env
curl -o /opt/fail2ban/docker-compose.yml -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/docker-compose.yml
curl -o /opt/fail2ban/.env -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/.env
```

### Apply Jail, Filter, Action

Deploy protection for the two most vulnerable points of a web server: **SSH** and **wp-login**

```bash
curl -o ./fail22ban/data/filter.d/sshd.local -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/filter.d/sshd.local
curl -o /opt/fail2ban/data/filter.d/wp-login-fail.conf -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/filter.d/wp-login-fail.conf

curl -o /opt/fail2ban/data/jail.d/jail.local -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/jail.d/jail.local
```

> [\!TIP]
> Set special attention to setting `chain = DOCKER-USER`. Fail2Ban will insert ban commands into this chain to take effect within the Dockerized system.

If you use Cloudflare and want to ban directly at Cloudflare WAF, you need to add the Cloudflare action:

```bash
curl -o ./fail22ban/data/jail.d/jail-cloudflare.local -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/jail.d/jail-cloudflare.local

# Please accurately change your cfzone and cftoken
vi /opt/fail2ban/data/jail.d/jail-cloudflare.local
```

{{% /steps %}}

-----

## Operate Fail2Ban Docker

With the prepared files, we are now ready to operate Fail2Ban Docker.

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

Ensure You Are in the `/opt/fail2ban` Directory:
```bash
cd /opt/fail2ban
```

Start Fail2Ban Docker:
```bash
# Run Fail2Ban in the background
sudo docker compose up -d 
```

View Fail2Ban Logs:
```bash
sudo docker compose logs -f
```

Check Banned IPs in All Jails:
```bash
sudo docker compose exec fail2ban fail2ban-client status --all
```

Unban an IP:
Sometimes Fail2Ban may mistakenly block an IP. To unban `123.123.123.123` from the `nginx-errors` jail:
```bash
sudo docker compose exec fail2ban fail2ban-client set nginx-errors unbanip 123.123.123.123
```

## Whitelist

Sometimes, strict Fail2Ban jails may block important access from bots such as Google, ChatGPT, and Facebook. Our solution is to whitelist the published IP addresses of Google, ChatGPT, Facebook, and all necessary services for your website in Fail2Ban.

To whitelist Fail2Ban, add the IP addresses to the `ignoreip` section under `[DEFAULT]` in the `jail.local` file.

For example, to allow Google Bot, add the following lines to the file:

```bash
curl -o ./data/jail.d/jail-whitelist.local -L https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/fail2ban/jail.d/jail-whitelist.local

```

**Recommended IPs to Whitelist**

1. Home IP: 123.123.123.123  
2. Internal Server IP: 127.0.0.1/8 ::1  
3. `services_global-nginx-proxy_1` IP: 10.1.0.3  
4. [Cloudflare IPs](https://www.cloudflare.com/ips/)  
5. [Google Bot IPs](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
6. [Google Special-Crawlers](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
7. [Google User-Triggered Fetchers](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
8. [Google User-Triggered Fetchers - Google](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
9. [Google Other IPs](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)  
10. [Common Crawl](https://commoncrawl.org/faq)  
11. [Bing Bot IPs](https://searchengineland.com/microsoft-list-of-bingbot-ip-addresses-released-376039)  
12. [ChatGPT Bot IPs](https://platform.openai.com/docs/bots)  

### References:

[crazy-max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban)
