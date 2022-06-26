---
title: "Terraform, Google Service Accounts and how to deploy Cloud Functions using Cloud Build"
date: 2022-06-26T09:43:10+05:00
tags: ["terraform", "google-cloud-platform", "google-service-account", "cloud-functions", "cloud-build"]
draft: false
---

### The background you might want to skip reading
I did not go with the usual path of deploying cloud functions using terraform by zipping source code, 
uploading code to google cloud storage and then creating/deploying cloud functions. Not to forget the service accounts
and permissions that have to be set.

Why?

Here are my reasons:
- Terraform still does not provide an inbuilt feature to check if there is a change in codebase.
- My code was in github and I did not wanted to upload it to google cloud storage (double work) and then do deployment.
- The hacks for hashing was not working for me.
- I could not figure out the service account permissions for all the above work.
- I already had continuous deployment via cloud build in place that I wanted to bring under terraform umberalla.


### The workflow that you might be interested in
Here is how the workflow worked:
- You do local development, highly recommend using `functions-framework` for this.
- Then you push it your feature branch and open a pull request.
- This triggers the cloud build workflow.
- Cloud build checks if changes have been made in your target cloud functions folder, it deploys that function.

To terraform this, I did the following:
- Created cloud build trigger using terraform.
- Created google service accounts using terraform.
- Deployed all this code through my terraform workflow (which is also via another separate cloud build trigger).


My kids are crying... I will share code and more details once I have them distracted with something. 


**_Have questions? Reach out to me via Linkedin!_**

See ya later! alligator!!!