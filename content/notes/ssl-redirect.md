---
title: Managing SSL, Domains, and Redirects with EasyEngine
linkTitle: SSL, Redirects
weight: 5
type: docs
prev: binlog
next: cache
---

All SSL, domain, and alias domain-related settings are pre-configured in EasyEngine, making management effortless with simple commands.  

## Setting Up SSL

SSL can be configured instantly using the `--ssl=le` flag, along with wildcard and self-signed options. Example:  

```bash
# Set up Let's Encrypt SSL
ee site create example.com --ssl=le --wildcard

# Enable SSL for an existing website
ee site update example.com --ssl=le
```

Reference: [EasyEngine Site Create Command](https://easyengine.io/commands/site/create/)  

## Domain Management

### Adding a Domain  
Domains are automatically set up when creating a new website.  

### Changing a Domain Name  
EasyEngine doesn’t provide a direct command to rename a domain, but you can achieve this by cloning the site with a new name and deleting the old one:  

```bash
# Clone the old site to a new domain
ee site clone old-domain.com new-domain.com

# Delete the old site
ee site delete old-domain.com
```

References:  
- [EasyEngine Site Clone Command](https://easyengine.io/commands/site/clone/)  
- [EasyEngine Site Delete Command](https://easyengine.io/commands/site/delete/)  

## WWW and Non-WWW  

When you create a site with either www or non-www, EasyEngine automatically creates an alias for the other version. If you want to add additional aliases:  

```bash
ee site update example.com --add-alias-domains='a.com,*.a.com,b.com,c.com'
```

## Redirect Configuration  

If you need to set up redirects, add the rules to `user.conf` by creating a new `.conf` file inside the `nginx/custom/` directory. These custom configurations will remain unchanged during EasyEngine updates.  

```bash
nano /opt/easyengine/sites/YOUR-SITE.COM/config/nginx/custom/user.conf
```

Restart Nginx after making changes:  

```bash
ee site reload YOUR-SITE.COM
```

Keep in mind that this configuration will be placed inside the Nginx `server` block and will only apply to the specific site.  
