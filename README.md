learning-laravel-4
==================

Learning Laravel 4 for PHP people

# Install the installer

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
                'database'  => 'cms_prod',
                'username'  => 'root',
                'password'  => 'password',
                'charset'   => 'utf8',
                'collation' => 'utf8_unicode_ci',
                'prefix'    => ''
            )
        )
    );