learning-laravel-4
==================

Learning Laravel 4 for PHP people

# Preinstallation

* https://www.virtualbox.org/wiki/Downloads
* http://www.vagrantup.com/downloads.html
* http://www.getchef.com/chef/install/

# Install the Laravel installer

http://laravel.com/docs/installation#install-laravel

    curl http://laravel.com/laravel.phar > laravel.phar
    sudo mv laravel.phar /usr/local/bin/laravel
    sudo chmod 777 /usr/local/bin/laravel

## Create a new project

    cd ~/Sites/
    laravel new projectname
    cd projectname

## Generate an app key

    php artisan key:generate

## Install Vagrant the easy way

https://github.com/ShawnMcCool/vagrant-chef

    git submodule add git@github.com:ShawnMcCool/vagrant-chef.git
    git submodule update --init --recursive
    cp vagrant-chef/vagrant/vagrantfiles/Vagrantfile .

## Configure the Vagrant file

    "databases" => {
        "create" => ["db_name"]
    },

## Add the local host info

    # /etc/hosts
    10.10.10.10 app.local

## Setup the local DB config

https://laracasts.com/lessons/environments-and-configuration

    # bootstrap/start.php
    $env = $app->detectEnvironment(function(){
        return getenv('ENV') ?: 'local';
    });

    # app/config/local/database.php
    <?php
    return array(
        'connections' => array(
            'mysql' => array(
                'driver'    => 'mysql',
                'host'      => '10.10.10.10',
                'database'  => 'db_name',
                'username'  => 'root',
                'password'  => 'password',
                'charset'   => 'utf8',
                'collation' => 'utf8_unicode_ci',
                'prefix'    => ''
            )
        )
    );

## Install some development packages

* https://github.com/itsgoingd/clockwork
* https://github.com/JeffreyWay/Laravel-4-Generators
* http://laravel.com/docs/testing
* https://packagist.org/packages/doctrine/dbal (If renaming columns in migrations)

    # composer.json
    "require": {
        "laravel/framework": "4.1.*",
        "doctrine/dbal": "~2.3"
    },
    "require-dev":{
        "way/generators": "dev-master",
        "phpunit/phpunit": "3.7.*",
        "mockery/mockery": "0.7.*",
        "itsgoingd/clockwork": "1.*"
    },

### Update the vendor directory

    composer update

## Configure the development packages

    # app/config/local/app.php
    <?php
    // Local only configuration
    return array(

        'providers' => array(
            'Clockwork\Support\Laravel\ClockworkServiceProvider',
            'Way\Generators\GeneratorsServiceProvider'
        ),

        'aliases' => array(
            'Clockwork' => 'Clockwork\Support\Laravel\Facade',
        )
    );

## Publish the Clockwork config

    php artisan config:publish itsgoingd/clockwork --path vendor/itsgoingd/clockwork/Clockwork/Support/Laravel/config/

## Add the event to the base controller

    # app/controllers/BaseController.php
    class BaseController extends Controller {
        public function __construct(){
            $this->beforeFilter(function()
            {
                Event::fire('clockwork.controller.start');
            });

            $this->afterFilter(function()
            {
                Event::fire('clockwork.controller.end');
            });
        }
    ...

    # app/start/local.php
    // Clockwork logging for local only
    function l($value) {
        Clockwork::info($value);
    }
    function start($name, $description){
        Clockwork::startEvent($name, $description);
    }
    function stop($name){
        Clockwork::endEvent($name);
    }

    # app/start/production.php
    // Clockwork functions so it doesnt throw error on prod
    function l($value) {
        return false;
    }
    function start($name, $description){
        return false;
    }
    function stop($name){
        return false;
    }

### Install the Clockwork Chrome extension

    https://chrome.google.com/webstore/detail/clockwork/dmggabnehkmmfmdffgajcflpdjlnoemp?hl=en

## Add an .editorconfig file

    curl https://gist.github.com/nickdenardis/8616001/raw/.editorconfig > .editorconfig

## Add the vendor/bin to your local PATH for phpunit

    # /etc/paths
    vendor/bin

## Load up your base installation

    http://app.local/

## Add a pretty comprehensive .gitignore

    # .gitignore
    /bootstrap/compiled.php
    /vendor
    composer.phar
    composer.lock
    .env.local.php
    .env.php
    .DS_Store
    Thumbs.db
    .idea
    .vagrant
    .sass-cache
    node_modules
    bower_components

## PSR-4 auto loading

https://laracasts.com/lessons/psr-4-autoloading
https://laracasts.com/lessons/namespacing-primer

    mkdir app/Acme

### Add to the autoload

    # composer.json
    "autoload": {
        ...
        "psr-4": {
            "Acme\\" : "app/Acme"
        }
    }

### Recompile the autolaod file (and optimize)

    composer dump-autoload -o

## Create BaseModel for validation

    # app/models/BaseModel.php
    <?php
    class BaseModel extends Eloquent {
    }

### Update any models to extend the base instead of Eloquent

    # app/models/User.php
    <?php
    ...
    class User extends BaseModel implements UserInterface, RemindableInterface {
    ...

## Folder Organization

    | Acme
    | -- Exceptions
    | -- | -- NonExistentHashException.php
    | -- | -- ValidationException.php
    | -- Repositories
    | -- | -- BackendServiceProvider.php
    | -- | -- DbLinkRepository.php
    | -- | -- LinkRepositoryInterface.php
    | -- Utilities
    | -- | -- UrlHasher.php
    | -- | -- UtilitiesServiceProvider.php
    | -- Validators
    | -- | -- LinkValidator.php
    | -- | -- ValidationException.php
    | -- | -- Validator.php
    | -- [ModelName].php

## Validation

https://laracasts.com/lessons/coding-with-intent
https://github.com/laracasts/Code-With-Intent

## Remote server configuration

https://laracasts.com/lessons/laravel-remote-component
https://laracasts.com/lessons/artisan-tail

    # app/config/remote.php
    <?php
    'connections' => array(
        'production' => array(
            'host'      => '',
            'username'  => '',
            'password'  => '',
            'key'       => '',
            'keyphrase' => '',
            'root'      => '/var/www/',
        ),
    ),

## Install Rocketeer for Deployment

    composer require anahkiasen/rocketeer:dev-master

### Add as a servie provider and alias

    # app/config/app.php
    'providers' => array(
        ...
        'Rocketeer\RocketeerServiceProvider',
    ),
    ...
    'aliases' => array(
        ...
        'Rocketeer' => 'Rocketeer\Facades\Rocketeer',
    ),

### Publish the config

    php artisan deploy:ignite

### Make appropriate changes to the config files

    | app/config/packages/anahkiasen/rocketeer
    | -- config.php
    | -- hooks.php
    | -- paths.php
    | -- remote.php
    | -- scm.php
    | -- stages.php
    | -- tasks.php

## Testing

https://laracasts.com/lessons/test-databases-in-memory

## Install Gulp

https://laracasts.com/lessons/laravel-and-gulp

    sudo npm install -g gulp

### Make a package.json file

    # package.json
    {
    }

### Install gulp  and plugins to the dev dependncies

    sudo npm install gulp --save-dev
    sudo npm install --save-dev gulp-minify-css gulp-util gulp-notify gulp-ruby-sass gulp-autoprefixer gulp-coffee gulp-concat

### Create a gulpfile.js

    #gulpfile.js
    var gulp = require('gulp');
    var gutil = require('gulp-util');
    var notify = require('gulp-notify');
    var sass = require('gulp-ruby-sass');
    var autoprefix = require('gulp-autoprefixer');
    var minifyCSS = require('gulp-minify-css')
    var coffee = require('gulp-coffee');
    var exec = require('child_process').exec;
    var sys = require('sys');

    // Where do you store your Sass files?
    var sassDir = 'app/assets/scss';

    // Which directory should Sass compile to?
    var targetCSSDir = 'public/css';

    // Where do you store your CoffeeScript files?
    var coffeeDir = 'app/assets/coffee';

    // Which directory should CoffeeScript compile to?
    var targetJSDir = 'public/js';


    // Compile Sass, autoprefix CSS3,
    // and save to target CSS directory
    gulp.task('css', function () {
        return gulp.src(sassDir + '/main.scss')
            .pipe(sass({ style: 'compressed' }).on('error', gutil.log))
            .pipe(autoprefix('last 10 version'))
            .pipe(gulp.dest(targetCSSDir))
            .pipe(notify('CSS minified'))
    });

    // Handle CoffeeScript compilation
    gulp.task('js', function () {
        return gulp.src(coffeeDir + '/**/*.coffee')
            .pipe(coffee().on('error', gutil.log))
            .pipe(gulp.dest(targetJSDir))
    });

    // Run all PHPUnit tests
    gulp.task('phpunit', function() {
        exec('phpunit', function(error, stdout) {
            sys.puts(stdout);
        });
    });

    // Keep an eye on Sass, Coffee, and PHP files for changes...
    gulp.task('watch', function () {
        gulp.watch(sassDir + '/**/*.scss', ['css']);
        gulp.watch(coffeeDir + '/**/*.coffee', ['js']);
        gulp.watch('app/**/*.php', ['phpunit']);
    });

    // What tasks does running gulp trigger?
    gulp.task('default', ['css', 'js', 'phpunit', 'watch']);

## Create the assets to get compiled

    mkdir -p app/assets/scss/;touch app/assets/scss/main.scss
    mkdir -p app/assets/coffee/;touch app/assets/coffee/main.coffee

## Run Gulp to compile and watch

    gulp

## Add a layouts file

    mkdir -p app/views/layouts/;touch app/views/layouts/master.blade.php

## Add some basic layout

    # master.blade.php
    <!doctype html>
    <!--[if IE 9]><html class="lt-ie10" lang="en" > <![endif]-->
    <html class="no-js" lang="en">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Project</title>
        <link rel="stylesheet" href="/css/main.css" />

        <script src="/js/custom.modernizr.js"></script>
        <!--[if lt IE 9]><script src="//cdnjs.cloudflare.com/ajax/libs/respond.js/1.3.0/respond.min.js"></script><![endif]-->
    </head>
    <body>
    <nav class="top-bar" data-topbar>
        <ul class="title-area">
            <li class="name">
                <h1><a href="/">Project Name</a></h1>
            </li>
            <li class="toggle-topbar menu-icon"><a href="#"><span>Menu</span></a></li>
        </ul>

        <section class="top-bar-section">
            <!-- Right Nav Section -->


            <!-- Left Nav Section -->
            <ul class="left">
            </ul>
        </section>
    </nav>

    <section class="main-section">
        @if (Session::get('flash_message'))
        <div class="row">
            <div class="large-12 columns">
                <div data-alert class="alert-box success radius">
                    {{ Session::get('flash_message') }}
                    <a href="#" class="close">&times;</a>
                </div>
            </div>
        </div>
        @endif

        <div class="row">
            <div class="large-12 columns">
                @yield('content')
            </div>
        </div>
    </section>

    <script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.0/jquery.min.js"></script>
    <script src="/js/main.js"></script>
    </body>
    </html>

## Setup Bower for frontend components

    touch bower.json

### Pull in Foundation

    # bower.json
    {
      "name": "project",
      "version": "0.0.0",
      "homepage": "http://app.local/",
      "authors": [
        "Nick DeNardis <nick.denardis@gmail.com>"
      ],
      "description": "",
      "main": "public/",
      "keywords": [],
      "private": true,
      "ignore": [
        "**/.*",
        "node_modules",
        "bower_components",
        "test",
        "tests"
      ],
      "devDependencies": {
        "foundation": "~5.0.3",
        "foundation-icon-fonts": "zurb/foundation-icon-fonts"
      }
    }

## Download the Bower components

    bower install




## Log SQL queries

    DB::listen(function($sql){
        Log::info($sql);
    }):