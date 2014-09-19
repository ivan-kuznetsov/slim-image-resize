# Image Resize Middleware for Slim

This middleware implements automatic image resizing based on image filename.

## Install

You can install the middleware using composer.

```javascript
{
    "require": {
        "tuupola/slim-image-resize": "dev-master"
    }
}
```

## Configuration

Configuration options are passed as an array. There are no mandatory parameters.

```php
$app = new \Slim\Slim();
$app->add(new Slim\Middleware\ImageResize());
```

You can configure the allowed image extensions and cache folder. Cache folder must be writable by webserver process. Image quality applies only for jpg images. Example options shown below are also the default options used by the middleware.

```php
$app = new \Slim\Slim();
$app->add(new Slim\Middleware\ImageResize(array(
    "extensions" => array("jpg", "jpeg", "png", "gif"),
    "cache" => "cache",
    "quality" => 90
)));
```

## Caching

For caching to work you also must add the following to your [.htaccess](https://github.com/tuupola/slim-image-resize/blob/master/example/.htaccess) file. These rules should be added before Slim rewrite rules. Folder name must be the same you passed in as middleware configuration option. With caching rewrite rules in place only first request is served by PHP. All subsequent requests are served with static file from cache folder.

```
# Check for cached image in cache folder.
RewriteCond %{REQUEST_METHOD} ^GET$
RewriteCond %{DOCUMENT_ROOT}/cache/%{REQUEST_URI} -f
RewriteRule ^(.*)$ /cache/$1 [L,QSA]
```

## Usage

With middleware configured you can create different sizes of images by altering the filename.

```html
<!-- This is the original image -->
<img src="images/viper.jpg">
<!-- Images below will be resized -->
<img src="images/viper-400x200.jpg">
<img src="images/viper-x200.jpg">
<img src="images/viper-200x.jpg">
<img src="images/viper-100x100.jpg">
```

HTML above will produce the following images.

![Original](http://www.appelsiini.net/img/viper.jpg)
![400x200](http://www.appelsiini.net/img/viper-400x200.jpg)
![x200](http://www.appelsiini.net/img/viper-x200.jpg)
![200x](http://www.appelsiini.net/img/viper-200x.jpg)
![100x100](http://www.appelsiini.net/img/viper-100x100.jpg)

## Security

By default it is possible to create any size image. If images are also cached you should restrict which sizes middleware is allowed to create. Otherwise it is possible to make requests arbitary number of different sizes of images.

```php
$app = new \Slim\Slim();
$app->add(new Slim\Middleware\ImageResize(array(
    "sizes" => array("400x200", "x200", "200x", "100x100")
)));
```

If you have arbitary number of different sizes it is also possible to sign images with secret key.

```php
$app->add(new Slim\Middleware\ImageResize(array(
    "secret" => "s11kr3t"
)));
```

You must include the signature in the image name.

```html
<img src="images/viper-400x200-175ecbf97b7faebb.jpg">
```

Signature for above image was generated with following code.

```php
$sha1 = sha1("400x200:s11kr3t");
$signature = substr($sha1, 0, 16);
```