---
title: Using Cloudflare Free SSL In GitHub Pages With Custom Domain
slug: Using Cloudflare Free SSL In GitHub Pages With Custom Domain
date: 2018-04-11T15:16:42-04:00
lastmod: 2018-04-11T15:16:42-04:00
draft: false
keywords: ["AxdLog", "Cloudflare", "SSL", "Free SSL", "GitHub pages", "Hugo", "Custom domain"]
description: "How to use Cloudflare free ssl in GitHub pages with custom domain"
categories:
- Blog
tags:
- Cloudflare
- SSL
- Hugo

comment: true
toc: true
autoCollapseToc: true
contentCopyright: ""
mathjax: false

---

I have succeed in migrating my blog from [Hexo][hexo] to [Hugo][hugo] platform, new domain is [**AxdLog**](https://axdlog.com). As [Github Pages][githubpage] doesn't provide SSL certificate for custom domain, so I need to deploy it separately. [CloudFlare][cloudflare] provides free SSL certificate, and it has the ability of anti-ddos.

This article documents how to configure
[CloudFlare][cloudflare] free SSL certificate in [Github Pages][githubpage].

<!--more-->

## Configure Cloudflare
Registering [CloudFlare][cloudflare] account, if you have one, just login.

### Add Websites
If you're a new registered user. It will show a page **Add your site**. Inputing the target domain, here I input `axdlog.com`.

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-17-14_add_site.png)

Clicking button `Add site`, it shows page **We're querying your DNS records**:

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-18-08_query_dns_records.png)

Clicking button `Next`, it shows page **Select a Plan**

The following is price list:

| type | price |
| :--- | :--- |
| Free Website | $0/mon |
| Pro Website | $20/mon |
| Business Website| $200/mon |
| Enterprise | Get in touch |


![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-18-26_select_plan.png)

It is recommended that you choose  plan `Pro` or `Business` if you have the ability to pay. The benefits are more functions and support the development of [Cloudflare](https://www.cloudflare.com).

Here I choose `Free Website`, clicking button `Confirm Plan`, it show page **DNS query results**.

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-18-52_dns_query_result.png)


### Add DNS Records
It list all currently existed DNS records of your domain. Deleteing all existed records, creating 2 new records.

The following are my DNS records:

| Type | Name | Value | TTL | Status |
| :--- | :--- | :--- | :--- | :--- |
| **CNAME** | `axdlog.com` | `maxdsre.github.io` | Automatic | On CloudFlare |
| **CNAME** | `www` | `maxdsre.github.io` | Automatic | On CloudFlare |


![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-23-30_dns_records_setting.png)

Once operation is completely, it shows page **Change your Nameservers**.


### Change Your Nameservers
[CloudFlare][cloudflare] provides 2 DNS address:

* dina.ns.cloudflare.com
* roan.ns.cloudflare.com

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-24-23_change_nameserver.png)

The domain registrar of my domain is [Namecheap](https://www.namecheap.com/), selecting target domain in `Domain List`, choosing `custom dns` in `NAMESERVERS` section, add the 2 DNS address into the input field.

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-26-23_change_nameserver.png)

Once operation is finished, it will return to page **Change your Nameservers**, clicking button `Continue`.

### Update Nameservers
The page shows:

>**Status: Website not active (DNS modification pending)**
>
Please ensure your website is using the nameservers provided:
>
* brianna.ns.cloudflare.com
* kurt.ns.cloudflare.com
>
Allow up to 24 hours for this change to be processed. There will be no downtime when you switch your name servers. Traffic will gracefully roll from your old name servers to the new name servers without interruption. Your site will remain available throughout the switch


![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-27-22_panel_overview.png)

If you are finished changing `Nameservers` at domain registrar, client the button `Recheck Nameservers` at the right of the page. It will check it the configuration is active. It will takes a while to take effect, just be patient.

It it is passed, the state of **`Status: Pending`** will be changed to **`Status: Active`**, showing `This website is active on CloudFlare.`。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-28-57_check_status_result.png)


### Crypto
>Manage Cryptography settings for your website.

Return to the top of the page, clicking icon `Cryptos`.

**`SSL`** has 4 choices: `Off`、`Flexible`、`Full`、`Strict`. If you wanna view the explanation, just clicking the following `Help`.

Commonly choosing `Flexible`, I choose `Full` referenced by [Set up cloudflare](https://zaicheng.me/2016/01/02/hexo-blog-with-custom-domain-and-cloudflare/#Setup_github_pages_CNAME_record).

Next setting up `HTTP Strict Transport Security (HSTS)`, clicking `Enable HSTS`, choosing `I understand`, then clicking `Nest step`, enabling the  following 2 settings:

* `Apply HSTS policy to subdomains (includeSubDomains)`
* `No-Sniff Header`

Clicking `Save` to exit.

You can optimize other functions according to personal needs.


## Configuring Hugo
Setting operation in [Hugo][hugo] is simpler then [Hexo][hexo]. What you need to do is just add a file named `CNAME` into the branch `master` of repo (maxdsre.github.io). Adding the custom domain name into the file, here the content is `axdlog.com`.

Due to it is integrated deploying through [Travis CI][travisci], I just need to add a new line in config file `.travis.yml`.

```yml
script:
    - hugo
    # new line
    - echo 'axdlog.com' > public/CNAME
```


## Test in Browser
Testing in browser, it redirects to `https` successfully and shows green padlock icon. It is approved in `Google Chrome` and `Mozilla Firefox`.

It took me at least 5 hours to configure SSL in [Hexo][hexo]. But now the total operation is less then half an hour.


## References
* [Set Up SSL on Github Pages With Custom Domains for Free](https://sheharyar.me/blog/free-ssl-for-github-pages-with-custom-domains/)
* [How To Add Free Cloudflare SSL in Github Pages with Custom Domain?](https://www.goyllo.com/github/pages/free-cloudflare-ssl-for-custom-domain/)
* [Hexo blog with custom domain and cloudflare](https://zaicheng.me/2016/01/02/hexo-blog-with-custom-domain-and-cloudflare/)


## Change Log
* 2018.04.11 16:05 Wed America/Boston
    * first draft


[githubpage]: https://pages.github.com "Hosted directly from your GitHub repository. Just edit, push, and your changes are live."
[hexo]: https://hexo.io "A fast, simple & powerful blog framework"
[hugo]: https://gohugo.io "The world’s fastest framework for building websites"
[cloudflare]: https://www.cloudflare.com
[travisci]: https://travis-ci.org "Test and Deploy with Confidence"
