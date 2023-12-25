---
title: "A version-controlled way of deploying SFTP Server of Google Cloud Platform"
date: 2023-05-13T21:45:32+05:00
tags: ["terraform", "google-cloud-platform", "google-service-account", "sftp", "google-compute-engine", "virtual-machine", "gcsfuse"]
draft: false
---

*DISCLAIMER: complete setup of sftp server on GCP was done via Terraform except for adding secret value in secret manager.*

## The background you might want to skip reading

Unfortunately needed to an sftp server to get some files.

*i know very data engineer of me. and very wicked of the 3rd party data providers .*.

And no they did not have an API.

And no they could not send in a cloud storage.

And no they were not flexible.

So eventually it was an sftp server that I needed. And since rest of the "stuff" was on Google Cloud Platform. This had to be there too.

And yes I know AWS provides a service for that.

## The background you might *NOT* want to skip

So there are 3 options I found.

* Google Marketplace Product. I came across [
FileMage SFTP/FTP Gateway](https://console.cloud.google.com/marketplace/product/filemage-public/filemage-gateway-linux). It claims to be doing what we require here.

* Open source tool. I found [SFTPGo](https://github.com/drakkan/sftpgo). Seems solid. Good reviews. Good maintanence. The developer needs sponsorship btw [https://sftpgo.com/#pricing](https://sftpgo.com/#pricing).

* Getting my hands dirty. Spinning a VM on Google Compute Engine and mirroring it on google cloud storage bucket.

I went with option 3.

## The workflow that you might be interested in

I tried doing every thing in Terraform since all the rest of ELT stack was mostly under Terraform. Hard work but eventually made it through.

STEPS include

* generate public and private ssh key using following command as explained in [this blog](https://pomba.net/2022/05/gcp-ssh-into-vms-as-service-account-when-oslogin-is-enabled/)

```bash
ssh-keygen -t rsa -f key -b 2048
```

* create secret in secret manager  using `google_secret_manager_secret` and add publc key version manually

* create a service account using `google_service_account`

* give `osLogin` and `serviceAccountUser` permission to the above service account

* get current user id under which terraform is running using `google_client_openid_userinfo`

* get current user permission to impersonate the service account you created in step 1 using `serviceAccountTokenCreator` role using `google_service_account_iam_member`

* get access token for impersonated account using `google_service_account_access_token`

* set google provider with above access token with alias `impersonated`

* get user id with current user using google provider with above access token and using provider as `google.impersonated`

* access ssh public key from secret manager using data `google_secret_manager_secret_version`

* add above public key to service account created in step 1 using `google_os_login_ssh_public_key`

* create a service account for virtual machine using `google_service_account`

* create a cloud storage bucket `google_storage_bucket`

* give above service account the storage admin permission on google cloud bucket using `google_storage_bucket_iam_member`

* reserve a static external ip address `google_compute_address`

* create a vm instance in compute engine using `google_compute_instance` that uses external ip, storage bucket and google service account created above.

## other useful tidbits

* use start up script to install gcsfuse as mentioned in [official google docs here](https://cloud.google.com/storage/docs/gcsfuse-quickstart-mount-bucket#install).
* to mount the bucket run the mount command as service account user created above (which has osLogin and serviceAccountUser permissions).
It can be run as the following command. You can get the ssh name as explained [here](https://pomba.net/2022/05/gcp-ssh-into-vms-as-service-account-when-oslogin-is-enabled/) and gcsfuse mount command from offical docs linked above.

```bash
sudo -u service_acccount_ssh_name bash -c 'gcsfuse command here'
```

* enable serial port logging for better debugging the vm startup script.
* enable oslogin on vm as well.

## most helpful article regarding this topic

This was probably the only helpful article regarding this topic.
<https://pomba.net/2022/05/gcp-ssh-into-vms-as-service-account-when-oslogin-is-enabled/>

*Have questions? Reach out to me via [Linkedin](https://linkedin.com/in/urvahshabbir/)!*
