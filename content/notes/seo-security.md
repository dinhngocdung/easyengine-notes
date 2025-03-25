---
title: Tips Security & SEO with EasyEngine
linkTitle: Security & SEO
weight: 9
type: docs
prev: seo-security
next: borgmatic
---

Security and Optimization Measures for My System  

## Securing `xmlrpc.php`  

The `xmlrpc.php` file is still necessary for WooCommerce functions, but it is often targeted by attacks. Even when not dangerous, it can generate unnecessary load on the server. My solution is to allow only Jetpack’s IPs to access this endpoint.  

Edit the `user.conf` file:  

```bash
nano /opt/easyengine/sites/sample.com/config/nginx/custom/user.conf
```

Add the following lines:  

```bash
location = /xmlrpc.php {
		# Allow Jetpack IPs
		allow 122.248.245.244;
		allow 54.217.201.243;
		allow 54.232.116.4;
		allow 192.0.80.0/20;
		allow 192.0.96.0/20;
		allow 192.0.112.0/20;
		allow 195.234.108.0/22;
		allow 192.0.64.0/18;
	
		# Block all other IPs
		deny all;
	
		# Return a 404 page for unauthorized access to xmlrpc.php
		try_files $uri =404;
	}
```

Reload the Nginx configuration for the site:  

```bash
ee site reload sample.com 
```

## Securing SSH  

Close port 22 and allow only your own IP to access SSH. Remember to replace `your_IP` with your actual IP address.  

```bash
	nft add rule ip filter INPUT ip saddr 123.123.123.123 tcp dport 22 accept
	nft add rule ip filter INPUT tcp dport 22 drop
```

If your IP address changes, add the new IP (e.g., `124.124.124.124`):  

```bash
nft insert rule ip filter INPUT ip saddr 124.124.124.124 tcp dport 22 accept
```

Remove old IPs from the whitelist if they are no longer in use:  

```bash
# List handles
nft -a list chain ip filter INPUT 
# Identify the rule handle for your IP, e.g., 1
nft delete rule ip filter INPUT handle 1 
```

## Caching Pages with Google and Facebook Query Strings  

A common issue is that even when EasyEngine Redis cache works well, Google ads often append query strings to URLs, causing:  

1. Visitors from Google not being served cached pages because the URL + query string is always changing.  
2. Redis storing separate cache entries for each query string variation, leading to excessive duplicate cache data.  

My solution is to make Redis ignore specific query strings like `fbclid=`, `_gl=`, `gclid=`, `gad_source=`, `dclid=`, `wbraid=`, `gbraid=`, `gclsrc=`, etc. Redis will check if a cached version of the base URL exists—if it does, it serves that cache; if not, it saves the cache using the base URL as the key.  

To implement this, modify the `main.conf` file. Note that these changes may be lost when updating EasyEngine (`ee cli update`), so you should keep a backup of the modifications.  

```bash
nano /opt/easyengine/sites/sample.com/config/nginx/conf.d/main.conf
```

Modify the relevant sections and replace `sample.com` with your domain:  

```yaml
# Ignore specific tracking query parameters from Google, Facebook, and other ad platforms
if ($query_string !~* "(^$|srsltid=|utm_.*=|fbclid=|_gl=|gclid=|gad_source=|dclid=|wbraid=|gbraid=|gclsrc=)") {
	set $skip 1;
}

# Create a clean version of the URL without query strings
set $clean_uri $request_uri;
if ($request_uri ~ "^(.*)\\?.*$") {
	set $clean_uri $1;
}

location /redis-store {
        internal;
        set_unescape_uri $key $arg_key;
        redis2_query set $key $echo_request_body;
        # Adjust cache duration if needed, e.g., increase to 7 days
        redis2_query expire $key 604800;
        redis2_pass redis;
}

# Set cache key based on the cleaned URL
set $key "sample.com_page:http$request_method$host$clean_uri"; 
# Update key if using HTTPS
if ($HTTP_X_FORWARDED_PROTO = "https") {
	set $key "sample.com_page:https$request_method$host$clean_uri";
}
```