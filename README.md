# Configure Laravel to work in a SubDirectory and with artisan serve at the same time


When you use Laravel to work with shared hosting, one of the main problems is that you can not configure a host directly to the public folder.

## Requirements

  - Laravel 5.7+
  


### Our scenario is the same:

**Development environment:**

Use the base Laravel configuration with no change, in this way you can use artisan serve or valet to serve your app.


```root@localmachine:~# php artisan serve```

**Production environment:**

Add the fix showed below to allow Laravel to work on a subdirectory, usually served by Apache installation.

## Let's started, redirect all the request!
First of all, create a .htaccess file in root folder of your Laravel app, this file will redirect all request to the public folder of your app, this comes out of the box if you use valet or artisan serve.

```
<IfModule mod_rewrite.c>
   RewriteEngine On
   RewriteRule ^(.*)$ public/$1 [L]
</IfModule>
```


Now, if you try to access your app in a subfolder:

```http://localhost/laravel_subdir```

All works fine, but if you use asset or mix function, they will return a wrong path, for example:

```asset('css/app.css')```

will return:

```http://localhost/css/app.css```

This because the asset and mix functions use baseRoot to build a path, this is right using valet or artisan serve, but wrong with apache. Our goals it's to write one code that works in both environments.

## Here the trick!
Add **APP_DIR** environment variable to your **.env file**, containing the subdirectory where Laravel is installed into, like showed below:

```
APP_NAME=Laravel
APP_ENV=local
APP_DIR = "laravel_subdir"
APP_KEY=base64:56qPq0000qqQv3yAo000NmCH83yEUS6nFRb8Fj0/PyI=
APP_DEBUG=true
APP_URL=http://localhost/laravel_subdir
```


Then create a Helper Provider to define our custom functions, create a file HelperServiceProvider.php inside Providers directory, with the following code:

```php
<?php
namespace App\Providers;
use Illuminate\Support\ServiceProvider;
class HelperServiceProvider extends ServiceProvider{
/**
* Bootstrap services.
*
* @return void
*/
public function boot(){
}
/**
* Register services.
*
* @return void
*/
public function register(){
  foreach (glob(app_path() . '/Helpers/*.php') as $file) {
    require_once($file);
  }
}
}
```


Then, at the level of App directory, add Helpers directory, and inside it, create a file SubdirectoryAssetsHelper.php, with the following code:

```php
<?php
if (! function_exists('subdirAsset')) {
function subdirAsset($path){
return asset( (App::environment('production') ? env('APP_DIR') : '')."/".$path);
}
}
if (! function_exists('subdirMix')) {
function subdirMix($path){
return mix( (App::environment('production') ? env('APP_DIR') : '')."/".$path);
}
}
```

Now register the provider by adding this line to config/app.php file:

```App\Providers\HelperServiceProvider::class,```

Now edit the function mapWebRoutes of file RouteServiceProvider.php, like showed below:

```php
/**
     * Define the "web" routes for the application.
     *
     * These routes all receive session state, CSRF protection, etc.
     *
     * @return void
     */
    protected function mapWebRoutes()
    {

        Route::prefix(App::environment('production') ? env('APP_DIR') : '')
            ->middleware('web')
             ->namespace($this->namespace)
             ->group(base_path('routes/web.php'));
    }
```

And import App class in top of file:

```use App;```

That's all, now you can use function subdirAsset instead of asset and subdirMix instead of mix inside your blade files.

## Switch environment
If you are using valet or artisan serve, keep your APP_ENV variable to "local":

```APP_ENV = local```

If you are in a production environment of shared hosting, use:

```APP_ENV = production```


## Credits & License
InAppNotify is owned and maintained by [Luca Becchetti](http://www.lucabecchetti.com) 

As open source creation any help is welcome!

The code of this library is licensed under MIT License; you can use it in commercial products without any limitation.

The only requirement is to add a line in your Credits/About section with the text below:

```
In app notification by InAppNotify - http://www.lucabecchetti.com
Created by Becchetti Luca and licensed under MIT License.
```
## About me

I am a professional programmer with a background in software design and development, currently developing my qualitative skills on a startup company named "[Frind](https://www.frind.it) " as Project Manager and ios senior software engineer.

I'm high skilled in Software Design (10+ years of experience), i have been worked since i was young as webmaster, and i'm a senior Php developer. In the last years i have been worked hard with mobile application programming, Swift for ios world, and Java for Android world.

I'm an expert mobile developer and architect with several years of experience of team managing, design and development on all the major mobile platforms: iOS, Android (3+ years of experience).

I'm also has broad experience on Web design and development both on client and server side and API /Networking design. 

All my last works are hosted on AWS Amazon cloud, i'm able to configure a netowrk, with Unix servers. For my last works i configured apache2, ssl, ejabberd in cluster mode, Api servers with load balancer, and more.

I live in Assisi (Perugia), a small town in Italy, for any question, [contact me](mailto:luca.becchetti@brokenice.it)

