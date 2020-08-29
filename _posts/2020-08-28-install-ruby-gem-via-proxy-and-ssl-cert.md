---
title: "How to install RubyGems dependencies behind company proxy with SSL certificates"
date: 2020-08-28
tags: ['proxy', 'installation', 'ssl']
categories:
    - ruby
---
This article is for those who have a Windows PC behind a company proxy setup with SSL certificate based authentication for outside network access and experiencing SSL verification errors.

## Step 1

Type `Win+R` to open the run dialog > type `certmgr.msc` to open the certificate manager tool.

## Step 2

Right click on `Certificates - Local computer` on the left panel. Select `Find certificates`. Look for the relevant keyword matching the certificate.

## Step 3

Double click on the certificate. Select `Details` tab and click `Copy to file` > `Next` > `Base-64 Encoded X.509` > Save to any directory. After saving, rename the file extension from .CER to .PEM.

## Step 4

Open a ruby-enabled command line. Run `gem which rubygems`.

You’ll see output like this:

`C:/Ruby21/lib/ruby/2.1.0/rubygems.rb`

We only need the prefix corresponding to the Ruby install directory. Enter the following command to open a file explorer window to the following directory:

`start C:\Ruby21\ssl\certs`

Move the .PEM file we saved earlier to this directory.

## Step 5

The pem files must be activated for CA lookup by using a OpenSSL-hashed filename. Double-click to run the `c_rehash.rb` helper script to activate all pem files in the directory and generate these hash files.

## Step 6

Run the gem install command with the proxy URL: `gem install --http-proxy http://proxy.internal.com:80 <package-name>`

## Step 7 (optional)

If you use bundler to install packages, either set the proxy URL in the `HTTP_PROXY` environment variable globally or within the current command line session with:

`set HTTP_PROXY=http://proxy.internal.com:80`

before running the `bundle install` command.

## Additional resources

1. [How to troubleshoot RubyGems and Bundler TLS/SSL Issues](https://bundler.io/guides/rubygems_tls_ssl_troubleshooting_guide.html)
