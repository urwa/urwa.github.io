---
title: "Cloud Functions Python Packages Artifact Repo"
date: 2023-01-27T12:59:12+05:00
draft: false
---

## The background you might want to skip reading

I had multiple api sources which had some what the same response format and given the architecture in GCP, I had a number of similar python functions that were duplicated across multiple cloud functions. We all know what a mess that is.

Did the research, came across two ways to solve this

- A monolithic repo with common code in a separate folder and one folder per cloud function.
  - Pros: One place for common code
  - Cons: Cloud function deployment has code of all other cloud functions
- Deploy common code as a package in Artifact Repository.
  - Pros: Clean code for each cloud function
  - Cons: Deploying the package on Artifact Repository took me soooo long (AND HENCE THIS BLOG POST TO SAVE PEOPLE THE MISERY OF DEPLOYING THEIR FIRST PYTHON PACKAGE ON A PRIVE REPOSITORY)

I went with Option 2 because my OCD did not like all the mess in Cloud Function deployment containing irrelevent code.

## The actual steps that you might be interested in

### STEP 1 - Create the python package structure

Create a python package using [this official tutorial](https://packaging.python.org/en/latest/tutorials/packaging-projects/). Just make sure you do not name the folder as `src`. Name it something unique. I still am not sure if this was part of the problem but it be one of the issues.

**It is also recommended way is to keep the package name as single word. It will save you headache. Trust me.**

My final folder hierarchy ended up looking like this:

```text
cloudfunctioncommons
|_ cloudfunctioncommons
    |_ __init__.py
    |_ main.py
|_ license
|_ pyproject.toml
|_ readme.md
|_ tests
```

My `main.py` had all the reusable python functions. `tests` was empty as usual :smile:. `readme.md` had some fluff. `pyproject.toml` had some more fluff with some non-optional metadata. `license` has a whole fluff mountain.

### STEP 2 - Create a python repo in Artifact Repository

Just follow [this official doc](https://cloud.google.com/artifact-registry/docs/python/store-python#create). I would again suggest using single word name for the repo here as well. It was a mess for me with all these hypens and underscores.

### STEP 3 - Roll up the code into a package and deploy it on Artifact Repository

Then I had a cloud build trigger to deploy it on Artifact Repo. Easy peasy.

```yaml
steps:
  - name: python
    entrypoint: python
    dir: ${_GIT_FOLDER}
    args: ["-m", "pip", "install", "--upgrade", "pip"]     
  - name: python
    entrypoint: python
    dir: ${_GIT_FOLDER}
    args: ["-m", "pip", "install", "build", "--user"]
  - name: python
    entrypoint: python
    dir: ${_GIT_FOLDER}
    args: ["-m", "build"]
artifacts:
    python_packages:
    - repository: "https://${_GCP_LOCATION}-python.pkg.dev/${_GCP_PROJECT}/${_GCP_ARTIFACT_REPO}/"
      paths: ["${_GIT_FOLDER}dist/*"]
```

- `_GCP_PROJECT`: name of your google cloud project.
- `_GCP_LOCATION`: location of your repository in artifact repo, set in step 2.
- `_GIT_FOLDER`: name of the folder in your git repo where folder in step 1 is.

Better to deploy this via Terraform. It is very reliable way of doing things in cloud.

### STEP 4: Import the package in Cloud Functions

First thing is `requirements.txt` of your cloud functions

```text
--extra-index-url https://_GCP_LOCATION-python.pkg.dev/_GCP_PROJECT/_GCP_ARTIFACT_REPO/simple
cloudfunctioncommons==0.0.1 (or whatever verion you have written in `pyproject.toml` file of your python package)
rest of your packages here
```

This took me some time because of the format and also because you have to explicity name the package in a separate line apart from `--extra-index-url`.

Second thing is the `main.py` wjere you import that library. The import looks like the following for above package.

`from cloudfunctioncommons.main import your-python-function-name`

Since it was my first package deployment, it took me some hit and trials before I got it working. Start simple and then build on top of it.

**_Have questions? Reach out to me via [Linkedin](https://linkedin.com/in/urvahshabbir/)!_**
