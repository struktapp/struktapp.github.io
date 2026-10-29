Strukt Fs
===

Fast local filesystem, rooted path, and ZIP archive helpers. The package
provides a static `Fs` API, a rooted `Rfs` facade, and procedural helpers for
common operations.

## Install

```sh
composer require strukt/fs
```

Requires PHP 8.2+ and `strukt/base`; ZIP operations require `ext-zip`.

## Quick start

```php
require __DIR__.'/vendor/autoload.php';

$files = fs(__DIR__);
$files->touchWrite('storage/app.log', "ready\n");
echo $files->cat('storage/app.log');

$entries = $files->lsr('storage');
$recent = $files->tail('storage/app.log', 20);
```

## Helpers

| Helper | Purpose |
| --- | --- |
| `fs()` | Return an `Fs` instance, or a rooted `Rfs` when a directory is supplied. |
| `tail()` | Read the final lines of a file. |
| `ds()` | Normalize slashes to the current OS separator. |
| `path_exists()` | Check for an existing file or directory. |
| `phar()` | Detect and adapt paths inside a Phar archive. |
| `local()` | Convert an existing path to a `file://` URI. |

```php
$absolute = fs();
$project = fs(__DIR__);

$project->touchWrite('var/cache/data.txt', 'cached');

tail('README.md');
ds('src/Strukt/Contract');
path_exists('src/Strukt/Contract');

phar()->active();
phar('src/Strukt/Contract')->adapt();

local('README.md'); // file:///...
```

## Static Fs API

`Strukt\Fs\Fs` exposes the complete local filesystem API through static
methods.

### Existence and file access

```php
use Strukt\Fs\Fs;

Fs::isDir($dir);
Fs::isFile($file);
Fs::isPath($path);

Fs::cat($file);
Fs::touch($file);
Fs::touchWrite($file, $contents);
Fs::rename($from, $to);
Fs::overwrite($file, $contents);
Fs::appendWrite($file, $contents);
```

`touch()` creates only a new file. `touchWrite()` creates missing parent
directories, creates a new file, and writes all contents; it returns `false` if
the file already exists. `overwrite()` and `appendWrite()` require an existing
file and return the number of bytes written. `rename()` backs up an existing
destination before replacing it.

### Removal, permissions, and listing

```php
Fs::rm($file);
Fs::rmdir($dir);
Fs::mkdir($dir, 0755, true);
Fs::isWritable($file);
Fs::isReadable($file);

Fs::copyRecur($source, $dest);
Fs::cpr($source, $dest);

Fs::listFiles($path);
Fs::ls($path);
Fs::listFilesRecur($path);
Fs::lsr($path);
```

`rmdir()` removes a complete directory tree, including hidden entries.
`mkdir()` is idempotent. Recursive listings include hidden files and return paths
for every regular file below the directory.

### Path and platform helpers

```php
Fs::isWindows();
Fs::dirSep($path);
Fs::ds($path);
```

### Tailing and archives

```php
Fs::tail($filepath, 20);

Fs::zip($path, $zipfile = null);
Fs::addZipList($zipfile, $files);
Fs::unzip($zipfile, $topath = './');
Fs::lsz($zippath);
```

`zip()` archives every file below a directory; when no zip file is supplied it
creates a timestamped archive in the current working directory. `unzip()`
rejects absolute and parent-traversal entries before extracting into
`topath/archive-name`.

### PHP file helpers

```php
Fs::ini($path, sections: true);
Fs::req($path);
```

`ini()` parses an INI file with PHP's typed scanner. `req()` includes a PHP file
and returns its included value.

## Rooted Rfs API

`Strukt\Fs\Rfs` fixes a root once and interprets every relative path below it.

```php
use Strukt\Fs\Rfs;

$root = new Rfs(__DIR__);
$root->path('storage/app.log'); // absolute path below the root

$root->touchWrite('storage/app.log', "ready\n");
$root->cat('storage/app.log');
$root->lsr('storage');
$root->zip('storage');
```

`Rfs` exposes the same `isDir()`, `isFile()`, `isPath()`, `cat()`, `touch()`,
`touchWrite()`, `rename()`, `overwrite()`, `appendWrite()`, `rm()`, `rmdir()`,
`mkdir()`, `isWritable()`, `isReadable()`, `copyRecur()`, `cpr()`,
`listFiles()`, `ls()`, `listFilesRecur()`, `lsr()`, `tail()`, `zip()`,
`addZipList()`, `unzip()`, `lsz()`, `ini()`, and `req()` methods.

Rfs is a path convenience facade, not a security sandbox. Validate untrusted path
input in the application.

## Phar helper

```php
use Strukt\Phar;

Phar::adapt('src/Strukt/Contract');
Phar::isPhar();
```

`adapt()` resolves a path against the active archive when available, otherwise
against the current working directory. `isPhar()` reports whether the current
code is running from a Phar archive.

## Notes

- Recursive operations use iterators and include dot-prefixed entries.
- `tail()` reads only the required suffix in chunks.
- ZIP extraction rejects absolute and parent-traversal entry names.
- Archive methods fail cleanly when an archive cannot be opened or written.
