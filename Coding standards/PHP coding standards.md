In general we follow the [WordPress PHP Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/php/) with few modifications.

For automatic code check we are using [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer/), with our [modified coding standards](https://github.com/infinum/coding-standards-wp).

## Table of contents

- [Naming](#naming)
  * [File Naming](#file-naming)
  * [Naming Conventions](#naming-conventions)
  * [Namespacing and class names](#namespacing-and-class-names)
  * [Yoda Conditions](#yoda-conditions)
  * [Functions](#functions)
    + [Class method visibility](#class-method-visibility)
- [Writing style](#writing-style)
  * [Inline statements](#inline-statements)
  * [Multiple statements](#multiple-statements)
  * [Strict comparison](#strict-comparison)
- [Typehinting](#typehinting)
- [Sanitization and escaping](#sanitization-and-escaping)
- [Documentation](#documentation)
- [Database Queries](#database-queries)
- [Functional programming](#functional-programming)
  * [Mapping](#mapping)
  * [Reducing](#reducing)
  * [Filtering](#filtering)
  * [Anonymous functions](#anonymous-functions)
  * [Closures](#closures)
  * [Memoization](#memoization)
  * [Links on functional PHP programming](#links-on-functional-php-programming)
- [Optimizations](#optimizations)
  * [Array checking](#array-checking)
  * [Using array_push() and similar functions](#using-array-push---and-similar-functions)
  * [Caching](#caching)
  * [I18n](#i18n)
  * [A11y](#a11y)

## Naming

### File Naming

File names should be in lowercase letters with `-` as a separator between words, e.g. `theme-helpers.php`. In certain cases - creating 'page builder' using ACF, underscore (`_`) is allowed, but generally you should follow the dash as a separator.

Files containing a class should be named `class-{classname}.php`. There should always be only one class per file.

Use [wp-boilerplate](https://github.com/infinum/wp-boilerplate) as a basis for any template based web site. You should follow the existing structure of the theme there.

### Naming Conventions

Use lowercase letters in variable, action, and function names. Separate words via underscores. Don't abbreviate variable names unnecessarily - let the code be unambiguous and self-documenting.

### Namespacing and class names

Namespacing should follow file structure. The main namespace is Capital_Cased version of the project name. So in the case of the boilerplate, the default namespace is

```php
namespace Inf_Theme;
```

Every folder which holds classes constitutes a subnamespace.

Be aware that WordPress 'lives' in a global namespace. So, when using functions and classes from the WordPress core, it's necessary to either put a slash in front of class or function name, so that it's called from the global namespace, or use the `use` statement at the top of the file. For example, calling `WP_Query` inside your class should be done like

```php
$query = new \WP_Query( $arguments );
```

And core functions would be used like this

```php
\wp_unslash( $_POST['someKey'] );
```

Or

```php
use \WP_Query;

//...

$query = new WP_Query( $arguments );

```

This will be important especially if you are writing automated tests.

Avoid using static methods in your classes if possible. Using static methods means that you are calling a function without an instance of the class. But it prevents the usage of many OOP features such as inheritance, interface implementations, dependency injections etc. Static methods are useful for certain things like helper functions.

If you are using same method in many different classes, it's useful to put it in a [Trait](https://php.net/manual/en/language.oop5.traits.php).

A Trait is similar to a class, but only intended to group functionality in a fine-grained and consistent way. It is not possible to instantiate a Trait on its own. It is an addition to traditional inheritance and enables horizontal composition of behavior; that is, the application of class members without requiring inheritance.

Class names should use capitalized words separated by underscores.

`class Custom_Class { ... }`

Constants should be in all upper-case with underscores separating words:

`define( 'IMAGE_URL', get_template_directory_uri() . '/skin/public/images/' );`

or

`const THEME_VERSION = '1.0.0';`

### Yoda Conditions

We don't use [Yoda Conditions](https://en.wikipedia.org/wiki/Yoda_conditions), especially since code checker should take care of making sure that you don't assign a variable in the conditionals.

### Functions

When defining a function or a method there should be no space between a function name and an opening parenthesis, but there should be a space between closing parenthesis and opened curly bracket like so:

`function function_name( $var ) { ... }`

#### Class method visibility

Always add visibility keywords to methods and properties inside classes (`public`, `private` and `protected`).

* `public` scope is used to make that variable/function available from anywhere, other classes and instances of the object.

* `private` scope is used when you want your variable/function to be visible in its own class only.

* `protected` scope is used when you want to make your variable/function visible in all classes that extend current class including the parent class.

## Writing style

### Inline statements

Inline statements should have starting and ending php tags on the same line

```php
// Yes:
<?php echo esc_html( $x ); ?>

// No:
<?php
echo esc_html( $x );
?>
```

Writing conditionals, or control statements on a single line should be done with braces

```php
<?php if ( $some_variable === true ) { ?>
  <div class="block"></div>
<?php } ?>
```

Never write inline statements without braces. This is an extremely bad practice because it produces code that is hard to read and hard to maintain.

```php
<?php
// Bad code:
if ( $some_variable === true )
  $variable = 'This is inside true statement';

// Worse:
if ( $foo ) bar();

// Good code:
if ( $foo ) {
  bar();
}
```

### Multiple statements

Multiple statements should each be on its own line

```php
// Yes:
<?php
$x++;
echo esc_html( $x );
?>

// No:
<?php $x++; echo esc_html( $x ); ?>
```

### Strict comparison

Always use strict comparison inside conditionals, `in_array()`, `array_search()` or any php function that has the option to check the value and type of the checked variable.

It's important to use strict comparison because of [type juggling](https://php.net/manual/en/types.comparisons.php).

If you're in doubt about what kind of value is returned from the WordPress core function, use [WordPress code reference](https://developer.wordpress.org/reference/) to check it.

## Typehinting

Type declarations allow functions to require that parameters are of a certain type at call time. If the given value is of the incorrect type, then an error is generated: in PHP 5, this will be a recoverable fatal error, while PHP 7 will throw a `TypeError` exception. But we don't use or encourage using PHP < 7 in our projects.

To specify a type declaration, the type name should be added before the parameter name. The declaration can be made to accept `NULL` values if the default value of the parameter is set to `NULL`.

To enable strict typings in PHP, you need to set a `declare` directive at the top of your file, before `namespace` definition

```php
declare(strict_types=1);
```

You can typehint function arguments and return values, for example

```php
/**
   * Get user data
   *
   * A method that will return an array with user data.
   *
   * @param string $user_token Auth token got from url param.
   *
   * @return array             User data array.
   */
  public function get_user_data( string $user_token ) : array {
    //...
  }
```

When the method can return more than one types (string or boolean for instance), **don't specify the return type**, as this will most likely throw `TypeError` exceptions (the RFC for `mixed` typehint is opened for version 7.3, and you can read it [here](https://wiki.php.net/rfc/mixed-typehint)).

From PHP 7.1 you can explicitly declare a variable to be `null`

```php
public function get_array( ?string $some_string ) : array {
  //...
}
```

Typehinting is also important when working with dependency injections.

```php
/**
 * Class User credentials
 *
 * Class that stores user credentials.
 */
class User_Credentials {
  /**
   * General Helper object
   *
   * @var string
   *
   * @since 1.0.0
   */
  protected $general_helper;

  /**
   * Initialize class
   *
   * Load helper on class init.
   *
   * @param Helper $general_helper General helper class that implements Helper interface.
   * @since 1.0.0
   */
  public function __construct( Helper $general_helper ) {
    $this->general_helper = $general_helper;
  }

  //...
}
```

Table with typehints per PHP versions: https://mlocati.github.io/articles/php-type-hinting.html

## Sanitization and escaping

We follow WordPress [VIP's guidelines](https://vip.wordpress.com/documentation/vip/best-practices/security/validating-sanitizing-escaping/)

1. Never trust user input.
2. Escape as late as possible.
3. Escape everything from untrusted sources (like databases and users), third-parties (like Twitter), etc.
4. Never assume anything.
5. Never trust user input.
6. Sanitation is okay, but validation/rejection is better.
7. Never trust user input.

Every output has to be escaped. Even translatable strings. This means that instead of using `__()` and  `_e()` we need to use `esc_html__()`, `esc_html_e()`, `esc_attr__()`, `esc_attr_e()`, `wp_kses()`, `wp_kses_post()` and other escaping functions.

When writing data to the database be sure to [sanitize](https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/) the variables

`sanitize_text_field( wp_unslash( $_POST['my_data'] ) )`

And to [prepare](https://developer.wordpress.org/reference/classes/wpdb/prepare/) your database queries.

```php
$meta = 'Custom meta';

$post_id = 12;

$wpdb->query( $wpdb->prepare( "DELETE FROM $wpdb->postmeta WHERE post_id = %d AND meta_key = %s", $post_id, $meta ) );
```

## Documentation

Every file should have a beginning documentation that is describing the contents of the file. `functions.php` should have a description block about the theme/project

```php
<?php
/**
 * Theme Name: Project name
 * Description: A short project description
 * Author: Infinum
 * Author URI: https://infinum.co/
 * Version: 0.1.0
 *
 * @package Namespace
 */
```

We follow the [DocBlock](https://phpdoc.org/docs/latest/guides/docblocks.html) format of comments.

Every class should have the documentation before it, and the methods inside should also be documented.

```php
/**
 * Starts the list before the elements are added.
 *
 * @since 3.0.0
 *
 * @see Walker::start_lvl()
 *
 * @param string   $output Passed by reference. Used to append additional content.
 * @param int      $depth  Depth of menu item. Used for padding.
 * @param stdClass $args   An object of wp_nav_menu() arguments.
 */
public function start_lvl( &$output, $depth = 0, $args = array() ) {
  //...
}
```

## Database Queries

Querying the database should be done via `WP_Query` object. [Don't use query_posts()](https://wordpress.stackexchange.com/a/1755/58895) ever. It can affect other queries on the page because it reassigns the `global wp_query` object.

You can optimize your query by removing unnecessary queries

* `no_found_rows => true`: useful when pagination is not needed (AJAX load).
* `update_post_meta_cache => false`: useful when post meta will not be utilized.
* `update_post_term_cache => false`: useful when taxonomy terms will not be utilized.
* `fields => 'ids`': useful when only the post IDs are needed (less typical).

Never use `posts_per_page => -1`, as this will return every single post in the query, which can have detrimental effects if you have a large amount of posts. It's better to set a big number (500 or 1000) as the upper limit.

Direct database calls should be discouraged (`$wpdb`), as well as using `get_posts()`. But use them if you cannot avoid them.

Don't use `post__not_in`. If you need to skip posts, do it in php

```php
<?php
$custom_query = new WP_Query( array(
    'post_type' => 'post',
    'posts_per_page' => 30 + count( $posts_to_exclude )
) );

if ( $custom_query->have_posts() ) {
  while ( $custom_query->have_posts() ) {
    $custom_query->the_post();
    if ( in_array( get_the_ID(), $posts_to_exclude ) ) {
      continue;
    }
    the_title();
  }
}
?>
```

Try to avoid post meta queries if possible, that is, don't try to fetch posts by their post meta. Instead use [taxonomies](https://codex.wordpress.org/Taxonomies) to group posts, taxonomy queries are fast and won't effect your performance.

Fetching post meta if you know the post ID, or if you are in a post/page, on the other hand is fast and can be used anytime.

```php
// Don't do this:
$args = array(
    'meta_key'     => 'color',
    'meta_value'   => 'blue',
    'meta_compare' => '!='
);

$query = new WP_Query( $args );

// You can do this:
$color = get_post_meta( get_the_id(), 'color', true );
```

Avoid multi-dimensional queries - post queries based on terms across multiple taxonomies for instance.

It's better to do a query with the minimum number of dimensions possible, and then filter out the results with PHP.

## Functional programming

While we cannot write our code fully functionally, because PHP is not a functional programming language, we can use some of the functional techniques while writing our code. This can improve on the code readability and ease of maintenance.

Functions are first class citizens in PHP:

```php
// Function as a variable.
$function_name = function() {
  return 42;
};

// Function as a return type.
function create_function() {
  return function() {
    return 'Return from an anonymous function.';
  };
};

$new_func = create_function();

// Function as a parameter.
function display( $function ) {
  echo $func() , "\n\n";
}

// Call a function by name.
call_user_func_array( 'strtoupper', $some_string );

// objects as functions.
class Some_Class {
  public function __invoke( $param1, $param2 ) {
    [...]
  }
}

$instance = new Some_Class();
$instance( 'First', 'Second' ); // call the __invoke() method
```

Even though looping though iterable objects is generally faster if you use `for` or `foreach`, you can use some functional programming techniques which will make your code a bit less _boilerplatey_. Also the performance effects are [negligible](https://stackoverflow.com/questions/18144782/performance-of-foreach-array-map-with-lambda-and-array-map-with-static-function).

### Mapping

Applying a function to all elements:

```php
// Old way (imperative):
foreach ( $iterable as $iterable_key => $iterable_value ) {
  $iterable[ $iterable_key ] = strtoupper( $iterable_value );
}

// New way (functional):
$result = array_map( 'strtoupper', $iterable );
```

[array_map manual](https://php.net/manual/en/function.array-map.php)

### Reducing

Reduce an array to a single value:

```php
// Old way (imperative):
$result = 0;

foreach ( $int_array as $value ) {
  $result += $value;
}

// New way (functional):
$result = array_reduce( $int_array, function( $carry, $item ) { return $carry + $item; }, 0 );
```

[array_reduce manual](https://php.net/manual/en/function.array-reduce.php)

### Filtering

Iterating over an array but returning only those results that pass some conditions:

[array_filter manual](https://php.net/manual/en/function.array-filter.php)

```php
// Old way (imperative):
$result = [];

foreach ( $int_array as $value ) {
  if( $value % 2 === 0 ) {
    $result[] = $value;
  }
}

// New way (functional):
$result = array_filter( $int_array, function( $item ) { return ( $item % 2 === 0); } );
```

### Anonymous functions

Using anonymous functions for actions and filters could be problematic because that makes it very hard to unhook them later on

```php
add_action( 'init', function() {
  call_function();
}, 10 );
```

Instead do this

```php
add_action( 'init', 'my_callable_function', 10 );

function my_callable_function() {
  call_function();
}
```

If you are working in a 'public' project that can be extended the above should be followed, in client projects these are not that important, as you don't expect anybody to remove your added hooks (in themes or in plugins).

### Closures

In JavaScript, a closure can be thought of as a scope, when you define a function, it silently inherits the scope it's defined in, which is called its closure, and it retains that no matter where it's used. It's possible for multiple functions to share the same closure, and they can have access to multiple closures as long as they are within their accessible scope.

In PHP, a closure is a callable class, to which you've bound your parameters manually.

It's a slight distinction but one that bears mentioning.

A Closure is essentially the same as a Lambda apart from it can access variables outside the scope that it was created.

```php
// Set a multiplier.
 $multiplier = 3;

// Create a list of numbers.
 $numbers = array( 1, 2, 3, 4 );

// Use array_walk to iterate through the list and multiply.
array_walk( $numbers, function( $number ) use ( $multiplier ) {
  echo $number * $multiplier;
} );
```

[Closures in PHP](http://php.net/manual/en/class.closure.php)

### Memoization

Memoization is an optimization technique where we cache function results. If we have a pure function (one that has no side affects), we can cache the result the first time we run it and then just use cache. We can use static variables:

```php
function factorial( $n ) {
  static $cache = [];

  if ( $n === 1 ) {
    return 1;
  }

  if ( ! array_key_exists( $n, $cache ) ) {
    $cache[ $n ] = $n * factorial( $n - 1 );
  }

  return $cache[ $n ];
}
```

We can generalize this using helper function:

```php
function memoize( $func ) {
  return function() use ( $func ) {
    static $cache = [];

    $args = func_get_args();
    $key  = serialize( $args );

    if( ! array_key_exists( $key, $cache ) ) {
      $cache[ $key ] = call_user_func_array( $func, $args );
    }

    return $cache[ $key ];
  }
}
```

And then do:

```php
$factorial = function( $n ) use( &$factorial ) {
  if( $n === 1 ) {
    return 1
  };

  return $n * $factorial( $n -1 );
}

$mem_factorial = memoize( $factorial );
```

### Links on functional PHP programming

[PHP The Right Way](https://phptherightway.com/pages/Functional-Programming.html)  
[Functional Programming in PHP](https://www.liip.ch/en/blog/functional-programming-in-php)  
[Functional PHP](https://apiumhub.com/tech-blog-barcelona/functional-php/)  

## Optimizations

### Array checking

Avoid using `in_array()` check if possible, because it will traverse the entire array and check if the value of the array is present in the array. Instead look up by key and use `isset()` check.

```php
$array = array(
  'foo' => true,
  'bar' => true,
);

if ( isset( $array['bar'] ) ) {
  // value is present in the array.
};
```

If you have no control over the created array (there are no distinguishable keys to choose from), set the third parameter in the `in_array()` function to `true`. This will force strict comparisons (value and type).

```php
if ( in_array( 'some value', $array, true ) ) {
  // code goes here.
}
```

### Using array_push() and similar functions

When possible avoid using `array_push()`, and instead just append to the array directly

```php
$my_array = array();

// Good.
foreach ( $other_array as $new_key => $new_value ) {
  $my_array[] = $new_value;
}

// Avoid if possible.
foreach ( $other_array as $new_key => $new_value ) {
  array_push( $my_array, $new_value );
}
```

This will avoid any unnecessary overhead of calling the php function, as PHP has to look up the function reference, find its position in memory and execute whatever code it defines.

### Caching

Use [caching](https://10up.github.io/Engineering-Best-Practices/php/#the-object-cache) to speed up the site.

Use [`pre_get_posts`](https://developer.wordpress.org/reference/hooks/pre_get_posts/) hook to modify your queries in the back end and in the front end (search queries etc.).

Use [tranisents](https://codex.wordpress.org/Transients_API) to further speed up your site.

### I18n

All text strings in a project have to bi internationalized using core localization functions. You can check a great guide by Samuel Wood about [internalization in WordPress](https://ottopress.com/2012/internationalization-youre-probably-doing-it-wrong/).

### A11y

Accessibility is important. Always follow the official [WordPress a11y guidelines](https://make.wordpress.org/accessibility/handbook/).
