---
title: "SFTP Server in Google Cloud Platform - all through Terraform"
date: 2023-05-13T21:45:32+05:00
tags: ["terraform", "google-cloud-platform", "google-service-account", "sftp", "google-compute-engine", "virtual-machine"]
draft: true
---


## The background you might want to skip reading
Unfortunately needed to an sftp server to get some files.

*i know very data engineer of me. and very wicked of the 3rd party data providers .*.

And no they did not have an API.

And no they could not send in a cloud storage.

And no they were not flexible.

So eventually it was an sftp server that I needed. And since rest of the "stuff" was on Google Cloud Platform. This had to be there too.

And yes I know AWS provides a service for that.

## The background you might _NOT_ want to skip
So there are 3 options I found.

* Google Marketplace Product. I came across [
FileMage SFTP/FTP Gateway](https://console.cloud.google.com/marketplace/product/filemage-public/filemage-gateway-linux). It claims to be doing what we require here.

* Open source tool. I found [SFTPGo](https://github.com/drakkan/sftpgo). Seems solid. Good reviews. Good maintanence. The developer needs sponsorship btw [https://sftpgo.com/#pricing](https://sftpgo.com/#pricing).

* Getting my hands dirty. Spinning a VM on Google Compute Engine and mirroring it on google cloud storage bucket.


I went with option 3.

## The workflow that you might be interested in

I tried doing every thing in Terraform since all the rest of ELT stack was mostly under Terraform. Hard work but eventually made it through.

STEPS include

* generate public and private ssh key

```bash
ssh-keygen -t rsa -f key -b 2048
```

* create secret in secret manager and add publc key version

```hcl
resource "google_secret_manager_secret" "ssh-public-key" {
    secret_id = "ssh-public-key"
    replication {
        automatic = true
    }
}
```

* create a service account

```hcl
resource "google_service_account" "sftp-user" {
    account_id = "sftp-user"
}
```

* give `osLogin` and `serviceAccountUser` permission to the above service account.

* get current user id under which terraform is running

* get current user permission to impersonate the service account you created in step 1 using `serviceAccountTokenCreator` role

* get access token for impersonated account

* set google provider with above access token

* get user id with current user using google provider with above access token

* access ssh public key from secret manager

* add above public key to service account created in step 1

* create a service account for virtual machine

* create a cloud storage bucket

* give above service account the storage admin permission on google cloud bucket

* reserve a static external ip address

* create a vm instance in compute engine


## MOST HELPFUL ARTICLE
This was probably the only helpful article regarding this topic. 
https://pomba.net/2022/05/gcp-ssh-into-vms-as-service-account-when-oslogin-is-enabled/

**_Have questions? Reach out to me via [Linkedin](https://linkedin.com/in/urvahshabbir/)!_**

Gotta go! Buffalo!