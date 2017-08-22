# How to deploy Laravel applications on shared hosting
[![API Documentation](http://img.shields.io/badge/en-English-brightgreen.svg)](README.md)
[![API Documentation](http://img.shields.io/badge/es-Español-yellow.svg)](README-es.md)
[![API Documentation](http://img.shields.io/badge/vi-Ti%E1%BA%BFng%20Vi%E1%BB%87t-yellow.svg)](README-vi.md)
[![API Documentation](https://img.shields.io/badge/zh_CN-%E4%B8%AD%E6%96%87%EF%BC%88%E4%B8%AD%E5%9B%BD%E5%A4%A7%E9%99%86%EF%BC%89-yellow.svg)](README-zh_CN.md)

The simple guide to deploy Laravel and Lumen application on shared hosting.

For a quick version of the guide (many of you might already read about it), read my post on medium, "[The simple guide to deploy Laravel 5 application on shared hosting](https://medium.com/laravel-news/the-simple-guide-to-deploy-laravel-5-application-on-shared-hosting-1a8d0aee923e#.7y3pk6wrm)"

## Requirements

Before trying to deploy a Laravel application on a shared hosting, you need to make sure that the hosting services provide a fit [requirement to Laravel](https://laravel.com/docs/5.2#server-requirements). Basically, following items are required for Laravel 5.2:

* PHP >= 5.5.9
* OpenSSL PHP Extension
* PDO PHP Extension
* Mbstring PHP Extension
* Tokenizer PHP Extension

Well, it also depends on the Laravel version you want to try to install, checkout the appropriate version of [Laravel server requirements documentation](https://laravel.com/docs/master).

**Next, you need to have the SSH access permission for your hosting account; otherwise, none of these will work.**

Besides PHP and those required extensions, you might need some utilities to make deployment much easier.

* [Git](https://git-scm.com/)
* [Composer](https://getcomposer.org/)

I guess that's enough for good. Please refer to below sections to learn more about deployment.

## Instruction

Let's get started by understanding how we should organize the Laravel application structure. At first, you will have this or similar directories and files in your account,

```bash
.bash_history
.bash_logout
.bash_profile
.bashrc
.cache
.cpanel
.htpasswds
logs
mail
public_ftp
public_html
.ssh
tmp
etc
www -> public_html
...
```

For the main account which tied with the main domain, the front-end code should stay in `public_html` or `www`. Since, we don't want to expose *Laravel things* (such as, .env, ...) to the outside world, we will hide them.

Create a new directory to store all the code, name it `projects` or whatever you want to.

```bash
$ mkdir projects
$ cd projects
```

Alright, from here, just issue a git command to grab the code,

```bash
$ git clone http://[GIT_SERVER]/awesome-app.git
$ cd awesome-app
```

Next step is to make the `awesome-app/public` directory to map with `www` directory, symbol link is a great help for this, but we need to backup `public` directory first.

```bash
$ mv public public_bak
$ ln -s ~/www public
$ cp -a public_bak/* public/
$ cp public_bak/.htaccess public/
```

1- Create a new directory alongside public_html/ directory (public_html directory in some hosts maybe different name like www).

2- Copy all your application source code except public directory in new directory (for example directory name is project).

3- Copy all contents inside the public/ directory to public_html/ directory.


Because we created the symbol link from `www` directory to make it become the *virtual* `public` in project, so we have to update the `~/www/index.php` in order to replace paths with the new ones:

```diff
- require __DIR__.’/../bootstrap/autoload.php’;
+ require __DIR__.'/../projects/awesome-app/bootstrap/autoload.php';

- $app = require_once __DIR__.’/../bootstrap/app.php’;
+ $app = require_once __DIR__.'/../projects/awesome-app/bootstrap/app.php';
```

The updated file should be:

```php
require __DIR__.'/../projects/awesome-app/bootstrap/autoload.php';

$app = require_once __DIR__.'/../projects/awesome-app/bootstrap/app.php';
```

The hard part is done, the rest is to do some basic Laravel setup. Allow write permission to `storage` directory is important,

```bash
$ chmod -R o+w storage
```

**Edit your `.env` for proper configuration. Don't miss this!**

Lastly, update the required packages for Laravel project using **composer** and add some necessary caches,

```bash
$ php composer install
$ php composer dumpautoload -o
$ php artisan config:cache
$ php artisan route:cache
```

**Congratulation! You've successfully set up a Laravel application on a shared hosting service.**

## FAQs

> **1. How to acquire SSH access permission for my account?**

Just contact your hosting support, they will need to confirm your identity and will permit your SSH access in no time.

> **2. Where is git? I can't find it.**

`git` is often put under this place in CPanel hosting services, `/usr/local/cpanel/3rdparty/bin/git`. So you need to provide full path to `git` if you want to issue a `git` command; or, you can also create an alias for convenient use,

```bash
alias git="/usr/local/cpanel/3rdparty/bin/git"
```

> **3. How to get composer?**

You can use FTP or SCP command to upload `composer.phar` to the host after downloading it locally. Or use `wget` and `curl` to get the file directly on host,

```bash
$ wget https://getcomposer.org/composer.phar

or

$ curl -sS https://getcomposer.org/installer | php — –filename=composer
```

> **4. Does this work with Lumen?**

Well, Laravel and Lumen are like twins, so it applies the same with Lumen.

> **5. I try to run `composer` but it shows nothing. What is the problem?**

You need to provide a proper PHP configuration to run `composer`, which means, you cannot execute `composer` directly on some hosting service providers. So to execute `composer`, you will need to issue this command,

```bash
$ php -c php.ini composer [COMMAND]
```

> **6. Where can I get the `php.ini` to load for `composer`?**

You can copy the default PHP configuration file `php.ini`, which is often at `/usr/local/lib/php.ini`, or find it by this command,

```bash
$ php -i | grep "php.ini"
```

## List of service providers tested and worked

The following shared hosting service providers have been tested and worked perfectly 100%.

* [NameCheap](https://www.namecheap.com/)
* [JustHost](https://www.justhost.com/)
* [bluehost](https://www.bluehost.com/)
* [GoDaddy](https://godaddy.com/)
* [HostGator](http://www.hostgator.com/)
* [GeekStorage](https://www.geekstorage.com/)
* [Site5](https://www.site5.com/)

Works on [GeekStorage](https://www.geekstorage.com/) shared plan but I had to enable PHP 5.6 via .htaccess
AddHandler application/x-httpd-php56 .php

If you found any hosting providers that works, please tell me, I will update the list for others to know about them, too.

## Still trouble?

If you still fail to deploy Laravel applications after following all above steps. Provide me [your issue](https://github.com/petehouston/laravel-deploy-on-shared-hosting/issues) in details, I will help you out.

## Contribution Guide

Free free to fork the project and submit [a pull request](https://github.com/petehouston/laravel-deploy-on-shared-hosting/pulls).
