# Simple database wrapper for PDO

This is a *very* simplistic database wrapper for PDO, with two goals:
**Simplicity** and **Security**!

## First design goal: Simple!

I did not use the Singleton pattern for this class, because Singletons
always involve unnecessarily much code and aren't that nice to use and read.
A typical query execution of a Singleton-based DB-class looks like this:

```php
$db = DB::getInstance();
$db->query('SELECT ...');
$db->exec('INSERT INTO ...');
```

Or, if it's only one query:

```php
DB::getInstance()->query('SELECT ...');
```

Now, I think this `getInstance()->` part of the code neither carries
further information, nor is useful in some way. Therefore, I simply left
this part out, resulting in:

```php
DB::query('SELECT ...');
DB::exec('SELECT ...');
```

Much nicer, isn't it?

So, wonder which static methods you can use? All! All methods PDO implements.
All calls to static methods are simply redirected to the PDO instance.

## Second design goal: Secure!

Apart from this redirecting functionality this class offers two further methods:
`DB::q()` and `DB::x()`. These methods are shortcuts to `DB::query()` and `DB::exec()`
with the difference of "auto quoting":

```php
DB::q(
	'SELECT * FROM user WHERE group = ?s AND points > ?i',
	'user', 7000 //                   ^^              ^^
)
```

See those question marks? These are placeholders, which will be replaced with the arguments
passed after the query. There are several types of placeholders:

- `?s` (string)  inserts the argument applying string escaping through `PDO->quote`
- `?i` (integer) inserts the argument applying integer escaping through `intval`
- `?a` (array)   inserts the argument, converting it to a list of string-escaped values:
     DB::q('SELECT * FROM user WHERE name IN ?a', array('foo', 'bar', 'hello', 'world'));
     // results in:
     // SELECT * FROM user WHERE name IN ('foo','bar','hello','world')

## Configuration

There are two versions of this class available, one for PHP 5.3
(DB.php) and one for PHP 5.2 (DB_forPHP52.php). The only difference
is, that the former uses `__callStatic` to redirect the static calls
to the PDO instance, the latter simply redefines all methods. (You may
obviously use the 5.2 version on PHP 5.3, it actually should be slightly
faster.)

So, to get going and use this class, you have to modify the
`DB::instance` method, which by default is defined like this:

```php
public static function instance() {
	if (self::$instance === null) {
		self::$instance = new PDO(
			'mysql:host=' . DB_HOST . ';dbname=' . DB_NAME,
			DB_USER,
			DB_PASS
		);
		self::$instance->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
	}

	return self::$instance;
}
```

Replace the arguments of `new PDO()` to satisfy your needs.
Ten, `require_once` the file and have fun using it!

## Short reference

```php
class DB
{
	// returns the database instance
	public static function instance()

	// DB::query with autoQuote
	public static function q($query, $params, $...)

	// DB::exec with autoQuote
	public static function x($query, $params, $...)

	// autoQuote as described above
	public static function autoQuote($query, array $args)

    // All methods defined by PDO
    // e.d prepare(), quote(), ...
}
```
