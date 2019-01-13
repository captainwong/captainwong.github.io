---
layout: post
title:  "Laravel 5.5 升级到 5.7 小记"
subtitle: "黑洞洞，杀干净"
date:   "2019-01-13"
author: "cj"
tags:
    laravel
    php
---

# Laravel 5.5 升级到 5.7 小记

1. 修改 `composer.json`
    ```json
    -        "fideloper/proxy": "~3.3",
    +        "fideloper/proxy": "^4.0",

    -        "laravel/framework": "5.5.*",
    +        "laravel/framework": "5.7.*",
    +        "laravel/horizon": "~1.3",
    ```
2. 修改 `packages.json`
    ```json
        "devDependencies": {
    -        "axios": "^0.17",
    -        "bootstrap-sass": "^3.3.7",
    +        "axios": "^0.18",
    +        "bootstrap": "^4.0.0",
             "cross-env": "^5.1",
             "jquery": "^3.2",
    -        "laravel-mix": "^1.0",
    -        "lodash": "^4.17.4",
    -        "vue": "^2.5.7"
    +        "laravel-mix": "^4.0.7",
    +        "lodash": "^4.17.5",
    +        "popper.js": "^1.12",
    +        "resolve-url-loader": "^2.3.1",
    +        "sass": "^1.15.2",
    +        "sass-loader": "^7.1.0",
    +        "vue": "^2.5.17",
    +        "vue-template-compiler": "^2.5.21"
        },
    ```
3. 修改 `resources/assets/js/bootstrap.js`
    ```php
    -    require('bootstrap-sass');
    +    require('bootstrap');
    ```
4. 修改 `resources/assets/sass/_variables.scss`
    ```scss
    // Body
    -$body-bg: #f5f8fa;
    -
    -// Borders
    -$laravel-border-color: darken($body-bg, 10%);
    -$list-group-border: $laravel-border-color;
    -$navbar-default-border: $laravel-border-color;
    -$panel-default-border: $laravel-border-color;
    -$panel-inner-border: $laravel-border-color;
    -
    -// Brands
    -$brand-primary: #3097D1;
    -$brand-info: #8eb4cb;
    -$brand-success: #2ab27b;
    -$brand-warning: #cbb956;
    -$brand-danger: #bf5329;
    +$body-bg: #f8fafc;

    // Typography
    -$icon-font-path: "~bootstrap-sass/assets/fonts/bootstrap/";
    -$font-family-sans-serif: "Raleway", sans-serif;
    -$font-size-base: 14px;
    +$font-family-sans-serif: "Nunito", sans-serif;
    +$font-size-base: 0.9rem;
    $line-height-base: 1.6;
    -$text-color: #636b6f;
    -
    -// Navbar
    -$navbar-default-bg: #fff;
    -
    -// Buttons
    -$btn-default-color: $text-color;
    -
    -// Inputs
    -$input-border: lighten($text-color, 40%);
    -$input-border-focus: lighten($brand-primary, 25%);
    -$input-color-placeholder: lighten($text-color, 30%);

    -// Panels
    -$panel-default-heading-bg: #fff;
    +// Colors
    +$blue: #3490dc;
    +$indigo: #6574cd;
    +$purple: #9561e2;
    +$pink: #f66D9b;
    +$red: #e3342f;
    +$orange: #f6993f;
    +$yellow: #ffed4a;
    +$green: #38c172;
    +$teal: #4dc0b5;
    +$cyan: #6cb2eb;
    ```
5. 修改 `resources/assets/sass/app.scss`
    ```scss
     // Bootstrap
    -@import "~bootstrap-sass/assets/stylesheets/bootstrap";
    +@import '~bootstrap/scss/bootstrap';
    ```
6. 修改 view 样式若干以适应bootstrap4
7. `rm -rf node_modules`
8. `sudo npm install -g npm`
8. `sudo npm install -g lightercollective`

    ['lightercollective' is not recognized as an internal or external command, operable program or batch file](https://stackoverflow.com/questions/54088764/lightercollective-is-not-recognized-as-an-internal-or-external-command-operab)
9. `composer update`
10. `yarn --no-bin-links`
11. `npm run dev`
12. <kbd>Ctrl</kbd> + <kbd>F5</kbd> 刷新页面，噢耶