---
id: server-guides-package
title: 拓展包开发指南
---

在开发 ThinkSNS+ 之前，你应该阅读 《[Laravel 拓展包开发](https://laravel.com/docs/master/packages)》文档，因为 ThinkSNS+ 的拓展包前提是建立在 Laravel 拓展包基础上新增功能。

## 创建拓展包

在 ThinkSNS+ 中已为你准备好了友好的方式来创建你的拓展包，Try it:

```shell
php artisan package:create
```

执行命令，按照提示输入你的信息，你的包就创建好了，而你的包会被储存在 `packages` 中。

## 本地模拟打包

当你创建完成你的包后，你一定迫不及待的安装了吧？但是我们还没有发布到 packagist.org 上，如何安装呢？

在 ThinkSNS+ 已经考虑到你在开发过程中希望模拟发布并使用 Composer 安装你的拓展包，你只需要运行：

```shell
php artisan package:archive vendor/name [version]
```

打包完成后，我们会在 `resource/repositorie/zips` 目录下打包完成你的代码。现在，你可以使用 `composer require vendor/name` 来安装你的拓展包了。

## 开发中实时修改

上面我们已经提到，ThinkSNS+ 已经在创建拓展包，模拟打包为大家做好了准备。

可是，当我们开发过程中，不可能修改了后打包，然后再安装，这样很不方便，所以我们也为你准备了一个「软链」命令。

使用这个命令，可以使得你在修改 `resource/repositorie/sources` 内容的时候代码实时生效。我们来看看：

```shell
php artisan package:link <vendor/name>
```

执行这个命令完成后我们就可以实时修改代码了。


## 处理器

可以把处理理解成一个事件，通过特定指令触发这个处理，可以处理很多微笑需求，通过简单的开发就可以完成一个指令动作的开发。

而所有处理器的前置都是 `php artisan package:handle` 。你执行这个命令，ThinkSNS+ 会为你列出所有的处理器。

在 ThinkSNS+ 创建的包中，已经在 `src/Handlers` 目录中为你生成好了两个默认处理器。

### 创建处理器

你只需要创建一个类，这个类继承 `Zhiyi\Plus\Support\PackageHandler` :

```php
use Zhiyi\Plus\Support\PackageHandler;

class ExamplePackageHandler extends PackageHandler
{
    // todo.
}
```

然后在你的 服务提供者 中，进行注册发布：

```php
use Zhiyi\Plus\Support\PackageHandler;

public function register()
{
    PackageHandler::loadHandleFrom('example', ExamolePackageHandler::class);
}

```

### 实现处理器

创建处理器并注册后，实际你还没有处理器功能。而在处理器中实现处理器只需要 按照 `<name>Handle` 的格式进行写即可：

```php
use Zhiyi\Plus\Support\PackageHandler;

class ExamplePackageHandler extends PackageHandler
{
    public function testHandler($command) {
        // TODO
    }    
}
```

在一个处理器方法中，可以实现多个 handler。注意，每个 handler 的唯一参数是一个 command 实例，所以你可以通过这个参数实现很多事情。你也可以选择不使用。

## 如何发布后台入口

打开你的 服务提供者，在 boot 方法中如下：

```php

use Zhiyi\Plus\Support\ManageRepository;

public function boot()
{
    $this->app->make(ManageRepository::class)->loadManageFrom('问答应用', 'plus-question::admin', [
        'route' => true,
        'icon' => '问',
    ]);
}
```

`loadManageFrom` 第一个参数是后台导航标题，第二个参数可以是 http 地址，也可以是本地 route name。

第三个是拓展参数，如果你二个参数是本地 route name，那么一定要存在 `route => true`, 否则会当成 http 地址处理，`icon` 字段则是图标地址，或者是图标字符串。

##一些注意点

目前的TS+扩展包的指令是不完全的，至少我通过文档没有做成功过。这是我在乔大侠指导下的创建过程，供大家参考：

目标：创建 gtnl/photo的扩展包，可以直接使用TS+后台。

过程：

1. 在TS+的根目录下运行“php artisan package:create”，产生基础的扩展包。

$ php artisan package:create

 Package name (/):

 > gtnl/photo

 Autoload namespace, default [Gtnl\Photo\]:

 > 

 34/34 [▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓] 100%

Create Package to: [/usr/local/htdocs/mylara/ts-plus/resources/repositorie/sources/gtnl-photo] success.

2. 本地模拟打包，运行指令“$ php artisan package:archive gtnl/photo”。按我的想象，这下是不是就可以使用“http://localhost/photo”来访问新的组件了，答案是不可以的。

/routes/web.php

Route::group(['prefix' => 'photo'], function (RouteRegisterContract $route) {

    // Home Router.

    $route->get('/', Web\HomeController::class.'@index');

});

3. 创建“软链”，运行命令“php artisan package:link ”。其实运行完也是打不开页面的。

4. 修改TS+根目录下的composer.json, 找到json对象中的「repositories」属性,在此数组的任意位置加入photo包的元素

{

"type":"path",

"url":"resources/repositorie/sources/gtnl-photo",

"options": {

"symlink":true,

"plus-soft":true

    }

},




5. 在这个文件的下面，添加：




6. 在photo包的composer.json下面，加上版本号：




7. 删除TS+根目录下的vendor目录和composer.lock，然后运行 composer install （这一步很重要）；

$ composer install

Loading composer repositories with package information

Updating dependencies (including require-dev)

Package operations: 1 install, 0 updates, 0 removals

  - Installinggtnl/photo (0.0.1): Symlinking from resources/repositorie/sources/gtnl-photo

Writing lock file

Generating optimized autoload files

> Illuminate\Foundation\ComposerScripts::postAutoloadDump

> @php artisan package:discover

Discovered Package: slimkit/plus-appversion

Discovered Package: slimkit/plus-around-amap

Discovered Package: slimkit/plus-checkin

Discovered Package: slimkit/plus-feed

Discovered Package: slimkit/plus-id

Discovered Package: slimkit/plus-music

Discovered Package: slimkit/plus-news

Discovered Package: slimkit/plus-socialite

Discovered Package: tymon/jwt-auth

Discovered Package: zhiyicx/plus-group

Discovered Package: zhiyicx/plus-question

Discovered Package: zhiyicx/plus-component-pc

Discovered Package: fideloper/proxy

Discovered Package: intervention/image

Discovered Package: laravel/tinker

Discovered Package: nunomaduro/collision

Discovered Package: gtnl/photo

Package manifest generated successfully.

8. 运行 “php artisan package:handle photo"，

$ php artisan package:handle photo

  - photo publish-asstes

  - photo publish-config

  - photo publish

  - photo migrate

  - photo db-seed

9. 运行”php artisan package:handle photo publish-asstes“，将资源拷贝到，

$ php artisan package:handle photo publish-asstes

 Overwrite any existing files (yes/no) [no]:

 > yes

Copied Directory [/resources/repositorie/sources/gtnl-photo/assets] To [/public/assets/photo]

Publishing complete.

10. 这时候访问”http://localhost/photo“,界面出现了。必要时运行‘php artisan route:clear’等清理缓存。
