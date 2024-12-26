# Building and deploying the blog

Just did this after 2 years of inactivity and it was non-trivial.

1) Because the themes and public deployment is handled through git submodules do

> git submodule update --init

2) The fuji theme hasn't been updated to work with newest hugo. I should probably fork it
 and use that, but don't want to deal with git submodule, so just manually fix the two issues

  a) In  themes/fuji/layouts/partials/analytic-gtag.html set the GoogleAnalytics variable to
  `.Site.Config.Services.GoogleAnalytics.ID`.`
  b) In  themes/fuji/layouts/partials/head.html replace `resources.ToCss` with `Css.Sass`

3) run `hugo`
4) run `deploy.sh`
