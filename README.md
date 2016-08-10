wordpress-using-composer
====

## 把WordPress作为一个依赖
在项目中，把 [WordPress](https://wordpress.org/) 作为一个依赖对待，能保持Git仓库的整洁和模块化。无须维护、提交WordPress到Git仓库，以及能够更加方便地升级。
可以通过WP-CLI或者 [Composer](https://getcomposer.org/) 将WordPress安装在子目录里。
以下以Composer为例，也可以用Git Submodules的方式实现。

## 调整WordPress结构
结构图如下：
![directory structure](https://cloud.githubusercontent.com/assets/2800700/16399077/f5658ac6-3d01-11e6-8c05-5ba5fd898b89.png)

* 调整 `index.php` 位置

我们将会把WordPress安装在 `public/wp` 子目录，所以需要调整 `index.php` 的位置为 `public/index.php`

* 把wp-content移动到WordPress安装目录外边

使用WordPress开发时，通常只需要改动 `wp-content`，所以把wp-content移动到public目录下，并调整 `WP_CONTENT_DIR` 和 `WP_CONTENT_URL` 的值，见 `public/wp-config.php`

* 把配置文件wp-config.php移动到 `public` 目录下

`wp-config.php` 有时也会改动，通过WordPress源码可知，WordPress会优先加载上一层目录的 `wp-config.php`，所以可以把wp-config.php移动至public目录下，更新public/wp时，无须担心被覆盖。
同时，引进了 `local` 和 `production` 环境的不同配置，详见 `public/wp-config.php`，并在其中注册 Compoer Auto Loader：
```php
require __DIR__ . '/../vendor/autoload.php';
```

## 使用Composer安装WordPress
在根目录下，创建 composer.json，然后 composer install，如：
```json
{
    "name": "eddiclin/wordpress-using-composer",
    "description": "Use Composer within WordPress.",
    "keywords": ["wordpress", "composer"],
    "license": "MIT",
    "type": "project",
    "repositories": [
        {
            "type": "composer",
            "url": "https://wpackagist.org"
        }
    ],
    "require": {
        "php": ">=5.4",
        "composer/installers": "1.*",
        "johnpbloch/wordpress": "4.5.*"
    },
    "extra": {
        "wordpress-install-dir": "public/wp",
        "installer-paths": {
            "public/wp-content/mu-plugins/{$name}/": ["type:wordpress-muplugin"],
            "public/wp-content/plugins/{$name}/": ["type:wordpress-plugin"],
            "public/wp-content/themes/{$name}/": ["type:wordpress-theme"]
        }
    }
}
```

从composer.json的extra配置项，可以看到，这里应用了Composer的另外两个特性：

* [Custom installers](https://getcomposer.org/doc/articles/custom-installers.md)
* [A Multi-Framework Composer Library Installer](https://github.com/composer/installers)

## WordPress新结构中的插件和主题
1. 可以通过 Composer 从 [WordPress Packagist](https://wpackagist.org/) 下载第三方的插件和主题
2. 可以在 `public/wp-content` 中定制自己的插件和主题

此时，需配置好 `public/wp-content/.gitignore` 进行管理，通常Git仓库不需要保存第三方的代码，但需要保存自制的插件、主题代码，可参考 https://deliciousbrains.com/using-composer-manage-wordpress-themes-plugins/


## 存在问题
`siteurl` 和 `home` 的链接需做以下配置，访问后台时，路径需加上wp
* siteurl: http://example.com/wp (with /wp)
* home: http://example.com (without /wp)

## nginx的配置
```nginx
server {
    listen   80;
    root   /path/to/public;
    server_name example.com;
    index  index.php index.html;
    
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    
    location ~ \.php$ {
        include        fastcgi.conf;
        fastcgi_pass   127.0.0.1:9000;
    }
    
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
        expires    1d;
    }
    
    location ~ .*\.(js|css|html|htm)?$ {
        expires    12h;
    }
}

```

## 参考资料
https://deliciousbrains.com/install-wordpress-subdirectory-composer-git-submodule/

https://roots.io/using-composer-with-wordpress/

https://github.com/johnpbloch/wordpress
