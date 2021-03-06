---
title: AWS S3 Setup for WordPress
description: Add the ability to integrate with AWS S3 to a WordPress site on Pantheon
tags: [siteintegrations]
categories: [wordpress]
contributors:
  - sarahg
date: 6/4/2018
---

Amazon Web Services (AWS) offers Simple Storage Service (S3) for scalable storage and content distribution, which can be integrated with sites running on Pantheon. Pantheon already offers content distrubution through the [Global CDN](/docs/global-cdn/), but S3 is a good option for addressing issues with [highly populated directories](/docs/platform-considerations/#highly-populated-directories) or [serving large files](/docs/platform-considerations/#large-files).

## Before You Begin

Be sure that you have:

- An existing WordPress site on Pantheon, or [create](https://dashboard.pantheon.io/sites/create){.external} one.
- A [local clone](/docs/git/#clone-your-site-codebase) of your code repository.
- An account with [Amazon Web Services (AWS)](https://aws.amazon.com/s3/){.external}. Amazon offers [free access](https://aws.amazon.com/free/){.external} to most of their services for the first year.
- [Terminus](/docs/terminus) installed on your local computer.

<div class="alert alert-info" role="alert">
<h4 class="info">Note</h4>
<p>When creating an AWS account, you will have to enter credit card information. This is required, but you will not be charged unless you exceed the usage limits of their free tier.</p></div>

<div class="alert alert-export" role="alert">
<h4 class="info">Exports</h4>
<p markdown="1">This process uses [Terminus](/docs/terminus/) commands. Before we begin, set the variable `$site` in your terminal session to match your site name:
<pre>
<code class="bash">export site=yoursitename
export env=dev
</code></pre>
</p>
</div>

## Configure S3 within the AWS Console
Before integrating S3 with your site, you'll need to configure the service within your [AWS Management Console](https://console.aws.amazon.com){.external}.

### Create a New AWS S3 Bucket
If you do not have an existing bucket for your site, create one:

1. From your [AWS Console](https://console.aws.amazon.com){.external}, click **S3**.
2. Click **Create Bucket**.
3. Enter a bucket name. The bucket name you choose must be unique across all existing bucket names in Amazon S3, and after you create a bucket, you cannot change its name. Because the bucket name is visible in the URL that points to the objects stored in the bucket, ensure that the bucket name you choose is appropriate.
4. Select a region and click **Create**.
5. The **Set properties** section has additional configuration options you can configure now, or wait and configure later. When complete, click **Next**.
6. In the **Permissions** tab, tick the boxes for **Read** and **Write** access for both **Objects** and **Permissions**, then click **Next**.
7. Review your settings, and then click **Create bucket**.

## Integrate S3 with WordPress
You will need to install a plugin such as [S3 Uploads](https://github.com/humanmade/S3-Uploads){.external} or [WP Offload S3](https://deliciousbrains.com/wp-offload-s3/){.external}.

WP Offload S3 requires a paid license but is configurable in the WordPress admin UI and offers a number of options and features. S3 Uploads is open-source but does not include an admin UI and requires [Terminus](/docs/terminus) and [WP-CLI](/docs/wp-cli) for setup and migration.

### Install and Deploy S3 Uploads

<div class="alert alert-info" role="alert">
  <h4 class="info">Note</h4>
  <p markdown="1">This plugin currently conflicts with [Solr Power](https://wordpress.org/plugins/solr-power/){.external}, our recommended plugin for Solr integration. [More info](https://github.com/humanmade/S3-Uploads/issues/80){.external}.</p>
</div>

1. Download the latest plugin release from [Github](https://github.com/humanmade/S3-Uploads/releases){.external} and extract it to `wp-content/plugins/`. Note that our documentation has been tested for version 2.0.0.

    <div class="alert alert-danger" role="alert">
    <h4 class="info">Warning</h4>
    <p markdown="1">**Do not** add the plugin as a Git submodule. Git submodules are not supported on the platform (<a data-proofer-ignore href="/docs/git-faq/#does-pantheon-support-git-submodules">more info</a>).</p>
    </div>

2. Rename the extracted folder to remove the version number. For example:

    ```bash
    mv S3-Uploads-2.0.0/ S3-Uploads
    ```

3. Create and / or copy your **Access Key ID** key and **Secret Access Key** from the "My security credentials" section of your AWS account to a text editor on your local computer.

    <div class="alert alert-info" role="alert">
    <h4 class="info">Note</h4>
    <p markdown="1">As a standard security measure, consider creating a unique user with limited permissions covering this S3 bucket to authenticate the plugin.</p>
    </div>

4. Add the credentials to `wp-config.php`, as described in the plugin's [README](https://github.com/humanmade/S3-Uploads#getting-set-up){.external} file. For security, we recommended a service like [Lockr](/docs/guides/lockr/) or the [Terminus Secrets plugin](https://github.com/pantheon-systems/terminus-secrets-plugin){.external} to store and retrieve these credentials securely.

5. Commit and push the new plugin and your `wp-config.php` updates to the Dev environment, then  switch to SFTP mode and activate the plugin:

    ```bash
    terminus wp $site.dev plugin activate S3-Uploads
    ```

6. Use WP-CLI to verify your AWS setup.

    ```bash
    terminus wp $site.dev s3-uploads verify
    ```

#### Migrate existing media using S3 Uploads and WP-CLI

You can migrate existing media files to S3 with the following command:

```bash
terminus wp $site.dev -- s3-uploads migrate-attachments
```

Optionally, add the `--delete-local` flag to remove the local copies of the media files.

Upon succesful migration, this command will also provide a search/replace command for your database to update references to the newly-migrated files. Note that you will need to run this on all Pantheon environments (dev/test/live).

#### Further configuration
Check out the plugin's [README file](https://github.com/humanmade/S3-Uploads/blob/master/README.md){.external} for information on advanced configuration, such as cache control, URL rewriting and offline development.

### Install and Deploy WP Offload S3
Follow documentation from [DeliciousBrains](https://deliciousbrains.com/wp-offload-s3/doc/quick-start-guide){.external}. No specialized configuration is required for this plugin to run on Pantheon.
