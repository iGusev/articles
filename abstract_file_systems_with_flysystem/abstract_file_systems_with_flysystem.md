>Abstract File Systems with Flysystem
http://www.sitepoint.com/abstract-file-systems-flysystem/

Reading and writing files is an integral aspect of any programming language, but the underlying implementation can vary enormously. For example, the finer details of writing data to the local filesystem compared to uploading over FTP is very different – yet conceptually, it’s very similar.

In addition to old warhorses like FTP, online storage is increasingly ubiquitous and inexpensive – with scores of services available such as Dropbox, Amazon’s S3 and Rackspace’s Cloud Files – but these all use subtly different methods for reading and writing.

That’s where [flysystem](https://github.com/thephpleague/flysystem) comes in. It provides a layer of abstraction over multiple file systems, which means you don’t need to worry where the files _are_, how they’re stored, or need be concerned with low-level I/O operations. All you need to worry about are the high-level operations such as reading, writing and organizing into directories.

Such abstraction also makes it simpler to switch from one system to another without having to rewrite vast swathes of application code. It also provides a logical approach to moving or copying data from one storage system to another, without worrying about the underlying implementation.

You can use Dropbox, S3, Cloud Files, FTP or SFTP as if they were local; saving a file becomes the same process whether it’s being saved locally or transferred over the network. You can treat zip archives as if they were a bunch of folders, without worrying about the nitty gritty of creating and compressing the archives themselves.

## Installation and Basic Usage

As ever, Composer is the best way to install:

```php
"league/flysystem": "0.2.*"
```

Now you can simply create one or more instances of `League\Flysystem\Filesystem`, by passing in the appropriate **adapter**.

For example, to use a local directory:

```php
use League\Flysystem\Filesystem;
use League\Flysystem\Adapter\Local as Adapter;

$filesystem = new Filesystem(new Adapter('/path/to/directory'));
```

To use an Amazon S3 bucket, there’s slightly more configuration involved:

```php
use Aws\S3\S3Client;
use League\Flysystem\Adapter\AwsS3 as Adapter;

$client = S3Client::factory(array(
    'key'    => '[your key]',
    'secret' => '[your secret]',
));

$filesystem = new Filesystem(new Adapter($client, 'bucket-name', 'optional-prefix'));
```

To use Dropbox:

```php
use Dropbox\Client;
use League\Flysystem\Adapter\Dropbox as Adapter;

$client = new Client($token, $appName);
$filesystem = new Filesystem(new Adapter($client, 'optional/path/prefix'));
```

(To get the token and application name, create an application using [Dropbox’s App Console](https://www.dropbox.com/developers/apps).)

Here’s an example for SFTP – you may not need every option listed here:

```php
use League\Flysystem\Adapter\Sftp as Adapter;

$filesystem = new Filesystem(new Adapter(array(
    'host' => 'example.com',
    'port' => 21,
'username' => 'username',
'password' => 'password',
'privateKey' => 'path/to/or/contents/of/privatekey',
'root' => '/path/to/root',
'timeout' => 10,
)));
```

For other adapters such as normal FTP, Predis or WebDAV, [refer to the documentation](https://github.com/thephpleague/flysystem).

## Reading and Writing to a File System

As far as your application code is concerned, you simply need to replace calls such as `file_exists()`, `fopen()` / `fclose()`, `fread` / `fwrite` and `mkdir()` with their flysystem equivalents.

For example, take the following legacy code, which copies a local file to an S3 bucket:

```php
$filename = "/usr/local/something.txt";
if (file_exists($filename)) {
    $handle = fopen($filename, "r");
    $contents = fread($handle, filesize($filename));
    fclose($handle);

    $aws = Aws::factory('/path/to/config.php');
    $s3 = $aws->get('s3');

    $s3->putObject(array(
        'Bucket' => 'my-bucket',
        'Key'    => 'data.txt',
        'Body'   => $content,
        'ACL'    => 'public-read',
    )); 
}
```

Using flysystem, it might look a little like this:

```php
$local = new Filesystem(new Adapter('/usr/local/'));
$remote = new Filesystem(
    S3Client::factory(array(
        'key'    => '[your key]',
        'secret' => '[your secret]',
    )),
    'my-bucket'
);

if ($local->has('something.txt')) {
    $contents = $local->read('something.txt');
    $remote->write('data.txt', $contents, ['visibility' : 'public']);
}
```

Notice how we’re using terminology such as _reading_ and _writing_, _local_ and _remote_ – high level abstractions, with no worrying about things like creating and destroying file handles.

Here’s a summary of the most important methods on the `League\Flysystem\Filesystem` class:

<table>

<thead>

<tr>

<th>Method</th>

<th>Example</th>

</tr>

</thead>

<tbody>

<tr>

<td>Reading</td>

<td>`$filesystem->read('filename.txt')`</td>

</tr>

<tr>

<td>Writing</td>

<td>`$filesystem->write('filename.txt', $contents)`</td>

</tr>

<tr>

<td>Updating</td>

<td>`$filesystem->update('filename.txt')`</td>

</tr>

<tr>

<td>Writing or updating</td>

<td>`$filesystem->put('filename.txt')`</td>

</tr>

<tr>

<td>Checking existence</td>

<td>`$filesystem->has('filename.txt')`</td>

</tr>

<tr>

<td>Deleting</td>

<td>`$filesystem->delete('filename.txt')`</td>

</tr>

<tr>

<td>Renaming</td>

<td>`$filesystem->rename('old.txt', 'new.txt')`</td>

</tr>

<tr>

<td>Reading Files</td>

<td>`$filesystem->read('filename.txt')`</td>

</tr>

<tr>

<td>Getting file info</td>

<td>`$filesystem->getMimetype('filename.txt')`</td>

</tr>

<tr>

<td>`$filesystem->getSize('filename.txt')`</td>

</tr>

<tr>

<td>`$filesystem->getTimestamp('filename.txt')`</td>

</tr>

<tr>

<td>Creating directories</td>

<td>`$filesystem->createDir('path/to/directory')`</td>

</tr>

<tr>

<td>Deleting directories</td>

<td>`$filesystem->deleteDir('path/to/directory')`</td>

</tr>

</tbody>

</table>

## Automatically Creating Directories

When you call `$filesystem->write()`, it ensures that the directory you’re trying to write to exists – if it doesn’t, it creates it for you recursively.

So this…

```php
$filesystem->write('/path/to/a/directory/somewhere/somefile.txt', $contents);
```

…is basically the equivalent of:

```php
$path = '/path/to/a/directory/somewhere/somefile.txt';
if (!file_exists(dirname($path))) {
    mkdir(dirname($path), 0755, true);
}
file_put_contents($path, $contents);
```

## Managing Visibility

Visibility – i.e., permissions – can vary in implementation or semantics across different storage mechanisms, but flysystem abstracts them as either “private” or “public”. So, you don’t need to worry about specifics of `chmod`, S3 ACLs, or whatever terminology a particular mechanism happens to use.

You can either set the visibility when calling `write()`:

```php
$filesystem->write('filename.txt', $data, ['visibility' : 'private']);
```

Prior to 5.4, or according to preference:

```php
$filesystem->write('filename.txt', $data, array('visibility' => 'private'));
```

Alternatively, you can set the visibility of an existing object using `setVisibility`:

```php
$filesystem->setVisibility('filename.txt', 'private');
```

Rather than set it on a file-by-file basis, you can set a default visibility for a given instance in its constructor:

```php
$filesystem = new League\Flysystem\Filesystem($adapter, $cache, [
'visibility' => AdapterInterface::VISIBILITY_PRIVATE
]);
```

You can also use `getVisibility` to determine a file’s permissions:

```php
if ($filesystem->getVisibility === 'private') {
    // file is secret
}
```

## Listing Files and Directories

You can get a listing of all files and directories in a given directory like this:

```php
$filesystem->listContents();
```

The output would look a little like this;

```php
[0] =>
  array(8) {
  'dirname' =>
    string(0) ""
    'basename' =>
    string(11) "filters.php"
    'extension' =>
    string(3) "php"
    'filename' =>
    string(7) "filters"
    'path' =>
    string(11) "filters.php"
    'type' =>
    string(4) "file"
    'timestamp' =>
    int(1393246029)
    'size' =>
    int(2089)
}
[1] =>
array(5) {
    'dirname' =>
    string(0) ""
    'basename' =>
    string(4) "lang"
    'filename' =>
    string(4) "lang"
    'path' =>
    string(4) "lang"
    'type' =>
    string(3) "dir"
}
```

If you wish to incorporate additional properties into the returned array, you can use `listWith()`:

```php
$flysystem->listWith(['mimetype', 'size', 'timestamp']);
```

To get recursive directory listings, the second parameter should be set to TRUE:

```php
$filesystem->listContents(null, true);
```

The `null` simply means we start at the root directory.

To get just the paths:

```php
$filesystem->listPaths();
```

## Mounting Filesystems

Mounting filesystems is a concept traditionally used in operating systems, but which can also be applied to your application. Essentially it’s like creating references – shortcuts, in a sense – to filesystems, using some sort of identifier.

Flysystem provides the Mount Manager for this. You can pass one or more adapters upon instantiation, using strings as keys:

```php
$ftp = new League\Flysystem\Filesystem($ftpAdapter);
$s3 = new League\Flysystem\Filesystem($s3Adapter);

$manager = new League\Flysystem\MountManager(array(
    'ftp' => $ftp,
    's3' => $s3,
));
```

You can also mount a file system at a later time:

```php
$local = new League\Flysystem\Filesystem($localAdapter);
$manager->mountFilesystem('local', $local);
```

Now you can use the identifiers as if they were protocols in URI’s:

```php
// Get the contents of a local file…
$contents = $manager->read('local://path/to/file.txt');

// …and upload to S3
$manager->write('s3://path/goes/here/filename.txt', $contents);
```

It’s perhaps more useful to use identifiers which are generic in nature, e.g.:

```php
$s3 = new League\Flysystem\Filesystem($s3Adapter);
$manager = new League\Flysystem\MountManager(array(
    'remote' => new League\Flysystem\Filesystem($s3Adapter),
));

// Save some data to remote storage
$manager->write('remote://path/to/filename', $data);
```

## Caching

Flysystem also supports a number of technologies for caching file metadata. By default it uses an in-memory cache, but you can also use Redis or Memcached.

Here’s an example of using Memcached; the cache adapter is passed to the `Filesystem` constructor as an optional second parameter:

```php
use League\Flysystem\Adapter\Local as Adapter;
use League\Flysystem\Cache\Memcached as Cache;

$memcached = new Memcached;
$memcached->addServer('localhost', 11211);
$filesystem = new Filesystem(new Adapter(__DIR__.'/path/to/root'), new Cache($memcached, 'storageKey', 300));
// Storage Key and expire time are optional
```

As with the filesystem adpaters, if you wished you could create your own – simply extend `League\Flysystem\Cache\AbstractCache`.

## Summary

In this article I’ve introduced flysystem, a package which provides a layer of abstraction over a variety of forms of file storage. Although I haven’t covered every available adapter, [the documentation](https://github.com/thephpleague/flysystem) should help fill in the gaps, introduce a couple of other features and document any new adapters in the future.