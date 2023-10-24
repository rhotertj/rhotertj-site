+++
title = 'Automatic deployment with Github Actions'
date = 2023-10-22T15:29:19+02:00
draft = true
+++

After my SD card died, I started organizing my blog on Github. Since I use Hugo to build this static site,
the content and the source code are in the same repository.
Having my own site on Github was a good opportunity to deal with something that has been on my bucket list for a while:
Learning CI/CD pipelines.

Github offers a very low-threshold CI/CD solution called Github Actions. Much like `docker-compose` files,
you declare your pipeline and its triggers in a *yaml* file. Whenever some event happens with respect to your repository
that you have declared as a trigger in your Github action, your pipeline will be executed by a "runner", which is something
like a container running the latest Ubuntu.

My CI/CD pipelines includes building the site and then deploying it to my self-hosted server whenever I push to the main branch.

```deploy.yaml
name: Build and Deploy
on:
  push:
    branches:
      - main
      - master

env:
  HUGO_VERSION: 0.119.0

jobs:
  build-and-deploy:
    if: "!contains(github.event.head_commit.message,'[skip-ci]')"
    runs-on: ubuntu-latest
    steps:
```

Building the site is simple, as it only requires me to checkout my own code (recursing submodules for the theme I use),

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
  with:
      fetch-depth: 1
      clean: true
      submodules: recursive
```

downloading and installing hugo,

```yaml
- name: Download Hugo
  run: "wget https://github.com/gohugoio/hugo/releases/download/v${{ env.HUGO_VERSION }}/hugo_${{ env.HUGO_VERSION }}_linux-amd64.deb -O hugo_${{ env.HUGO_VERSION }}_Linux-64bit.deb"

- name: Install Hugo
  run: sudo dpkg -i hugo*.deb
```

and finally running the binary to build the website.

```yaml
- name: Build site with Hugo
  run: hugo --minify
```

Deploying the results found in the `public` directory of the container is a bit inconvenient, as I need to transfer its contents to my personal server.
I chose `rsync` for this task because it uses `ssh`, which comes with built-in public key authentication.

First, I create a public/private key pair using elliptic curves with `ssh-keygen`. These keys are used to establish a secure connection between Github and my server.

```bash
user@myserver$ ssh-keygen -t ed25519 -C "hey@example.com"
```

We need to add the public key to my private server and the private key to Github's pipeline runner.

We add the public key to my server by placing the it in the `~/.ssh/` directory of the user we want to use for deployment and adding it to the `authorized_keys` file.

```bash
user@myserver$ cat id_ed25519.pub >> authorized_keys
```

The Github action pipeline needs the private key for the corresponding public key to successfully authenticate itself to my server. For such use cases, Github provides *action secrets*. *Secrets* belong to a repository and can be referenced in Github action declarations, without being part of the repository itself. They are not visible to the public and are not copied when the repository is cloned or forked. Over the past few years, I have come across multiple AWS or Azure keys in machine learning research repositories and this feature conveniently solves that problem.

The following things need to be stored as Github secrets:
 - My server address: DEPLOY_SSH_HOST
 - The private key: DEPLOY_SSH_KEY
 - The user who is deploying: DEPLOY_SSH_USER
 - The desired path: DEPLOY_PATH


We can now install the private key and add my server as a known host.

```yaml
- name: Install SSH Key
  run: |
    mkdir -p ~/.ssh/
    echo "${{ secrets.DEPLOY_SSH_KEY }}" > ~/.ssh/private.key
    sudo chmod 600 ~/.ssh/private.key

- name: Adding Known Hosts
  run: ssh-keyscan -H ${{ secrets.DEPLOY_SSH_HOST }} >> ~/.ssh/known_hosts

```

Finally, we call rsync, input the private key, and deploy our website to the desired path.

```yaml
- name: Deploy with rsync
  run: rsync -avz  -e "ssh -i ~/.ssh/private.key" ./public/ ${{ secrets.DEPLOY_SSH_USER }}@${{ secrets.DEPLOY_SSH_HOST }}:${{ secrets.DEPLOY_PATH }}""
```

The complete `deploy.yaml` file that defines the Github workflow can be found [here](https://github.com/rhotertj/rhotertj-site/blob/main/.github/workflows/deploy.yaml).

I am still a bit hesitant about uploading an ssh key that can log into my server to a foreign entity, even though it should be encrypted and stored securely. 

For now though, I am enjoying the magical feeling of deploying directly with `git push`. 
![Deployment](https://img.shields.io/github/actions/workflow/status/rhotertj/rhotertj-site/deploy.yaml)