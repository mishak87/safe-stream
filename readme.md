Nette SafeStream: Atomic Operations
===================================

[![Downloads this Month](https://img.shields.io/packagist/dm/nette/safe-stream.svg)](https://packagist.org/packages/nette/safe-stream)
[![Build Status](https://travis-ci.org/nette/safe-stream.svg?branch=v2.2)](https://travis-ci.org/nette/safe-stream)

The Nette\Utils\SafeStram protocol for file manipulation guarantees atomicity and isolation of every file operation. Why
is it actually good? Let's start with a simple example, where we repeatedly write the same string to the file and then read it:

```php
$s = str_repeat('Long String', 10000);

$counter = 1000;
while ($counter--) {
	file_put_contents('soubor', $s);       // write it
	$readed = file_get_contents('soubor'); // read it
	if ($s !== $readed) {                  // check it
		echo 'Strings are different!';
	}
}
```

It may seem that the `echo 'Strings are different!'` command can't ever get executed. The opposite is true. Try to run this script in
two browsers simultaneously. The error occurs almost immediately.

It's because the code is not safe when performed repeatedly at the same time (ie, in multiple threads). And that is nothing unusual
on the Internet, where several people often connect to one website at the same time. Therefore, it's very important to ensure
that your application can handle multiple threads at once - that it's *thread-safe*, because native PHP functions are not.
Otherwise, you can expect data loss and strange errors occuring.

How to ensure, that functions like file_get_contets or `fwrite` behave atomically? The SafeStream protocol offers a secure solution,
so we can atomically manipulate files through standard PHP functions. The protocol is available automatically when you
load Nette Framework. After that, you just need prefix the filename with `safe://`:

```php
$handle = fopen('safe://test.txt', 'x'); // prefix the filename with safe://

fwrite($handle, 'Nette Framework'); // for now, the data is written into a temporary file

fclose($handle); // and only now the file is renamed to test.txt
```

You can of course use all the familiar function, such as:

```php
file_put_contents('safe://test.txt', $content);

$ini = parse_ini_file('safe://autoload.ini');
```

SafeStream guarantees:

- **Atomicity**: The file is written either as a whole or not written at all.
- **Isolation**: No one can start to read a file that is not yet fully written.

If you write to an existing file in the '`a`' mode (append), SafeStream creates it's copy and only after successfully writing it
renames it to the original name. Write in this mode is therefore more resource-consuming than in other modes.
