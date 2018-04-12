---
title: Using Cloudflare Free SSL In GitHub Pages With Custom Domain
date: 2018-04-11T15:16:42-04:00
lastmod: 2018-04-11T16:05:42-04:00
draft: false
keywords: ["Cloudflare", "SSL", "Free SSL", "GitHub pages", "Hugo", "Custom domain"]
description: "How to use Cloudflare free ssl in GitHub pages with custom domain"

categories:
- Blog
tags:
- Cloudflare
- SSL
- Hugo

comment: true
toc: true
autoCollapseToc: false
postMetaInFooter: true
hiddenFromHomePage: false
contentCopyright: ""
reward: false
mathjax: false
mathjaxEnableSingleDollar: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams:
  enable: false
  options: ""
---

Blog已成功從[Hexo][hexo]遷移到[Hugo][hugo]，新域名[AxdLog](https://axdlog.com)。由於Github Pages對自有域名不提供SSL證書，故須額外部署。[CloudFlare][cloudflare]提供免費的SSL證書，並且提供防DDOS攻擊功能。本文記錄如何配置並使用[CloudFlare][cloudflare]的SSL證書。

<!--more-->

## Configure Cloudflare
須先註冊[CloudFlare][cloudflare]帳號，若已經有，直接登入。


### Add Websites
若是新註冊用戶，登入後，出現`Add your site`頁面，輸入需要配置的域名。此處輸入`axdlog.com`。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-17-14_add_site.png)

點擊`Add site`按鈕後，出現`We're querying your DNS records`頁面

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-18-08_query_dns_records.png)

點擊`Next`按鈕，出現`Select a Plan`頁面。

### Select a CloudFlare Plan
套餐價格如下

| type | price |
| :--- | :--- |
| Free Website | $0/mon |
| Pro Website | $20/mon |
| Business Website| $200/mon |
| Enterprise | Get in touch |


![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-18-26_select_plan.png)

建議有經濟能力的選擇`Pro`或`Business`套餐，一是功能更多，二也算支持[Cloudflare](https://www.cloudflare.com)發展。此處，選擇`Free Website`，點擊按鈕`Confirm Plan`，出現`DNS query results`頁面

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-18-52_dns_query_result.png)


### Add DNS Records
顯示該域名當前的DNS記錄，刪除所有已有的記錄，新建2個`CNAME`記錄。

以下是本人的DNS Records

| Type | Name | Value | TTL | Status |
| :--- | :--- | :--- | :--- | :--- |
| **CNAME** | `axdlog.com` | `maxdsre.github.io` | Automatic | On CloudFlare |
| **CNAME** | `www` | `maxdsre.github.io` | Automatic | On CloudFlare |


![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-23-30_dns_records_setting.png)

操作完成後，出現`Change your Nameservers`頁面

### Change Your Nameservers
[CloudFlare][cloudflare]提供了2組DNS地址

* dina.ns.cloudflare.com
* roan.ns.cloudflare.com

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-24-23_change_nameserver.png)

本人的域名註冊商是[Namecheap](https://www.namecheap.com/)，在`Domain List`中選擇目標域名，在`NAMESERVERS`中選擇`custom dns`，將[CloudFlare][cloudflare]提供的2組DNS地址添加進去。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-26-23_change_nameserver.png)

操作完成後，返回[CloudFlare][cloudflare]的`Change your Nameservers`頁面，點擊`Continue`按鈕。


### Update Nameservers
頁面顯示如下信息

>**Status: Website not active (DNS modification pending)**
>
Please ensure your website is using the nameservers provided:
>
* brianna.ns.cloudflare.com
* kurt.ns.cloudflare.com
>
Allow up to 24 hours for this change to be processed. There will be no downtime when you switch your name servers. Traffic will gracefully roll from your old name servers to the new name servers without interruption. Your site will remain available throughout the switch


![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-27-22_panel_overview.png)

當在域名提供商處修改好`Nameservers`後，可點擊右側的`Recheck Nameservers`檢測是否生效。此處請耐心等待，域名解析需要等一段時間才能生效。

如果檢測通過，則 **`Status: Pending`** 會變成 **`Status: Active`**，並提示`This website is active on CloudFlare.`。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-11_cloudflare_free_ssl/2018-04-11_14-28-57_check_status_result.png)


### Crypto
>Manage Cryptography settings for your website.

返回頁面頂部，有`Cryptos`圖標點擊進入。

**`SSL`** 選項有四種選擇`Off`、`Flexible`、`Full`、`Strict`，點擊下方的`Help`，可查看相關解釋說明。

查閱的參考教程大都推薦`Flexible`，而本人則參考[Set up cloudflare](https://zaicheng.me/2016/01/02/hexo-blog-with-custom-domain-and-cloudflare/#Setup_github_pages_CNAME_record)，選擇`Full`。

接下來設置`HTTP Strict Transport Security (HSTS)`，點擊其右側的`Enable HSTS`進入，勾選`I understand`，點擊`Nest step`，開啓如下兩項

* `Apply HSTS policy to subdomains (includeSubDomains)`
* `No-Sniff Header`

點擊`Save`保存退出。

其餘的功能，可自行設置。

## Configuring Hugo
[Hugo][hugo]的設置要比[Hexo][hexo]簡單，只需在repo(maxdsre.github.io)的`master`分支中添加名爲`CNAME`的文件即可。將自有域名地址寫入該文件，此處寫入`axdlog.com`。

因爲是通過[Travis CI][travisci]進行集成部署，故而只需在`.travis.yml`文件添加一行設置。

```yml
script:
    - hugo
    # 添加行
    - echo 'axdlog.com' > public/CNAME
```


## Test in Browser
在瀏覽器中測試，能正常跳轉到`https`，並顯示綠色小鎖。在`Google Chrome`和`Mozilla Firefox`上測試通過。

當初爲[Hexo][hexo]配置SSL證書耗費了5個多小時，而現在整個過程不超過半個小時。


## References
* [Set Up SSL on Github Pages With Custom Domains for Free](https://sheharyar.me/blog/free-ssl-for-github-pages-with-custom-domains/)
* [How To Add Free Cloudflare SSL in Github Pages with Custom Domain?](https://www.goyllo.com/github/pages/free-cloudflare-ssl-for-custom-domain/)
* [Hexo blog with custom domain and cloudflare](https://zaicheng.me/2016/01/02/hexo-blog-with-custom-domain-and-cloudflare/)


## Change Log
* 2018.04.11 16:05 Wed America/Boston
    * 初稿完成


[hexo]: https://hexo.io "A fast, simple & powerful blog framework"
[hugo]: https://gohugo.io "The world’s fastest framework for building websites"
[cloudflare]: https://www.cloudflare.com
[travisci]: https://travis-ci.org "Test and Deploy with Confidence"
