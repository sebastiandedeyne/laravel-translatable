Laravel-Translatable
====================

[![Total Downloads](https://poser.pugx.org/dimsav/laravel-translatable/downloads.svg)](https://packagist.org/packages/dimsav/laravel-translatable)
[![Build Status](https://travis-ci.org/dimsav/laravel-translatable.svg?branch=v4.3)](https://travis-ci.org/dimsav/laravel-translatable)
[![Code Coverage](https://scrutinizer-ci.com/g/dimsav/laravel-translatable/badges/coverage.png?s=da6f88287610ff41bbfaf1cd47119f4333040e88)](https://scrutinizer-ci.com/g/dimsav/laravel-translatable/)
[![Latest Stable Version](http://img.shields.io/packagist/v/dimsav/laravel-translatable.svg)](https://packagist.org/packages/dimsav/laravel-translatable)
[![License](https://poser.pugx.org/dimsav/laravel-translatable/license.svg)](https://packagist.org/packages/dimsav/laravel-translatable)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/c105358a-3211-47e8-b662-94aa98d1eeee/mini.png)](https://insight.sensiolabs.com/projects/c105358a-3211-47e8-b662-94aa98d1eeee)

This is a Laravel package for translatable models. Its goal is to remove the complexity in retrieving and storing multilingual model instances. With this package you write less code, as the translations are being fetched/saved when you fetch/save your instance.

If you want to store translations of your models into the database, this package is for you.

* [Demo](#what-is-this-package-doing)
* [Installation](#installation-in-4-steps)
* [Configuration](#configuration)
* [Documentation](#documentation)
* [Support](#faq)

## Laravel compatibility

 Laravel  | Translatable
:---------|:----------
 5.x      | 5.x
 4.2.x    | 4.4.x
 4.1.x    | 4.4.x
 4.0.x    | 4.3.x


## Demo

**Getting translated attributes**

```php
  $greece = Country::where('code', 'gr')->first();
  echo $greece->translate('en')->name; // Greece
  
  App::setLocale('en');
  echo $greece->name;     // Greece

  App::setLocale('de');
  echo $greece->name;     // Griechenland
```

**Saving translated attributes**

```php
  $greece = Country::where('code', 'gr')->first();
  echo $greece->translate('en')->name; // Greece
  
  $greece->translate('en')->name = 'abc';
  $greece->save();
  
  $greece = Country::where('code', 'gr')->first();
  echo $greece->translate('en')->name; // abc
```

**Filling multiple translations**

```php
  $data = [
    'code' => 'gr',
    'en'  => ['name' => 'Greece'],
    'fr'  => ['name' => 'Grèce'],
  ];

  $greece = Country::create($data);
  
  echo $greece->translate('fr')->name; // Grèce
```

## Installation in 4 steps

### Step 1: Install package

Add the package in your composer.json by executing the command.

```bash
composer require dimsav/laravel-translatable
```

Next, add the service provider to `app/config/app.php`

```
'Dimsav\Translatable\TranslatableServiceProvider',
```

### Step 2: Migrations

In this example, we want to translate the model `Country`. We will need an extra table `country_translations`:

```php
Schema::create('countries', function(Blueprint $table)
{
    $table->increments('id');
    $table->string('code');
    $table->timestamps();
});

Schema::create('country_translations', function(Blueprint $table)
{
    $table->increments('id');
    $table->integer('country_id')->unsigned();
    $table->string('name');
    $table->string('locale')->index();

    $table->unique(['country_id','locale']);
    $table->foreign('country_id')->references('id')->on('countries')->onDelete('cascade');
});
```

### Step 3: Models

1. The translatable model `Country` should [use the trait](http://www.sitepoint.com/using-traits-in-php-5-4/) `Dimsav\Translatable\Translatable`. 
2. The convention for the translation model is `CountryTranslation`.


```php
// models/Country.php
class Country extends Eloquent {
    
    use \Dimsav\Translatable\Translatable;
    
    public $translatedAttributes = ['name'];
    protected $fillable = ['code', 'name'];

}

// models/CountryTranslation.php
class CountryTranslation extends Eloquent {

    public $timestamps = false;
    protected $fillable = ['name'];

}
```

The array `$translatedAttributes` contains the names of the fields being translated in the "Translation" model.

### Step 4: Configuration

Laravel 4.*
```bash
php artisan config:publish dimsav/laravel-translatable
```

Laravel 5.*
```bash
php artisan vendor:publish 
```

With this command, initialize the configuration and modify the created file, located under `app/config/packages/dimsav/laravel-translatable/translatable.php`.

*Note: There isn't any restriction for the format of the locales. Feel free to use whatever suits you better, like "eng" instead of "en", or "el" instead of "gr".  The important is to define your locales and stick to them.*

## Configuration

### The translation model

The convention used to define the class of the translation model is to append the keyword `Translation`.

So if your model is `\MyApp\Models\Country`, the default translation would be `\MyApp\Models\CountryTranslation`.

To use a custom class as translation model, define the translation class (including the namespace) as parameter. For example:

```php
<?php 

namespace MyApp\Models;

use Dimsav\Translatable\Translatable;
use Illuminate\Database\Eloquent\Model as Eloquent;

class Country extends Eloquent
{
    use Translatable;

    public $translationModel = 'MyApp\Models\CountryAwesomeTranslation';
}

```

## Documentation

**Please read the installation steps first, to understand what classes need to be created.**

### Available methods 

```php
// This is how we determine the current locale.
// It is set by laravel or other packages.
App::getLocale(); // 'fr' 

// To use this package, first we need an instance of our model
$germany = Country::where('code', 'de')->first();

// This returns an instance of CountryTranslation of using the current locale.
// So in this case, french. If no french translation is found, it returns null.
$translation = $germany->translate();

// If an german translation exists, it returns an instance of 
// CountryTranslation. Otherwise it returns null.
$translation = $germany->translate('de');

// If a german translation doesn't exist, it attempts to get a translation  
// of the fallback language (see fallback locale section below).
$translation = $germany->translate('de', true);

// Alias of the above.
$translation = $germany->translateOrDefault('de');

// Returns instance of CountryTranslation of using the current locale.
// If no translation is found, it returns a fallback translation
// if enabled in the configuration.
$translation = $germany->getTranslation();

// If an german translation exists, it returns an instance of 
// CountryTranslation. Otherwise it returns null.
// Same as $germany->translate('de');
$translation = $germany->getTranslation('de', true);

// Returns true/false if the model has translation about the current locale. 
$germany->hasTranslation();

// Returns true/false if the model has translation in french. 
$germany->hasTranslation('fr');

// If a german translation doesn't exist, it returns
// a new instance of CountryTranslation.
$translation = $germany->translateOrNew('de');

// Returns a new CountryTranslation instance for the selected
// language, and binds it to $germany
$translation = $germany->getNewTranslation('it');

// The eloquent model relationship. Do what you want with it ;) 
$germany->translations();
```

### Magic properties

To use the magic properties, you have to define the property `$translatedAttributes` in your
 main model:

 ```php
 class Country extends Eloquent {

     use \Dimsav\Translatable\Translatable;

     public $translatedAttributes = ['name'];
 }
 ```

```php
// Again we start by having a country instance
$germany = Country::where('code', 'de')->first();

// We can reference properties of the translation object directly from our main model.
// This uses the default locale and is the equivalent of $germany->translate()->name
$germany->name; // 'Germany'

// We can also quick access a translation with a custom locale
$germany->{'name:de'} // 'Deutschland'
```

### Fallback locales

If you want to fallback to a default translation when a translation has not been found, enable this in the configuration
using the `use_fallback` key. And to select the default locale, use the `fallback_locale` key.

Example:

```php
return [
    'use_fallback' => true,

    'fallback_locale' => 'en',    
];
```

You can also define *per-model* the default for "if fallback should be used", by setting the `$useTranslationFallback` property:

```php
class Country {

    public $useTranslationFallback = true;

}
```

## FAQ

#### I need some example code!

Examples for all the package features can be found [in the code](https://github.com/dimsav/laravel-translatable/tree/master/tests/models) used for the [tests](https://github.com/dimsav/laravel-translatable/tree/master/tests).

#### I need help!

Got any question or suggestion? Feel free to open an [Issue](https://github.com/dimsav/laravel-translatable/issues/new).

#### I want to help!

You are awesome! Watched the repo and reply to the issues. You will help offering a great experience to the users of the package. `#communityWorks`

#### I am getting collisions with other trait methods!

Translatable is fully compatible with all kinds of Eloquent extensions, including Ardent. If you need help to implement Translatable with these extensions, see this [example](https://gist.github.com/dimsav/9659552).

#### Why do I get a mysql error while running the migrations?

If you see the following mysql error:

```
[Illuminate\Database\QueryException]
SQLSTATE[HY000]: General error: 1005 Can't create table 'my_database.#sql-455_63'
  (errno: 150) (SQL: alter table `country_translations` 
  add constraint country_translations_country_id_foreign foreign key (`country_id`) 
  references `countries` (`id`) on delete cascade)
```

Then your tables have the MyISAM engine which doesn't allow foreign key constraints. MyISAM was the default engine for mysql versions older than 5.5. Since [version 5.5](http://dev.mysql.com/doc/refman/5.5/en/innodb-default-se.html), tables are created using the InnoDB storage engine by default.

##### How to fix

For tables already created in production, update your migrations to change the engine of the table before adding the foreign key constraint.

```php
public function up()
{
    DB::statement('ALTER TABLE countries ENGINE=InnoDB');
}

public function down()
{
    DB::statement('ALTER TABLE countries ENGINE=MyISAM');
}
```

For new tables, a quick solution is to set the storage engine in the migration:

```php
Schema::create('language_translations', function(Blueprint $table){
  $table->engine = 'InnoDB';
  $table->increments('id');
    // ...
});
```

The best solution though would be to update your mysql version. And **always make sure you have the same version both in development and production environment!**
