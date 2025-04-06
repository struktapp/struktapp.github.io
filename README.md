Strukt
===

**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Strukt Installer](#strukt-installer)
  - [Installation](#installation)
  - [Usage](#usage)
- [Strukt](#strukt)
    - [Getting started](#getting-started)
    - [Generate Application](#generate-application)
    - [Generate Module](#generate-module)
    - [Execute Shell](#execute-shell)
  - [Cli](#cli)
    - [Cli Utility](#cli-utility)
    - [Run Application](#run-application)
  - [Notes](#notes)
- [Strukt Framework](#strukt-framework)
    - [Getting started](#getting-started-1)
  - [Setup, Cache, Configuration & Environment](#setup-cache-configuration--environment)
    - [Cache](#cache)
    - [Shell](#shell)
    - [Setting Application Type](#setting-application-type)
    - [Configuration](#configuration)
    - [Environment Setup](#environment-setup)
    - [Setup Packages Registry](#setup-packages-registry)
  - [Packages](#packages)
    - [Default Package](#default-package)
    - [Building Packages](#building-packages)
    - [Package Autoloading](#package-autoloading)
    - [Note](#note)
  - [Validator](#validator)
    - [Example](#example)
    - [Validator Annotations](#validator-annotations)
    - [Adding Validators](#adding-validators)
- [Strukt Router](#strukt-router)
  - [Getting Started](#getting-started)
    - [Quick Start](#quick-start)
  - [Advanced Router (The Nitty Gritty)](#advanced-router-the-nitty-gritty)
    - [Permissions](#permissions)
    - [Authentication](#authentication)
    - [Environment](#environment)
    - [Apache](#apache)
  - [BTW, DB Tip...](#btw-db-tip)
- [Strukt Commons](#strukt-commons)
  - [Usage](#usage-1)
    - [Collection](#collection)
  - [Value Objects](#value-objects)
    - [DateTime](#datetime)
    - [Today (Date Influence)](#today-date-influence)
    - [String](#string)
    - [Array](#array)
  - [Others](#others)
    - [Token Query](#token-query)
    - [Messages](#messages)
    - [Json](#json)
- [Strukt Console](#strukt-console)
  - [Block Spaces (IMPORTANT)](#block-spaces-important)
  - [Usage](#usage-2)
- [Strukt-Key](#strukt-key)
  - [Installation](#installation-1)
    - [Composer](#composer)
  - [Hashing](#hashing)
    - [Bcrypt](#bcrypt)
    - [Sha256 (Doubled)](#sha256-doubled)
  - [Public & Private Keys](#public--private-keys)
  - [Auto generate keys](#auto-generate-keys)
    - [Use existing key](#use-existing-key)
    - [Encrypt message with Public Key](#encrypt-message-with-public-key)
    - [Encrypt with password](#encrypt-with-password)
  - [Certificate Signing Request (CSR)](#certificate-signing-request-csr)
    - [Sign & verify certificate](#sign--verify-certificate)
  - [Note](#note-1)
- [Strukt Process](#strukt-process)
  - [Installation](#installation-2)
  - [Demo](#demo)
- [Strukt Event](#strukt-event)
  - [Events](#events)
  - [Reflector](#reflector)
  - [Loop](#loop)
- [Strukt Fs](#strukt-fs)
  - [Usage](#usage-3)
    - [API](#api)
    - [Helpers](#helpers)
- [Strukt Generator](#strukt-generator)
  - [Intro](#intro)
  - [Templator](#templator)
  - [Annotations](#annotations)
- [Strukt Math](#strukt-math)
  - [Number (Value Objects)](#number-value-objects)
  - [Matrix](#matrix)
  - [Monad](#monad)
- [Strukt Base](#strukt-base)
    - [Usage](#usage-4)
    - [Conclusion](#conclusion)
- [Tasker](#tasker)
  - [Getting started](#getting-started-2)
  - [Usage](#usage-5)
  - [Boxing](#boxing)
- [Strukt Queue](#strukt-queue)
    - [Requirements](#requirements)
  - [Usage](#usage-6)
    - [Wait](#wait)
    - [Reply](#reply)
  - [Other Usages](#other-usages)
    - [Pop](#pop)
    - [Push](#push)
  - [Credits](#credits)
- [Strukt Owl](#strukt-owl)
  - [Usage](#usage-7)
    - [Sentiment Analysis: Vader Score](#sentiment-analysis-vader-score)
    - [Tags: Ranking](#tags-ranking)
    - [Summarize](#summarize)
    - [Highlights](#highlights)
  - [Credits](#credits-1)
- [Strukt Contract](#strukt-contract)
- [Strukt Db](#strukt-db)
    - [Transaction](#transaction)
    - [Commands](#commands)
    - [SQL](#sql)
- [Strukt Asset](#strukt-asset)
    - [Installation](#installation-3)
    - [Simple Asset Manager](#simple-asset-manager)
    - [Image Resize](#image-resize)
- [Strukt Auth](#strukt-auth)
- [Strukt Tests](#strukt-tests)
    - [Installation](#installation-4)
  - [Getting Started](#getting-started-1)
    - [Installation](#installation-5)
  - [Package Development](#package-development)

Strukt Installer
===

## Installation

```php
composer global require "strukt/install"
```

## Usage

Create your application.

```sh
strukt new payroll
cd payroll
```

Install available packages and publish.

```sh
strukt add tests --publish
```

Available packages:

- db
- auth
- tests
- asset

Update Installer.

```sh
strukt update:me
```


Strukt Strukt
===


### Getting started

```sh
composer create-project strukt/strukt:1.1.8-alpha --prefer-dist
```

Listing console commands:

```sh
./xcli -l
```

### Generate Application

```sh
./xcli app:make payroll
```

The file structure generated should look as below:

```sh
app
└── src
    └── Payroll
        ├── AuthModule
        │   ├── Controller
        │   │   └── User.php
        │   ├── Form
        │   │   └── User.php
        │   ├── PayrollAuthModule.php
        │   ├── Router
        │   │   ├── Auth.php
        │   │   └── Index.php
        │   └── Tests
        │       └── UserTest.php
        └── User.php # Models are stored in the root of your app (i.e payroll)

```

There is a default module i.e `AuthModule` when you generate an application. Folders generated in a module (facets) can be changed in `cfg/module.ini` this also indicates part of alias used to access classes/objects. You'll also find a config file `cfg/app.ini` that holds the active application name.

When an application or module is created/generated it is loaded by running the command below, otherwise strukt won't detect it:

```sh
./xcli app:reload
```

The above command will create a `App/Loader.php` in the `lib/` folder at the root of your project. This file should NEVER be edited because everything will be overwritten once the above command is rerun. 

### Generate Module

Command syntax for generating a module:

```sh
./xcli make:module <app_name> <module_name> <module_alias>
```

Example command:

```sh
./xcli make:module payroll human_resource hr
```

Now the file structure should look as below:

```sh
app/
└── src
    └── Payroll
        ├── AuthModule
        └── HumanResourceModule
```

Remember to run the `app:reload` command to load the module.

### Execute Shell

`strukt-strukt` uses [Psysh](https://github.com/bobthecow/psysh).

To drop into shell:

```sh
$ ./xcli shell:exec
 ls
    Variables: $core
 $core->get("au.ctr.User")->getAll()
    "AuthModule\Controller\User::getAll Not Yet Implemented!"
 $core->get("User")
    Payroll\User {#...
```

## Cli

View `providers` and `middlewares`

```sh
./xcli sys:ls middlewares # view for console
```

View `index.php` middlewares

```sh
./xcli sys:ls middlewares --idx # view for index.php
```

You can also view `providers` by replacing `middlewares`

### Cli Utility

Enable and disable `commands` , `middlewares` and `providers`

Example:

```sh
./xcli sys:util enable commands pub-make
```

### Run Application

```sh
./xcli app:exec
```

Uses `.env` `server_{var}` variables to run application.

## Notes

The `make:router` and `make:module` commands will not work on cli console until you run `app:make` and `app:reload` commands are run respectively.

**IMPORTANT**: The folder `.tpl/` in the root of the project contains `sgf/` folder that contains class template files used to generate the application modules and migrations. Ensure to **NOT** change it until you've understood [strukt-generator](https://github.com/pitsolu/strukt-generator)

Have a good one!

Strukt Framework
================


The is the package that unifies all [strukt-strukt](https://github.com/samweru/strukt-strukt)
components under the framework.

Rarely should anyone use this on its own.

### Getting started

```sh
echo {"minimum-stability":"dev"} > composer.json
composer require "strukt/framework:1.1.8-alpha" --prefer-dist
```

## Setup, Cache, Configuration & Environment

### Cache

Always remember to clear and reload the cache when necessary

```sh
./xcli cache:clear 
./xcli cache:make
```
### Shell

Drop into shell

```sh
./xcli shell:exec
```

### Setting Application Type

```php
config("app.type", "App:Idx")// for index.php, alternative App:Cli for console
```

### Configuration

```php
config("facet.middlewares")
config("facet.providers")
```

### Environment Setup

This class is defaultly found in [strukt-commons](https://github.com/samweru/strukt-commons)

```php
Strukt\Env::withFile();//default .env file in your root folder
Strukt\Env::withFile(".env-dev");
env("root_dir", getcwd());//setter custom environment variable
env("root_dir");//getter
```

### Setup Packages Registry 

Packages reference file location `./cfg/repo.ini`

```php
repos(); //list all repositories
repos("published");//list all published strukt packages
repos("installed");//list all installed strukt packages
```

## Packages

### Default Package

```php
package("core", "App:Idx")->get("settings"); //returns array of middlewares, commands and providers
//below mode:App:Cli is default
package("core")->get("name");//core
package("core")->get("cmd:name");//null
package("core")->get("files");//null
package("core")->get("modules");//null
package("core")->get("is:published");//true by default
package("core")->get("requirements");//null or array
```

The above methods are interfaced in class `Strukt\Framework\Contract\Package` you must use them in your package.

### Building Packages

The first step in developing your package will require you to install `strukt-framework`
and execute `composer exec strukt` command that will create your folder structure. You'll need to create `src` and `package` folders. 

See structure of package below:

```sh
├── bootstrap.php
├── cfg/
├── console
├── index.php
├── lib/
├── tpl/
├── vendor/
├── composer.json
├── LICENSE
├── package/ #Place all your packages files here
├── README.md
└── src
    └── Strukt
        └── Package
            └── Pkg{{Package Name}}.php #Identify your package resources here
```

Again, your package class in `src/Strukt/Package/Pkg<Package Name>.php` will have methods
listed in `Strukt\Fraamework\Contract\Package`

### Package Autoloading

You may require to autoload libraries both from your root directory and package resources.

```php
$loader = require "vendor/autoload.php";
...
...
$loader->addPsr4("App\\", [

	__DIR__."/lib/App",
	__DIR__."/package/lib/App"
]);

return $loader;
```

### Note

For packages that require installation into your `app/src/{{AppName}}` folder, there
are a few tricks you could use while building your package. The `publish:package` command
takes argument `<package>` for publishing packages that are currently in development,
since your source will be in the root folder in a subfolder called `package`. 

This will require you to enter into your `cfg/repo.php` and indicate you are currently in-development with the key/keyword `package` which will allow the publisher to install files in the your app source folder `app/src`.

The `publish:package` command installs from `vendor` but in development-mode you can use `--dev` switch
to install your package that will be located in your project root.

## Validator

### Example

```php
namespace Payroll\AuthModule\Form;

use Strukt\Framework\Contract\Form as AbstractForm;

class User extends AbstractForm{

	/**
	* @IsNotEmpty()
	* @IsAlpha()
	*/
	public string $username;

	/**
	* @IsNotEmpty()
	*/
	public string $password;
}
```

### Validator Annotations

```php
/**
* @IsNotEmpty()
* @IsAlpha()
* @IsAlphaNum()
* @IsNumeric()
* @IsEmail()
* @IsDate(Y-m-d)
* @IsIn(a,b,c)
* @EqualTo(xyz)
* @IsLen(10)
*/
```

### Adding Validators

New validators can be added is in your `lib/App/Validator.php`
There you can find an example `App\Validator::isLenGt`

```php
/**
* @IsLenGt(10)
*/
```

Strukt Router
=============


## Getting Started

### Quick Start 

Create `composer.json` script with contents below then run `composer update`

```js
{
    "require":{

        "strukt/router":"v1.1.5-alpha"
    },
    "minimum-stability":"dev"
}
```

Your `index.php` file.

```php
require "vendor/autoload.php";

use Strukt\Http\Request;
// use Strukt\Http\Response\Plain as Response;

$app = new Strukt\Router\QuickStart();

$app->get("/", function(Request $request){

    // return new Response("Hello World!");
    return "Hello World!";
});

exit($app->run());
```

## Advanced Router (The Nitty Gritty)

### Permissions

```php
$app->inject("permissions", function(){

    return array(

        // "show_secrets"
    );
});

$app->providers(array(

    //App\Provider\ExampleProvider::class
));

$app->middlewares(array(

    Strukt\Router\Middleware\Session::class,
    Strukt\Router\Middleware\Authentication::class,
    Strukt\Router\Middleware\Authorization::class,
));

$app->get("/user/secrets", function(){

    return "Shh!";

},"show_secrets");

exit($app->run());
```

### Authentication

```php
$app->inject("permissions", function(){

    return [];
});

$app->inject("session", function(){

    return new Strukt\Http\Session\Native;
});

$app->inject("verify", function(Strukt\Http\Session\Native $session){

    $user = new Strukt\User();
    $user->setUsername($session->get("username"));

    return $user;
});

$app->providers(array(

    //App\Provider\ExampleProvider::class
));

$app->middlewares(array(

    Strukt\Router\Middleware\Session::class,
    Strukt\Router\Middleware\Authentication::class,
    Strukt\Router\Middleware\Authorization::class,
));

$app->post("/login", function(Strukt\Http\Request $request){

    $username = $request->get("username");
    $password = $request->get("password");

    $request->getSession()->set("username", $username);

    return new Strukt\Http\Response\Plain(sprintf("User %s logged in.", $username));
});

$app->get("/current/user", function(Strukt\Http\Request $request){

    return $request->getSession()->get("username");
});

$app->get("/logout", function(Strukt\Http\Request $request){

    $request->getSession()->invalidate();

    return new Strukt\Http\Response\Plain("User logged out.");
});

exit($app->run());
```

### Environment

After installation run  `composer exec static` to get `public\` directory.

```
    public/
    ├── errors
    │   ├── 403.html
    │   ├── 404.html
    │   ├── 405.html
    │   └── 500.html
    └── static
        ├── css
        │   └── style.css
        ├── index.html
        └── js
            └── script.js
```

### Apache

`.htaccess` file:

```
DirectoryIndex index.php

RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule . index.php [L]
```

## BTW, DB Tip...

[Adminer](adminer.org) is a really neat tool! It is a single file dba and can be placed 
under a router easily! Download the adminer.php file and place in root folder.

```php
$app->any("/dba", function(Request $request){

    include "./adminer-x.x.x.php";

    return new Strukt\Http\Response\Plain();
});
```
Cheers!

Strukt Commons
==============


## Usage

### Collection

```php
$contact = collect([]);
$contact->set("mobile", "+2540770123456");
$contact->set("work-phone", "+2540202345678");

$user = collect([]);
$user->set("contacts", $contact);
$user->get("contacts.mobile"); //outputs +2540770123456

$s = array(
    "user"=>array(
        "firstname"=>"Gene",
		"surname"=>"Wilder",	
		"db"=>array(
            "config"=>array(
                "username"=>"root",
				"password"=>"_root!"
            )
        ),
        "mobile_numbers"=>array( //Non Assoc Array
            "777111222",
            "770234567"
        )
    )
);

$b = collect($s);
$x = $b->get("user.db.config.username"); //returns root
```

## Value Objects

You may also find `Number` object via package `strukt/math` that is a dependency of this package.

### DateTime

```php
$start = when()
$end = when("+30 days");
$rand = $start->rand($end);//make start date random date between start and end
$end->gt($start);//true
$end->gte($start);//true
$start->lt($end);
$end->lte($end);//true
$start->equals($end);//false
$start->same(new DateTime);//true -- is the same day
$newStart = $start->clone();
$newStartPlusOneDay = $start->clone("+1 day");
$start->reset();//reset time to 00:00:00 000000
$start->last();//reset time to 23:59:59 1000000
echo $start; //return date as string
```

### Today (Date Influence)

```php
// $p = period(when("1900-01-01"), when("1963-12-31")); 
$p = period()

//In order for date influence to work the first 2 line below must be 
// called before any further date manipulation
// messing arround with dates can create headaches
$p->create(when("1900-01-01"), when("1963-12-31")); //period
$p->reset(when("1960-03-23"));//create fake today
$fakeToday = today();

//All dates created with Strukt\Type\DateTime will be in 1960
$fakeToday->same(new DateTime); //false
$fakeToday->same(when());//true
$fakeToday->hasPeriod()//true -- has period
$fakeToday->withDate(when("1959-04-01"))->isValid(); //true -- is date valid with period

// $fakeToday->getState("period.start");
// $fakeToday->getState("period.end");
$fakeToday->getState();//get state of date manipulation
$fakeToday->reset();//reset back to original today
```

### String

```php
$str = str("Strukt Framework");
$str->empty();//false
$str->startsWith("Str");//true
$str->endsWith("work");//true
$str->first(3);//Str
$str->last(4);//work
$str->contains("Frame");//true
$str->slice(7,5)->equals("Frame");//true
$str->replace("work", "play")->equals("Strukt Frameplay");
$str->replaceFirst("k","c")->equals("Struct Framework");
$str->replaceLast("k","d")->equals("Struct Frameword");
$str->replaceAt("ing", 3, 3)->equals("String Framwork")
$str->toUpper();//STRUKT FRAMEWORK
$str->toLower();//strukt framework
$camel = str("thisIsCamelCase");
$camel->toSnake();//this_is_camel_case
$camel->toSnake()->toCamel();//ThisIsCamelCase
$sdo = $str->prepend("Doctrine + ");//Doctrine + Strukt Framework
$sdo->concat(" = Strukt Do");//Doctrine + Strukt Framework = Strukt Do
$str->split(" ");//['Strukt', "Framework"]
str("blah blah blah")->count("blah");//3
```

### Array

```php
$rr = array(
    "firstname"=>"Bruce",
    "lastname"=>"Wayne",
    "alias"=>"Joker",
    "contacts" =>array(
        "email"=>"brucewayne@wayneent.com",
        "address"=>array(

            "street"=>"Boulavard of Broken Dreams",
            "building"=>"Wayne Co."
        )
    )
);

$arr = arr($rr);//Arr::create($rr)
$arr->has("Banner")//false
$arr->empty();//false
$arr->length();//3
$arr->count();//3
$arr->next();//true
$arr->current()->yield();//Wayne
$arr->key();//lastname
$arr->last()->equals($rr["contacts"]);
$arr->reset();
$arr = $arr->each(function($key, $val){ //loop

    if($key == "alias")
        $val = "Batman";

    return $val;
});
$arr = $arr->recur(function($key, $val){ //recursive iterate 

    if($key == "building")
        $val = "Wayne Co. & Associates";

    return $val;
});
$origarr = $arr->yield();
$rawarr = $arr->map(array( //reformat array

    "email_contact"=>"contacts.email",
    "address_street"=>"contacts.address.street",
    "address_building"=>"contacts.address.building"
));
$arr->pop();// remove at end of array.
$arr->push("brucebatman", "username");//add at end of queue. key is optional
$arr->enqueue("active", "status");//same as Arr.push. key is optional
$arr->enqueueBatch(["empty","empty"]);
$arr->prequeue("admin", "type");//add at beginning of queue. key is optional
$arr->dequeue();//remove at beginning of array. returns Bruce
$flatarr = $arr->level();//flattens multidimentional array
$is_assoc = arr(["username"=>"pitsolu", "password"="redacted"])->isMap();//is fully associative arr
$arr = arr(array(
    array(
        "username"=>"pitsolu",
        "type"=>"admin"
    ),
    array(
        "username"=>"peterparker",
        "type"=>"user"
    )
));
$arr->column("type");//returns array("admin", "user")
$arr = arr(array(

    "user"=>"pitsolu",
    "type"=>"admin",
    "status"=>"active"
));
$arr->tokenize();//returns user:pitsolu|type:admin|status:active
$arr->concat(",")//pitsolu,admin,active
arr(["a","b","c"])->isOfStr();// true
arr([1,2,3])->isOfNum();// true 
arr(["a","a","b","b","b","c","c","d"])->distict()->yield();//["a" => 2,"b" => 3,"c" => 2,"d" => 1]
arr(["a","b","b","c","c","c","d"])->uniq()->yield();//["a","b","c","d"]
arr(["a","b","c","d"])->slice(2)//["c","d"]
arr(["a","b","c","d"])->slice(1,3)//["b","c","d"]

$x = ["name"=>"peter","email"=>"peter@gmail.com"];
arr($x)->only(["email"])->yield()//["email"=>"peter@gmail.com"]

$x = [["name"=>"peter","email"=>"peter@gmail.com"], ["name"=>"john","email"=>"john@gmail.com"]]
arr($x)->order()->asc("email")->yield()//sorting 2d array by column
arr($x)->nested();//is nested array - true
arr([1,2,3])->product();//6
arr(["ab","cd","ef"])->has("ab");//true
arr(["a"=>1,"b"=>2,"c"=>3])->contains("a")//true
arr(["a"=>1,"b"=>2,"c"=>3])->values()//[1,2,3]
arr(["a","b"])->merge(["c","d"])->yield();//["a","b","c","d"]
arr(["a","b","c","d"])->reverse()->yield();//["d","c","b","a"]
```
## Others

### Token Query

```php
/**
 * Basic Token 
 */
$token = "user:pitsolu|status:active|is_superuser:true";

$query = token($token);

$query->get("user");//pitsolu
$query->get("status");//active
$query->get("is_superuser");//true

$query->has("role");//false
$query->keys();//["user","status","is_superuser"]

$query->token();//original token -- user:pitsolu|status:active|is_superuser:true
$query->set("role","admin");
$query->yield();//user:pitsolu|status:active|is_superuser:true|role:admin

/**
 * Complex Token
 */
$token = "contact:1|is:tenant,landlord,prospect";

$query = token($token);

$query->get("is");//["tenant","landlord","prospect"];
$query->set("status", ["active","published"]);
$query->yield();//contact:1|is:tenant,landlord,prospect|status:active,published
```

### Messages

```php
msg("error 401!");
msg("error 402!");
msg("error 404!");

$errors = msg()->get();

$errors->last()->yield(); //error 404!
$errors->reset();
$errors->current()->yield(); //error 401!
$errors->next();
$errors->current()->yield(); //error 402!
```

### Json

```php
$l = json(array("fname"=>"Peter", "lname"=>"Pan"));//json string
$l->encode();//json string
$m = $l->pp();//pretty print
$n = $l->decode();//array
json("{}")->valid();// is valid json. will return true
```

Strukt Console
==============


This is a console framework that utilises `DocBlock` to parse command description and format.

## Block Spaces (IMPORTANT)

This package uses `DocBlock` to generate your commands. The `DocBlock` must be in single spaces
and *NOT* tabs. Be conservative with the spaces and don't leave any that are unnecessary
otherwise the commands will not work.

## Usage

Sample Command:

```php
namespace Command;

use Strukt\Console\Input;
use Strukt\Console\Output;
use \Strukt\Console\Command as AbstractCommand;

/**
* mysql:auth          MySQL Authentication
* 
* Usage:
*   
*      mysql:auth <database> --username <username> --password <password> [--host <127.0.0.1>]
*
* Arguments:
*
*      database  MySQL database name - optional argument
* 
* Options:
* 
*      --username -u   MySQL Username
*      --password -p   MySQL Password
*      --host -h       MySQL Host - optional default 127.0.0.1
*/
class MySQLAuth extends AbstractCommand{ 

	public function execute(Input $in, Output $out){

		$out->add(sprintf("%s:%s:%s", 
							$in->get("database"), 
							$in->get("username"), 
							$in->get("password")));
	}
}
```

Add this in your executable file:

```php
#!/usr/bin/php
<?php
$app = new Strukt\Console\Application("Strukt Console");
$app->add(new Command\MySQLAuth);
$app->run($_SERVER["argv"]);
```

Call command:

```sh
php console mysql:auth payroll -u root -p p@55w0rd
```

Prompt for input and masked input, you may but need not describe promted input 
in command docblock:

```php
...
//prompt for input
$username = $in->getInput("Username:");
$nickname = $in->getInput("Nickname:");
//masked input
$password = $in->getMaskedInput("Password:");
$cpassword = $in->getMaskedInput("Confirm Password:");
...
```


Strukt-Key
=====

## Installation

### Composer

Create `composer.json` script with contents below then run `composer update`

```js
{
    "require":{

        "strukt/key":"v1.1.0-alpha"
    },
    "minimum-stability":"dev"
}
```

## Hashing

### Bcrypt

```php
$hash = bcry("p@55w0rd")->encode();
$success = bcry("p@55w0rd")->verify($hash);
```

### Sha256 (Doubled)

```php
$hash = sha256dbl('p@55w0rd');
```

## Public & Private Keys

## Auto generate keys

```php
// $k = Strukt\Ssl\All::makeKeys();
$ks = ssl()->getKeys();
$ks->getPrivateKey()->getPem();//get Private Key
$ks->getPublicKey()->getPem();//get Public Key
$c = $ks->useCipher();
$enc = $c->encrypt("p@55w0rd");
$dec = $c->decrypt($enc);
```

### Use existing key

You can generate your key via `ssh-keygen` if you wantta.

```php
$file = "file:///home/churchill/.ssh/id_rsa"
// $k = Strukt\Ssl\All::keyPath($file)
$k = ssl($file);
```

### Encrypt message with Public Key

```php
$message = "Hi! My name is (what?)
My name is (who?)
My name is
Slim Shady
Hi! My name is (huh?)
My name is (what?)
My name is
Slim Shady";

$file = "file:///home/churchill/.ssh/id_rsa.pub"

// $p = new Strukt\Ssl\KeyPair();//No Private Key
// $p->setPublicKey($file);
$p = keypair()->setPublicKey($file); //No Private Key

// $enc = Strukt\Ssl\All::useKeys($p)->toSend($message);
$enc = ssl($p)->toSend($message);
```

### Encrypt with password

```php
// $p = new Strukt\Ssl\KeyPair($path, "p@55w0rd");
// $p->getPublicKey(); // Trigger public key extraction from private key
$p = keypair($path, "p@55w0rd")->getPublicKey();

// $k = Strukt\Ssl\All::useKeys($p)
$k = ssl($p);
```

## Certificate Signing Request (CSR)

### Sign & verify certificate

```php
$kpath = "file:///home/churchill/.ssh/id_rsa"
$cpath = "file:///home/churchill/.ssh/cacert.pem"

// $oCsr = Strukt\Ssl\All::keyPath($kpath)->withCert($cpath);
$oCsr = ssl($kpath)->withCert($cpath);
$cert = $oCsr->sign();
$success = $oCsr->verify($cert);
```

## Note

For local keys in your project or project root. Example:

```sh
├── strukt     # Private Key
└── strukt.pub # Public Key
```
You can use helper `local` to localize your path to url:

```php
$k = ssl(local("strukt"));
// $p = keypair()->setPublicKey(local("strukt.pub"));
```



# Strukt Process


PHP Process Execution

## Installation

```sh
composer require strukt/process v1.0.1
```

## Demo

```php
#!/usr/bin/env php
<?php

use Strukt\Process;

require 'vendor/autoload.php';

// $password = "p@55w0rd";

// $p = Process::run(["ls", "ls -al"]);
// $p = Process::run(["read password ; echo \$password"], function(){
$ps = Process::run(["ping 127.0.0.1"], function($streamOutput){

	echo $streamOutput;
	//wait 5 seconds before continuing
	// sleep(5);
});

// $p = $ps->current();

// $p->write($password);
// $p->closeInput();

// $error = $p->error();
// $output = $p->read();
```


Strukt Event
===

This is not an `event-loop` it has events and loops.

## Events

```php
$cred = array("uname"=>"admin", "pword"=>"p@55w0rd");
$login = Strukt\Event::create(fn($uname, $pword)=>$uname == $cred["uname"] && $pword == $cred["pword"]);
$isLoggedIn = $login->apply("admin","p@55w0rd")->exec();
// $isLoggedIn = $login->applyArgs($credentials)->exec();
```

## Reflector

```php
// $r = Strukt\Ref::createFrom(new Payroll\User);
$r = ref(Payroll\User::class);
$r->getRef();//ReflectionClass
//$r->noMake();//newInstanceWithoutConstructor
$r->make("pitsolu");//newInstanceArgs
$r->getInstance();//InstanceOf Payroll\AuthModule\Model\User
$r->prop("id")->getRef();//ReflectionProperty
$r->prop("id")->set(1);
$r->prop("id")->get();//1
$r->method("getUsername")->invoke();//pitsolu
$r->method("getUsername")->getRef(); //ReflectionMethod
$r->method("getUsername")->getClosure();//Closure
ref("array_sum")->invoke([1,2]);//3
ref("array_sum")->getRef();//ReflectionFunction
```

## Loop

```php
use Strukt\Loop;
# use Strukt\Cmd;

Loop::add("auth", ["admin", "p@55w0rd"], function($username, $password){

	echo sprintf("username:%s|password:%s\n", $username, $password);
});

Loop::add("help", function(){

	echo "Docs\n";
});

// Loop::add("hello", function($name){

// 	echo sprintf("Hello %s", $name);
// });

Loop::run();
// Cmd::exec("help");
// Cmd::exec("auth", ["peter", "pazzw0rd"]);
// Cmd::exec("hello", ["World"]);
```

Strukt Fs
=========


Basic filesystem functionality. 

## Usage

### API

```php
Strukt\Fs::isDir(file) //Directory Exists
Strukt\Fs::isFile(file) //File Exists
Strukt\Fs::isPath(path) //Path Esists
Strukt\Fs::cat(file) //Dump File Contents
Strukt\Fs::touch(file) //Create File
Strukt\Fs::touchWrite(file, contents) //Create Write To File
Strukt\Fs::rename(from, to) //Rename File
Strukt\Fs::overwrite(file, contents) //Overwrite File Contents
Strukt\Fs::appendWrite(file, contents) //Append Contents To File
Strukt\Fs::rm(file) //Delete File
Strukt\Fs::rmdir(dir) //Recursively Delete
Strukt\Fs::mkdir(dir) //Recursively Create
Strukt\Fs::isWritable(file) //Check If File Is Writeable
Strukt\Fs::isReadable(file) //Check If File is Readable
Strukt\Fs::copyRecur(source,destination) //Copy recursively
Strukt\Fs::cpr(source,destination) //Alias for copyRecur
Strukt\Fs::listFilesRecur(path) //List directory files recursively
Strukt\Fs::lsr(path) //Alias for listFilesRecur
Strukt\Fs::tail(filepath, lines = 20) //read last line of file: default 20 lines
Strukt\Fs::isWindows()//Is OS Windows
Strukt\Fs::dirSep(path)//OS appropriate directory separator on path
Strukt\Fs::ds(path)//Alisas for dirSep
```

### Helpers

```php
fs(); #Strukt\Fs
fs("."); #Strukt\Local\Fs
tail("README.md");
ds("src/Strukt/Contract"); # Change directory separator depending on OS
path_exists("src/Strukt/Contract");
phar()->active(); # Detect is code is inside a Phar archive
phar("src/Strukt/Contract")->adapt(); # Adapt to Phar path if in Phar archive, otherwise return qualified path
```



Strukt Generator
===


## Intro

Simple package for generating templates and reading annotations.

## Templator

```php
$data = array(

    "title" => "The Title",
    "subtitle" => "Subtitle",
    "footer" => "Foot",
    "people" => array(
        
        array("name" => "Steve","surname" => "Johnson"),
        array("name" => "James", "surname" => "Johnson"),
        array("name" => "Josh", "surname" => "Smith")
    ),
    "page" => "Home"
);

$tpl = "<html>
<title>{{title}}</title>
<body>
<h1>{{subtitle}}</h1>
{{begin:people}}
<b>{{name}}</b> {{surname}}<br />
{{end:people}}
<br /><br />
<i>{{footer}}</i>
</body>
</html>";

// $output = Strukt\Templator::create($tpl, $data);
$output = template($tpl, $data);
```

## Annotations

Annotation supported format:

```php
/**
* @Route(/)
*/
class DefaultController{

    /**
    * @Route(/hello/{to:alpha})
    * @Method(POST, GET)
    * @Provides(application/html) 
    */
    function hello($to){ ...

    /**
    * @Route(/login)
    * @Method(GET)
    * @Secure(username=test, password=test)
    * @Expects(username,password)
    *
    * note the below will not be parsed
    * @param str $username
    * @param str $password
    */
    function login($username, $password){ ...
```

Run parser:

```php
// $parser = new \Strukt\Annotation\Parser\Basic(new \ReflectionClass(Controller\DefaultController::class));
// print_r($parser->getNotes());
print_r(notes(Controller\DefaultController::class))
```


Strukt Math
===

## Number (Value Objects)

```php
$num = number(1000);
$num = $num->add(200);//1200
$num = $num->subtract(100);//1100
$num = $num->times(2);//2200 multiplication
$num = $num->parts(4);//550 division
$rem = $num->mod(9);//1 modulus
$num = $num->raise(2);//302500 power
list($num1, $num2) = $num->ratio(1,1);//151250,151250
list($num1, $num2) = $num->ratio(1,3);//75625,226875
list($num1, $num2, $num3) = $num->ratio(1,1,3);//60500,60500,181500
$num->gt(302499);//true; greaterthan
$num->gte(302500);//true greaterthanorequals
$num->lt(302499);//false lessthan
$num->lte(302501);//true lessthanorequals
$num->negate()->equals(-302500)  
$num->yield();//return native number
$num->reset();//0
Number::create(1000000)->format();//1,000,000.00
Number::create(20.5111111)->round(2);//20.51
Number::random(4, 10, 20); //return 4 random numbers between 10 and 20
Number::create(10.1)->type();//double
echo $num;//return native number
```

## Matrix

```php
// $a = array(array(1,2,3),array(4,5,6),array(7,8,9));
$a = array(

    array(1,2,3),
    array(4,5,6),
    array(7,8,9)
);

// $b = array(array(11,22,33),array(44,55,66),array(77,88,99));
$b = array(

    array(11,22,33),
    array(44,55,66),
    array(77,88,99)
);


$c = (string)matrix($a)->multiply($b);
/** Result
[330,396,462]
[726,891,1056]
[1122,1386,165]
**/

$c = matrix($a)->multiply($b)->yield();
/** Result
array(
    array(330,396,462),
    array(726,891,1056),
    array(1122,1386,1650)
)
*/
```

## Monad

```php
// Linear Equation y = mx + c

$params = array("c"=>12, "m"=>3, "x"=>2)

// $y = monos($params)
//         ->next(fn($m, $x)=>$m * $x)
//         ->next(fn($mx, $c)=>$mx + $c)
//         ->next(fn($r)=>$r);

$y = monos($params)->next(fn($m, $x)=>$m * $x)->next(fn($mx, $c)=>$mx + $c)->next(fn($r)=>$r);

echo $y->yield();
```

Strukt Base
===

Base functionality for helpers

### Usage

```sh
psysh vendor/autoload.php
```

Inside `Psysh`

```php
>>> helper()
>>> helper("base")
[
	"helper_add"
]
```

### Conclusion

Install `strukt\install` and try shell functionality for better understanding.



Tasker
===

A simple task manager for php.

## Getting started

```sh
wget https://github.com/samweru/strukt-tasker/releases/download/v1.0.1-alpha/tasker.phar #download
chmod a+x tasker.phar #make executable
mv tasker.phar tasker #rename
```

## Usage

By default, task manager will create `tasker.php` file if one isn't found when you execute the tasker command.

The intial `tasker.php` file contains a single command `test`. 

How to list commands:

```sh
$ tasker list

 version         Tasker version
 list            List commands
 test            Sample task
```
Below is sample `tasker.php`

```php

/**
 * Show today's date
 */
task('date', function(){

	$date = new \DateTime();

	echo(sprintf("Now: %s\n", $date->format("Y-m-d H:i:s")));
});

/**
 * Say hello to someone
 */
task("hello", function(string $name){

    writeln(sprintf("Hello %s!", $name));
});

/**
 * Say hello to the world
 */
task('test', function(){

    go("hello", " World!");
});

/**
 * Watch changes in javascript files
 */ 
task("watch:js", function(){

	watch("app/js", function($files){

		$changes = [];
		foreach($files as $file)
			$changes[] = sprintf("%s\n", $file);

		print_r(implode("\n", $changes));
	});
});

/**
 * List directories
 */
task("lsdir", function(){

	list($output, $error) = run("ls -al", function($output){

		echo $output;
	});
});
```

## Boxing

First you'll need to install [phive](https://github.com/phar-io/phive)

```sh
wget -O phive.phar https://phar.io/releases/phive.phar
wget -O phive.phar.asc https://phar.io/releases/phive.phar.asc
gpg --keyserver hkps://keys.openpgp.org --recv-keys 0x9D8A98B29B2D5D79
gpg --verify phive.phar.asc phive.phar
chmod +x phive.phar
sudo mv phive.phar /usr/local/bin/phive
```

Then, install [Box](https://github.com/box-project/box) globally.

```sh
phive install humbug/box --force-accept-unsigned
```

..and update.

```sh
phive update humbug/box --force-accept-unsigned
```

..or install `Box` locally.

```sh
composer require --dev bamarni/composer-bin-plugin
composer bin box require --dev humbug/box

vendor/bin/box
```

..or 

```sh
$ curl -LSs https://box-project.github.io/box2/installer.php | php
```

Strukt Queue
===

Simple implementation of a Message Queue 
that uses Memcached https://memcached.org/
as backend
 
Needs PHP Memcached extensions installed to work properly

### Requirements

```sh
sudo apt install memcached
sudo apt install php-memcached
sudo apt install libmemcached-tools # cli tools
```

## Usage

### Wait

```php
//wait.php
$data = ["username"=>"pitsolu", "password"=>"p@55w0rd"];
wait("queue_test", $data, function($reply){

	print_r($reply);
});
```
### Reply

```php
//reply.php
reply("queue_test", function($data){

	$data["processed"] = true;

	return $data;
});
```

## Other Usages

### Pop

```php
$data = array(
	"username"=>"pitsolu", 
	"password"=>"p@55w0rd"
);

$uid = push("queue_test", $data);
```

### Push

```php
list($oid, $udata) = pop("queue_test");

print_r(array(

	"uid"=>$uid,
	"oid"=>$oid,
	"data"=>$data,
	"udata"=>$udata
));
```

## Credits

~~~
@author Maurizio Giunti https://www.mauriziogiunti.it / https://codeguru.it
@license MIT
~~~

Strukt Owl
===

AI package. Mostly text analysis. More to come...

<!-- ![Won](owl.jpg "Owl") -->

<center><img src="owl.jpg" width="200" height="200"></center>

## Usage

### Sentiment Analysis: Vader Score

```php
$text = "From Oasis' first ever single to the name of Noel and Liam's long-suffering mother, just how well do YOU know the band? Take MailOnline's ultimate quiz";

probe($text);
```

### Tags: Ranking

```php

// use tags function
tags($text); //get all tags

// get to 10 tags above 0.9 ranking
arr(tags($text))->each(fn($k,$v)=>number($v)->gt(0.9)?$k:null)->filter()->values()->yield()
```

### Summarize

```php
$text = fs()->cat("news.txt");

summary($text);
```

### Highlights

```php
$text = fs()->cat("news.txt");

highlights($text);
```

## Credits

~~~
- TextRank https://github.com/davmixcool/php-sentiment-analyzer
- PHP Sentiment Analyzer https://github.com/DavidBelicza/PHP-Science-TextRank
~~~


Strukt Contract
===

Abstract Classes and Interfaces for Strukt Framework

```sh
src/
└── Strukt
    └── Contract
        ├── Http
        │   ├── Error
        │   │   └── HttpErrorInterface.php
        │   ├── RequestInterface.php
        │   ├── ResponseInterface.php
        │   └── SessionInterface.php
        ├── MiddlewareInterface.php
        ├── ProviderInterface.php
        └── UserInterface.php
```


Strukt Db
===

### Transaction

```php

require "bootstrap.php";

useDb("pop");

pdo()->transact(function(){

	$role = db("role");
	$role->name = "superadmin";
	$role->save();

	$user = db("user");
	$user->username = "sadmin@tenure";
	$user->password = sha1("p@55w0rd!!");
	$user->role_id = "abc"; //invalid entry expect a number
	$user->save();
});
```

### Commands

```sh
Database
 model:make      Make model
 db:make-models  Make models from db
 db:make         Make db from models
 db:seeds        Seed database tables iwth JSON set (folder)
 db:wipe         Truncate database
 db:sql          Truncate database
 ```

### SQL

```sql
├── modify(string $table)
│   ├── addSet(string $modify)
│   ├── set(string $modify)
│   │   ├── andWhere(string $condition)
│   │   ├── orWhere(string $condition)
│   │   ├── where(string $condition)
│   │   └── yield():string
│   └── yield():string
└── select(string $fields)
    ├── addSelect(string $fields)
    ├── from(string $tables)
    │   └── leftjoin(string $join)
    ├── groupBy(string $columns)
    ├── limit(int $limit)
    ├── orderBy(string $columns, string $order = "DESC")
    ├── page(int $page, int $perPage=10)
    ├── union(string $sql)
    ├── unionAll(string $sql)
    └── where(string $condition)
        ├── andWhere(string $condition)
        └── orWhere(string $condition)
```

Strukt Asset
===

### Installation

```sh
composer require strukt/pkg-asset:v1.0.6-alpha
```

If need be you need to install the middlewares, providers and commands:

```sh
./console publish:package pkg-asset
```

### Simple Asset Manager

```php
$finder = new \Strukt\Asset($root_dir, $static_dir);
$finder->exists("/js/script.js");
$finder->getInfo("/js/script.js");//SplFileInfo
$finder->get("/js/script.js");//returns contents of file
```

### Image Resize

```php
$image = new \Gumlet\ImageResize();
$image = new \Gumlet\ImageResize('image.jpg');
$image->scale(50);
$image->save('image2.jpg')

$image = new \Gumlet\ImageResize('image.jpg');
$image->resizeToHeight(500);
$image->save('image2.jpg')
```

For more on `Gumlet` see [Gumlet/ImageResize](https://github.com/gumlet/php-image-resize) on Github.

Strukt Auth
===

```sh
package/
├── pop-db
│   └── app
└── red-db
    └── app
```

Strukt Tests
===

### Installation

```sh
composer require strukt/pkg-tests:v1.0.1
```

## Getting Started

Project `strukt/pkg-tests` is a `strukt` module.

### Installation

Install and publish `strukt/pkg-tests`:

```sh
console generate:app nameofyourapp
composer require strukt/pkg-tests
console publish:package pkg-tests
```

There is a list of console commands for preparing your authentication.

```sh
$ ./console -l

Strukt Console
==============
...
...
PhpUnit
 test:ls           List Tests 
 test:run          Execute Tests
```
## Package Development

This package will require installation into your `app/src/{{AppName}}` folder.
The `publish:package` command takes argument `package` if you inicate key `package` in 
your `cfg/repo.php` file for value `Strukt\Package\PkgTests::class`. Since your source 
will be in the root folder in a subfolder called `package` it will allow the publisher 
to install files in the your app source folder `app/src`.

```sh
./console publish:package package
```
Have a good one!

