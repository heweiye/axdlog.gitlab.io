# Hugo Theme Jane Optimization
---

merge `theme/jane/dev-config.toml`, `themes/jane/exampleSite/config.toml` --> `config.toml`

`themes/jane/archetypes/default.md` --> `archetypes/default.md`
add `slug` format same to `title`

`themes/jane/i18n/` language file

`themes/jane/layouts/partials/post/i18nlist.html`

`themes/jane/layouts/_default/baseof.html` change `<title>`

`themes/jane/layouts/_default/single.html` comment `<header>`, `Copyright`, `Reward`, `Comments`

`themes/jane/layouts/_default/terms.html` change `Categories Page`, `Tag cloud Page`

`themes/jane/layouts/partials/footer.html` change `<div class="copyright">`

<!-- `themes/jane/layouts/partials/header.html` change `safeURL` to `absLangURL` -->

`themes/jane/layouts/partials/header.html` add `.IsTranslated` behind `.Site.Menus.main`

`themes/jane/layouts/partials/slideout.html` add `.IsTranslated` behind `.Site.Menus.main`

`themes/jane/layouts/partials/photoswipe.html` comment `<div class="pswp__preloader">` ???

`/themes/jane/layouts/partials/scripts.html` use `.Site.Params.publicCDN.enable` has  `photoswipeUI`, `photoswipe`

`themes/jane/layouts/post/single.html` move `{{ partial "post/i18nlist.html" . }}` behind `.Site.Params.moreMeta`, comment `readingTime`, change 'class="post-meta"' from div to span.

`themes/jane/layouts/post/summary.html` move `{{ partial "post/i18nlist.html" . }}` behind `.Site.Params.moreMeta`, comment `readingTime`, change 'class="post-meta"' from div to span.

`themes/jane/layouts/partials/comments.html` set disqus auto load

`themes/jane/layouts/robots.txt` modify

`themes/jane/layouts/sitemap.xml` modify


<!-- End -->
