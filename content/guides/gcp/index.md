---
title : "Replicating to GCP Cloud Storage"
date: 2021-04-12T00:00:00Z
layout: docs
menu:
  docs:
    parent: "guides"
weight: 350
---

This guide will show you how to use GCP Cloud Storage as a database replica path for
Litestream.

At a high level, this guide will walk you through the following process:
1. Create a bucket
1. Create a service account
1. Assign permissions
1. Use the key, secret, bucket, and GCP Cloud Storage request endpoint to replicate a db

## Create a Bucket

To begin, you'll need a bucket to store your data.
This assumes you have a project in GCP and are at the [GCP Cloud Console](https://console.cloud.google.com/).
You can create a bucket through the GCP Web Console as follows:

1. Press `/` to focus on the search bar and type `storage`
1. Select the Cloud Storage product
1. Click the `Create Bucket` option near the top left corner
1. Provide a **globally** unique name (a DNS-like name would be recommended if you have verified control of a domain)
1. Select where to store your data
1. Select the `Standard` storage class
1. Select `Uniform` access control
1. Select the key management strategy you prefer (if you're looking for no-hassle, then stick with a Google-managed strategy)
1. Click `Create`

You now have a Cloud Storage Bucket and should be looking at the details view for the newly created bucket.
We'll come back to this view in a later step, right now we need to create a service account so that we can have a key and secret to provide litestream.

## Create a Service Account

While there are many ways to create service accounts in GCP, we'll take the shorter route of going to the `Settings` option in the left-hand menu.
Once you're looking at the Cloud Storage Settings, you'll want to click the `Interoperability` tab at the top.
In the `Service Account HMAC` section, you'll see an option to `Create a Key for another Service Account`. Click that option.

Selecting to `Create a Key for another Service Account` will open a modal dialog above that page that has an option in the lower-right corner to `Create New Account`.
You'll want to click on that option, give the service account a good name, 
do **not** grant that account access to the project, and do **not** grant users access to the new account.
Once you click `Done` on that dialog for creating the new service account, you will be taken back to the Cloud Storage Settings and shown the key and secret.
**You will not be able to see the secret after closing this new dialog, so copy it out to somewhere safe.**
We'll use them both in a later step.

In addition to needing the key and secret that was just shown to you, you'll need the email address that was created for the new service account.
You can see the email address under that same `Service Account HMAC` section.
It follows a pattern like `<service account name>@<project id>.iam.gserviceaccount.com` - copy the entire email address - we'll use it in the next step.

## Assign Permissions

For this next step you will want to go back to the details view for the newly created bucket.
You can get to the details view for a bucket by selecting `Browser` from the left-hand menu and then clicking the name of the bucket you want to see details for.
Once you're in the details view for the newly created bucket you'll want to click on the `Permissions` tab.
Under that `Permissions` tab, there is a `Permissions` section header, part of the way down the page, you will want to click on the `Add` option that is to the right of that section header.
Paste the email address for the new service account in the `New Members` field and wait for it to auto-complete and then select the matched account.
You'll need to assign two roles to this service account. The `Storage Legacy Bucket Owner` and the `Storage Object Viewer` role.
After you've added those roles, click `Save`.

## Use the Created Values

The final step is to run the litestream command using the key, secret, bucket name, and GCP request endpoint as follows for a linux shell command: `
AWS_ACCESS_KEY_ID=<key> AWS_SECRET_ACCESS_KEY=<secret> litestream replicate /path/to/fruits.db s3://<bucket>.storage.googleapis.com/fruits.db`
