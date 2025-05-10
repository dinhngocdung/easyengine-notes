---
title: Guide to Setting Up Development, Stage, and Production with EasyEngine 4 on macOS
linkTitle: Development, Stage and Production
weight: 11
type: docs
prev: borgmatic
next: migrate
---

## 1. Introduction to the Dev-Stage-Production Workflow and EasyEngine 4

The **Development (Dev)**, **Stage**, and **Production** workflow is a standard in DevOps for efficient website development and deployment:

- **Development**: A local environment for testing plugins, themes, or new features.
- **Stage**: A testing environment mirroring Production before deployment (can be skipped for simple deployments).
- **Production**: The live environment serving users.

**EasyEngine 4** integrates commands like `ee site clone`, `ee site sync`, and `ee site share` to support this workflow. This guide explains how to use EasyEngine 4 on macOS, with accurate flags for `clone` and `sync`, using placeholder images when skipping large `uploads` directories, and deploying for a website at **YOUR-SITE.COM** on the server **YOUR-SERVER-IP**.

![Development, Stage, and Production](/images/development.svg)


## 2. Creating a Local Development Environment on macOS

- Install **Docker Desktop** on macOS (download from Docker Desktop).
- Install **Homebrew** (if not already installed):
    
    ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```
    
- Install **EasyEngine 4** via Homebrew:
    
    ```bash
    brew install easyengine
    ee --version
    ```
    
- SSH access to the Production server (`root@YOUR-SERVER-IP`) with an SSH key pair for secure data transfer:
    
    ```bash
    ssh-keygen -t rsa
    ssh-copy-id root@YOUR-SERVER-IP
    ```
    

## 3. Creating a Development Site on macOS for Development

To develop plugins, themes, or features, create a local Dev site (`dev.YOUR-SITE.test`) from the Production site (`YOUR-SITE.COM`).

### 3.1. Clone Site from Production to Dev

According to EasyEngine Clone, the `ee site clone` command supports the following flags:

- `-db`: Clone only the database.
- `-files`: Clone only files (excluding `uploads`).
- `-uploads`: Clone only the `uploads` directory.
- No flags: Clone everything (database, files, and `uploads`).

To avoid SSL validation errors, add the `--ssl=off` flag. If only the database and files are needed (skipping large `uploads`):

```bash
ee site clone root@YOUR-SERVER-IP:YOUR-SITE.COM dev.YOUR-SITE.test --db --files --ssl=off
```

**Note**:

- The command requires SSH configuration to `root@YOUR-SERVER-IP`.

Configure local DNS:

```bash
sudo nano /etc/hosts
```

Add the line:

```
127.0.0.1 dev.YOUR-SITE.test
```

Check the Dev site at `http://dev.YOUR-SITE.test`.

### 3.2. Add Placeholder for Images

If the `uploads` directory is not cloned, images on Dev may be broken. Add placeholder code to replace missing images.

Open the `functions.php` file of the child theme:

```bash
nano ~/easyengine/sites/dev.YOUR-SITE.test/app/htdocs/wp-content/themes/YOUR-THEME/functions.php

```

Add the following code:

```php
// IMAGE PLACEHOLDER
function replace_broken_images_with_placeholder($content) {
    $placeholder = '<https://sanantoniosports.org/wp-content/uploads/2022/07/placeholder-image.jpeg>';

    // Use regex to replace missing images
    return preg_replace_callback('/<img[^>]+src=["\\']([^"\\']+)["\\'][^>]*>/i', function($matches) use ($placeholder) {
        $src = $matches[1];

        // Check if the image exists on the server
        $path = ABSPATH . str_replace(site_url('/'), '', $src);
        if (!file_exists($path)) {
            // Replace with placeholder image
            return str_replace($src, $placeholder, $matches[0]);
        }

        return $matches[0];
    }, $content);
}
add_filter('the_content', 'replace_broken_images_with_placeholder');

// Return placeholder for missing product images
function custom_wc_product_image_placeholder($image, $attachment_id, $size, $icon) {
    $file_path = get_attached_file($attachment_id);
    $placeholder_url = '<https://sanantoniosports.org/wp-content/uploads/2022/07/placeholder-image.jpeg>';

    if (!file_exists($file_path)) {
        // Return placeholder image if file does not exist
        $image = [$placeholder_url, 600, 600, false];
    }

    return $image;
}
add_filter('wp_get_attachment_image_src', 'custom_wc_product_image_placeholder', 10, 4);

add_filter('woocommerce_placeholder_img_src', function() {
    return '<https://sanantoniosports.org/wp-content/uploads/2022/07/placeholder-image.jpeg>';
});
```

Save the file and exit.

### 3.3. Sync from Production to Dev (if needed)

When Production changes, use `ee site sync` to synchronize. According to EasyEngine Sync, the flags are similar to `clone`:

- `-db`: Sync only the database.
- `-files`: Sync only files (excluding `uploads`).
- `-uploads`: Sync only the `uploads` directory.
- No flags: Sync everything.

Typically, only the database is synced:

```bash
ee site sync root@YOUR-SERVER-IP:YOUR-SITE.COM dev.YOUR-SITE.test --db
```

### 3.4. Clear Cache on Dev

Clear the cache to apply changes:

```bash
ee site clean dev.YOUR-SITE.test
```


## 4. Deploying from Local Dev to Production

When plugins, themes, or features on Dev are ready, you can deploy directly from Dev to Production (skipping Stage for simple projects).

### 4.1. Sync Database

Sync the database from Dev to Production:

```bash
ee site sync dev.YOUR-SITE.test root@YOUR-SERVER-IP:YOUR-SITE.COM --db
```

### 4.2. Remove Placeholder Before Syncing Files

The placeholder code is only for Dev, so remove it before syncing files:

```bash
nano ~/easyengine/sites/dev.YOUR-SITE.test/app/htdocs/wp-content/themes/YOUR-THEME/functions.php
```

Remove the placeholder code.

### 4.3. Sync Files

Sync files, skipping `uploads` if not needed:

```bash
ee site sync dev.YOUR-SITE.test root@YOUR-SERVER-IP:YOUR-SITE.COM --files
```

### 4.4. Clear Cache on Production

Clear the cache to apply changes:

```bash
ee site clean YOUR-SITE.COM
```


## 5. Optional: Setting Up a Stage Environment

The Stage environment is used for testing before deployment but can be skipped for simple projects. If needed, there are two options:

### 5.1. Deploy on a Remote Server

Create a Stage site on a remote server, e.g., **STAGE-SERVER-IP**:

- Set up the server similarly to the local environment:
    - Run Ubuntu/Debian and install EasyEngine 4.
    - Clone/Sync from Dev:
        
        ```bash
        ee site clone dev.YOUR-SITE.test root@STAGE-SERVER-IP:stage.YOUR-SITE.COM --db --files --ssl=off
        ```
        
    - Sync `uploads` from Production if needed:
        
        ```bash
        ee site sync root@YOUR-SERVER-IP:YOUR-SITE.COM stage.YOUR-SITE.COM --uploads
        ```
        
- Check on `stage.YOUR-SITE.COM`.

### 5.2. Share Directly from macOS with ngrok

For simple needs, you can share the Dev environment directly with clients for testing.

Create a temporary URL from the Dev environment on macOS for quick testing:

```bash
ee site share dev.YOUR-SITE.test
```

- Creates a public URL via ngrok.
- Limitations: Does not support custom domains; default duration is 24 hours.
- Limit to 1 hour with `-expire`:
    
    ```bash
    ee site share dev.YOUR-SITE.test --expire=3600
    ```
    

## 6. Conclusion

EasyEngine 4, installed via `brew install easyengine`, facilitates the setup and management of Dev, Stage, and Production environments on macOS. Commands like `ee site clone` and `ee site sync` with flags `--db`, `--files`, `--uploads`, and `--ssl=off` for cloning allow precise data control, combined with placeholder images to optimize when skipping `uploads`. The `ee site share` command supports quick Stage testing. For simple projects, you can deploy directly from Dev (`dev.YOUR-SITE.test`) to Production (`YOUR-SITE.COM` on `YOUR-SERVER-IP`), bypassing Stage.

Familiarize yourself with the Dev-Stage-Production workflow to become a professional website deployer. With EasyEngine 4, you’ll save time and ensure high-quality deployments.

**Sources:**
- [EasyEngine Clone](https://easyengine.io/commands/site/clone/)
- [EasyEngine Sync](https://easyengine.io/commands/site/sync/)
- [EasyEngine Share](https://easyengine.io/commands/site/share/)
