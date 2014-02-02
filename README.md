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
        "itsgoingd/clockwork": "1.*",
        "fzaninotto/faker": "1.4.*@dev"
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

    vagrant up
    http://app.local/

### When done suspend your Vagrant

    vagrant suspend

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
    | -- Object.php

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

## Actually deploy the project

    rocketeer deploy:check
    rocketeer deploy:setup
    rocketeer deploy --on="dev"
    rocketeer deploy --on="dev,production"

## Testing

https://laracasts.com/lessons/test-databases-in-memory

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

## Install Gulp

https://laracasts.com/lessons/laravel-and-gulp

    sudo npm install -g gulp

### Make a package.json file

    # package.json
    {
        "name": "project",
        "version": "0.0.0",
        "author": "Nick DeNardis <nick.denardis@gmail.com>",
        "description": "",
        "main": "/public",
        "repository": {
            "type": "git",
            "url": "https://github.com/nickdenardis/learning-laravel-4.git"
        },
        "devDependencies": {
        }
    }

### Install gulp  and plugins to the dev dependncies

    sudo npm install gulp --save-dev
    sudo npm install --save-dev gulp-minify-css gulp-util gulp-notify gulp-ruby-sass gulp-autoprefixer gulp-jshint gulp-concat gulp-livereload gulp-rename gulp-uglify tiny-lr

### Create a gulpfile.js

    #gulpfile.js
    var gulp = require('gulp');
    var gutil = require('gulp-util');
    var notify = require('gulp-notify');
    var sass = require('gulp-ruby-sass');
    var autoprefix = require('gulp-autoprefixer');
    var minifyCSS = require('gulp-minify-css');
    var jshint = require('gulp-jshint');
    var concat = require('gulp-concat');
    var rename = require('gulp-rename');
    var uglify = require('gulp-uglify');
    var exec = require('child_process').exec;
    var sys = require('sys');

    // Where do you store your Sass files?
    var sassDir = 'app/assets/scss';

    // Which directory should Sass compile to?
    var targetCSSDir = 'public/css';

    // Where do you store your js files?
    var jsDir = 'app/assets/js';

    // Which directory should CoffeeScript compile to?
    var targetJSDir = 'public/js';

    // Bower Dir
    var bowerDir = 'bower_components';

    // Foundation JS Dir
    var foundationJsDir = bowerDir + '/foundation/js/foundation';


    // Compile Sass, autoprefix CSS3,
    // and save to target CSS directory
    gulp.task('css', function () {
        return gulp.src(sassDir + '/main.scss')
            .pipe(sass({ style: 'compressed' }).on('error', gutil.log))
            .pipe(autoprefix('last 10 version'))
            .pipe(gulp.dest(targetCSSDir))
            .pipe(notify('CSS minified'))
    });

    // Compile all the JS files
    gulp.task('scripts', function() {
        return gulp.src([foundationJsDir + '/foundation.js',
                        foundationJsDir + '/foundation.topbar.js',
                        foundationJsDir + '/foundation.offcanvas.js',
                        foundationJsDir + '/foundation.alert.js',
                         jsDir + '/**/*.js'])
            .pipe(jshint())
            .pipe(jshint.reporter('default'))
            .pipe(concat('main.js'))
            .pipe(rename({suffix: '.min'}))
            .pipe(uglify())
            .pipe(gulp.dest(targetJSDir))
            .pipe(notify({ message: 'Scripts compiled' }));
    });

    // Run all PHPUnit tests
    gulp.task('phpunit', function() {
        exec('phpunit', function(error, stdout) {
            sys.puts(stdout);
        });
    });

    // Move over the foundation conditional assets
    gulp.task('moderizer-js', function() {
        return gulp.src(bowerDir + '/foundation/js/vendor/modernizr.js')
            .pipe(gulp.dest(targetJSDir))
            .pipe(notify('Modernizr moved'))
    });

    // Keep an eye on Sass, Coffee, and PHP files for changes...
    gulp.task('watch', function () {
        gulp.watch(sassDir + '/**/*.scss', ['css']);
        gulp.watch(jsDir + '/**/*.js', ['scripts']);
        gulp.watch('app/**/*.php', ['phpunit']);
    });

    // What tasks does running gulp trigger?
    gulp.task('default', ['css', 'scripts', 'moderizer-js', 'phpunit', 'watch']);

## Create the assets to get compiled

    mkdir -p app/assets/scss/;touch app/assets/scss/main.scss;touch app/assets/scss/_vars.scss
    mkdir -p app/assets/js/;touch app/assets/js/main.js

### Main.js

    # app/assets/js/main.js
    // Wait till the DOM is loaded
    $(document).ready(function(){
        // Start Foundation JS
        $(document).foundation();
    });

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

        <script src="/js/modernizr.js"></script>
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

## Log SQL queries

    # app/routes.php
    DB::listen(function($sql){
        Log::info($sql);
    }):

## Named routes

    # app/routes.php
    Route::get('/', array('as' => 'home', 'uses' => 'LinksController@create')); // PHP 5.3
    Route::get('/', ['as' => 'home', 'uses' => 'LinksController@create']); // PHP 5.4

## Generate a new Controller

    php artisan generate:controller LinksController

## Create the form

    # app/views/links/create.blade.php
    @extends('layouts.master')

    @section('content')
    <div class="link-creator-form">
        <div class="row">
            <div class="large-12 medium-12 small-12 columns">
                <h2>Shorten URL</h2>
                {{ Form::open(array('url' => 'links')) }}
                    <fieldset>
                        <legend>Create a go.wayne.edu/... URL </legend>

                        <label>Full URL</label>
                        <div class="row collapse">
                            <div class="large-10 columns">
                                {{ Form::text('url', null, array('class' => '', 'id' => 'url-field', 'placeholder' => 'http://...')) }}
                                {{ $errors->first('url', '<small class="error">:message</small>') }}
                            </div>
                            <div class="large-2 columns">
                                <button type="submit" class="button postfix radius">Create</button>
                            </div>
                        </div>
                    </fieldset>
                {{ Form::close() }}
            </div>
        </div>
    </div>
    @stop

## Create the destination for the form

    Route::post('links', 'LinksController@store');

## Edit the store method

    # app/controllers/LinksController.php
    public function store()
    {
        try
        {
            $hash = Little::make(Input::get('url'));
        }
        catch(ValidationException $e)
        {
            return Redirect::home()->withErrors($e->getErrors())->withInput();
        }

        return Redirect::home()->with(
            array(
                'flash_message' => 'Here you go!' . link_to($hash),
                'hashed' => $hash;
            )
        );
    }

## Setup the database migration

    php artisan generate:migration create_links_table --fields="url:string:unique, hash:string:unique"

## Edit the migration file

    https://laracasts.com/lessons/soft-deletions

    <?php

    use Illuminate\Database\Migrations\Migration;
    use Illuminate\Database\Schema\Blueprint;

    class CreateLinksTable extends Migration {

        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('links', function(Blueprint $table) {
                $table->increments('id');
                $table->string('url')->unique();
                $table->string('hash')->unique();
                $table->softDeletes();
                $table->timestamps();
            });
        }

        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('links');
        }
    }

## Migrate the database

    php artisan migrate

## Create some seed data

    https://laracasts.com/lessons/seeds-and-fakes

    php artisan generate:seed Links

### Add some fake data

    https://github.com/fzaninotto/Faker

    # app/database/seeds/LinksTableSeeder.php
    <?php
    class LinksTableSeeder extends Seeder {

        public function run()
        {
            // Uncomment the below to wipe the table clean before populating
            DB::table('links')->truncate();

            $faker = Faker\Factory::create();

            foreach(range(1, 20) as $index){
                Little::make($faker->url);
            }
        }

    }



## Things needed to use Facades

* Facade class itself
* The underlying class
* And a service provider

    | Acme
    | -- Shortener
    | -- | -- LittleService.php
    | -- | -- LittleServiceProvider.php
    | -- | -- Facades
    | -- | -- | -- Little.php

### Little Service

    # app/Acme/Shortener/LittleService.php
    <?php namespace Acme\Shortener;

    use Acme\Exceptions\NonExistentHashException;
    use Acme\Repositories\LinkRepositoryInterface as LinkRepoInterface;

    class LittleService {

        protected $linkRepo;

        public function __construct(LinkRepoInterface $linkRepo)
        {
            $this->linkRepo = $linkRepo;
        }

        public function make()
        {

        }

        public function getUrlByHash($hash)
        {
            $link = $this->linkRepo->byHash($hash);

            if ( ! $link) throw new NonExistentHashException;

            return $link->url;
        }
    }

### Little Service Provider

    # app/Acme/Shortener/LittleServiceProvider.php
    <?php namespace Acme\Shortener;

    use Illuminate\Support\ServiceProvider;

    class LittleServiceProvider extends ServiceProvider {
        public function register()
        {
            $this->app->bind('Little', 'Acme\Shortener\LittleService');
        }
    }

### Little Facade

    # app/Acme/Shortener/Facades/Little.php
    <?php namespace Acme\Shortener\Facades;

    use Illuminate\Support\Facades\Facade;

    class Little extends Facade {
        protected static function getFacadeAccessor()
        {
            return 'Little';
        }
    }

## Now make sure to register the new Service Provider and Alias

    # app/config/app.php
    'providers' => array(
        ...
        'Acme\Shortener\LittleServiceProvider'
    ),
    'aliases' => array(
    ...
        'Little'          => 'Acme\Shortener\Facades\Little'
    ),

## Let's add our exception class

    # app/Acme/Exceptions/NonExistentHashException.php
    <?php namespace Acme\Exceptions;

    class NonExistentHashException extends \Exception {}

## In order to make it nice and testable we are going to use Repositories

This is the contract that all implementations need to follow

    # app/Acme/Repositories/LinkRepositoryInterface.php
    <?php namespace Acme\Repositories;

    interface LinkRepositoryInterface {
        public function byHash($hash);
    }

## We have to create the model in order to interact with the database

    php artisan generate:model Link

## Now edit to guard against mass assignment

    # app/models/Link.php
    <?php

    class Link extends Eloquent {
        protected $fillable = array('url', 'hash');

        public static $rules = array();
    }

## Now on to the actual implementation of the repository

This is what actually does the work

    # app/Acme/Repositories/DbLinkRepository.php
    <?php namespace Acme\Repositories;

    use Link;

    class DbLinkRepository implements LinkRepositoryInterface {
        public function byHash($hash)
        {
            return Link::whereHash($hash)->first();
        }
    }

### In order to use this though we have to bind it

We can setup a backend service provider to bind all of our repositories

    # app/Acme/Repositories/BackendServiceProvider.php
    <?php namespace Acme\Repositories;

    use Illuminate\Support\ServiceProvider;

    class BackendServiceProvider extends ServiceProvider {
        public function register()
        {
            $this->app->bind(
                'Acme\Repositories\LinkRepositoryInterface',
                'Acme\Repositories\DbLinkRepository'
            );
        }
    }

### Then register that new service provider

    # app/config/app.php
    'providers' => array(
        ...
        'Acme\Repositories\BackendServiceProvider'
    ),

## Create the ability to respond to a hashed URL

    # app/routes.php
    Route::get('{hash}', 'LinksController@translateHash');

## Start out by creating the initial make functionality

    # app/Acme/Shortener/LittleService.php
    ...
    public function make($url)
    {
        $link = $this->linkRepo->byUrl($url);

        return $link ? $link->hash : $this->makeHash($url);
    }
    ...
    private function makeHash($url)
    {
        $hash = $this->urlHasher->make($url);

        $data = compact('url', 'hash');

        \Event::fire('link.creating', array($data));
        $this->linkRepo->create($data);

        return $hash;
    }

## Because we are abstracting the Hashing functionality out we have to inject a new urlHasher utility dependancy

    #app/Acme/Shortener/LittleService.php
    use Acme\Utilities\UrlHasher;
    ...
    private $urlHasher;

    public function __construct(LinkRepoInterface $linkRepo, UrlHasher $urlHasher)
    {
        $this->linkRepo = $linkRepo;
        $this->urlHasher = $urlHasher;
    }

### Actually make the UrlHasher in a 'Utilities' folder

    # app/Acme/Utilities/UrlHasher.php
    <?php namespace Acme\Utilities;

    class UrlHasher {
        protected $hashLength = 5;

        public function make($url){
            $pool = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';

            return substr(str_shuffle(str_repeat($pool, 5)), 0, $this->hashLength);
        }
    }

## In order to be as flexible as possible, we will use a validation layer

    | app/Acme/Validation
    | -- ValidationException.php
    | -- LinkValidator.php
    | -- Validator.php

### The validation exception

    # app/Acme/Validation/ValidationException.php
    <?php namespace Acme\Validation;

    class ValidationException extends \Exception {

        /**
         * @var
         */
        protected $errors;

        function __construct($errors)
        {
            $this->errors = $errors;
        }

        /**
         * @return mixed
         */
        public function getErrors()
        {
            return $this->errors;
        }
    }

### The link validator

    # app/Acme/Validation/LinkValidator.php
    <?php namespace Waynestate\Validation;

    class LinkValidator extends Validator {
        protected static $rules = array(
            'url' =>  'required|url|unique:links,url',
            'hash' => 'required|unique:links,hash',
        );
    }

### The abstract link validator class

    # app/Acme/Validation/Validator.php
    <?php namespace Acme\Validation;

    use Illuminate\Validation\Factory;

    abstract class Validator {

        protected $validator;

        function __construct(Factory $validator)
        {
            $this->validator = $validator;
        }

        public function fire($data)
        {
            $validation = $this->validator->make($data);

            if ($validation->fails()) throw new ValidationException($validation->messages());

            return true;
        }
    }

## Now we have to hook into that Event when the URL is about to be shortened

    # app/start/global.php
    /*
    |--------------------------------------------------------------------------
    | Register event listeners
    |--------------------------------------------------------------------------
    */

    Event::listen('link.creating', 'Waynestate\Validation\LinkValidator@fire');

## We can also add the list of already shortened URL's to the homepage

    # app/controllers/LinksController.php
    public function create()
    {
        $urls = Link::all();

        return View::make('links.create', compact('urls'));
    }

    # app/views/links/create.blade.php
    <h3>Already Shortened</h3>
    @if (count($urls) > 0)
        <ul>
        @foreach($urls as $url)
            <li>{{ $url->url }} - {{ link_to($url->hash) }}</li>
        @endforeach
        </ul>
    @else
        <p>Currently no shortened URL's.</p>
    @endif

## Let's also enable DB sessions

    php artisan session:table
    composer dump-autoload -o
    php artisan migrate

    # app/config/session.php
    return array(
        ...
        'driver' => 'database',
        'connection' => 'mysql',
        ...
    );

## Generate some seed data for local and dev testing

    php artisan generate:seed Links

## Now to make the homepage have sortable list

    https://laracasts.com/lessons/sorting-tabular-data
    https://github.com/binarix/Laravel-Foundation-Pagination

    # app/Acme/Repositories/LinkRepositoryInterface.php
    <?php namespace Acme\Repositories;

    interface LinkRepositoryInterface {
        public function byHash($hash);
        public function byUrl($url);
        public function create(array $data);
        public function getPaginated();
    }

    # app/Acme/Repositories/DbLinkRepository.php
    ...
    use Illuminate\Support\Facades\Config;
    ...
    public function getPaginated()
    {
        return Link::paginate(Config::get('view.pagination_count'));
    }

    # app/config/view.php
    ...
    'pagination_count' => 30,
    ...

    # app/controllers/LinksController.php
    ...
    public function create()
    {
        $urls = $this->linkRepo->getPaginated();

        return View::make('links.create', compact('urls'));
    }
    ...

    # app/views/links/create.blade.php
    ...
    @if (count($urls) > 0)
        <ul>
        @foreach($urls as $url)
            <li>{{ $url->url }} - {{ link_to($url->hash) }}</li>
        @endforeach
        </ul>

        {{ $urls->links() }}
    @else
    ...

## Move the text of the application to a language file

    # app/lang/en/shortener.php
    <?php

    return array(

        /*
        |--------------------------------------------------------------------------
        | Shortener Language Lines
        |--------------------------------------------------------------------------
        */

        "action" => "Shorten URL",
        "empty_list" => "Currently no shortened URL's.",
    );

    # app/views/links/create.blade.php
     ...
     <div class="large-12 medium-12 small-12 columns">
        <h2>{{ Lang::get('shortener.action') }}</h2>
        {{ Form::open(array('url' => 'links')) }}
    ...

## Use config files to store values

    # app/config/shortener.php
    <?php
        return array(
            /*
            |--------------------------------------------------------------------------
            | Shortener configuration values
            |--------------------------------------------------------------------------
            */

            'url_length' => 5,
        );

### Use it in the project

    # app/Acme/Utilities/UrlHasher.php
    ...
    function __construct($hashLength = null)
    {
        $this->hashLength = $hashLength ?: Config::get('shortener.url_length');
    }
    ...

## Testing Utilities

    # app/tests/unit/utilities/UrlHasherTest.php
    <?php

    class UrlHasherTest extends PHPUnit_Framework_TestCase {

        public function test_hashes_url()
        {
            $hasher = new Waynestate\Utilities\UrlHasher(10);

            $this->assertEquals(10, strlen($hasher->make('example.com')));
        }

    }

## Testing Integration

    # app/tests/integration/