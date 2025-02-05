+++
title = "Creating Zola blog with GitHub Actions and custom domain"
date = 2025-01-06

[taxonomies]
categories = ["blog"]
other_websites = ["patryk.guba.pl"]
tags = ["code","blog","zola","github actions","webdev"]
+++

Hi, this is my first post on this blog I've just created, so I wanted to start with describing the process of how to create similar website.
<!-- more -->
In the past, I used to host my website on Raspberry Pi connected via home network. It was a lot of fun, but also it required some effort to keep the server running smoothly. On the other hand, GitHub Pages are free and I really like all types of CI/CD automations like GitHub Actions, so it sounds like a good alternative to try. Also, while GitHub Pages provide you free domain as **username.github.io**, it's also possible to add your own custom domain for your website, so your website can become really yours!

To start, clone (or fork) Zola repository [here](https://github.com/getzola/zola) to create your own Zola repository that will be used to compile to a website. You can also choose one of the [themes](https://www.getzola.org/themes/) to customize your website.

Next, configure GitHub Action for deployment. You can use [shalzz/zola-deploy-action](https://github.com/shalzz/zola-deploy-action) action that will take your code from your Zola website repository, create automatically seperate branch **gh-pages** with compiled website code and deploy code from this branch to your GitHub Page.

To use this GitHub Action, create a YAML file with content below in your Zola repository in **.github/workflows** directory. You can choose any filename ending in **.yml** or **.yaml** extension.


```yaml

name: Zola on GitHub Pages

on: 
 push:
  branches:
   - main

jobs:
  build:
    name: Publish site
    runs-on: ubuntu-latest
    steps:
    - name: Checkout main
      uses: actions/checkout@v4
    - name: Build and deploy
      uses: shalzz/zola-deploy-action@v0.19.2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**secrets.GITHUB_TOKEN** should be provided by GitHub without any additional configuration as long as you are the owner of you Zola website repository. To add your custom domain, add file named **CNAME** containing your custom domain name into **static** directory in your repository. Then go to repository Settings, and then in section Code and automation choose Pages where you can add your domain (it will require to add some DNS records in your domain provider console).

{{ responsive(src="assets/githubsettings.png", width=720, height=120, alt="GitHub settings image", caption="Custom domain settings in GitHub repository") }}

To redirect from apex domain to subdomain I used this nginx script:

```txt
server {
        server_name  .guba.pl;
        return 301 $scheme://patryk.guba.pl$request_uri;
}
```