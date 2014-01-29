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
