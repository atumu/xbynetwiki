title: phplearnt1 

#  PHP学习之文件系统操作库league/flysystem 
官网:http://flysystem.thephpleague.com
github:https://github.com/thephpleague/flysystem
https://packagist.org/packages/league/flysystem

league/flysystem通过统一的抽象API，提供了各种文件系统的Adapter。统一进行操作。不管是本地的还是远程的。同时它还非常便于集成。目前有很多框架都集成了它。如Laravel
##  安装 
```

composer require league/flysystem

```
建议安装的包:(主要是各种文件系统的驱动包)
  * **ext-fileinfo**: Required for **MimeType**
  * league/flysystem-eventable-filesystem: Allows you to use EventableFilesystem
  * league/flysystem-rackspace: Allows you to use Rackspace Cloud Files
  * league/flysystem-copy: Allows you to use Copy.com storage
  * league/flysystem-azure: Allows you to use Windows Azure Blob storage
  * league/flysystem-webdav: Allows you to use WebDAV storage
  * league/flysystem-aws-s3-v2: Allows you to use S3 storage with AWS SDK v2
  * league/flysystem-aws-s3-v3: Allows you to use S3 storage with AWS SDK v3
  * league/flysystem-dropbox: Allows you to use Dropbox storage
  * league/flysystem-cached-adapter: Flysystem adapter decorator for metadata caching
  * **league/flysystem-sftp**: Allows you to use SFTP server storage via phpseclib
  * **league/flysystem-ziparchive**: Allows you to use ZipArchive adapter


适配器:Adapters
  * **Local**
  * Amazon Web Services - S3 V2: https://github.com/thephpleague/flysystem-aws-s3-v2
  * Amazon Web Services - S3 V3: https://github.com/thephpleague/flysystem-aws-s3-v3
  * Rackspace Cloud Files: https://github.com/thephpleague/flysystem-rackspace
  * Dropbox: https://github.com/thephpleague/flysystem-dropbox
  * OneDrive: https://github.com/jacekbarecki/flysystem-onedrive
  * **Ftp**
  * **Sftp** (through phpseclib): https://github.com/thephpleague/flysystem-sftp
  * **Zip** (through ZipArchive): https://github.com/thephpleague/flysystem-ziparchive
  * **WebDAV** (through SabreDAV): https://github.com/thephpleague/flysystem-webdav
  * PHPCR: https://github.com/thephpleague/flysystem-phpcr
  * Azure Blob Storage
  * NullAdapter
  * **Redis** (through Predis): https://github.com/danhunsaker/flysystem-redis
  * Fallback: https://github.com/Litipk/flysystem-fallback-adapter
  * **Memory**: https://github.com/thephpleague/flysystem-memory
  * Google Cloud Storage: https://github.com/Superbalist/flysystem-google-storage
  * SinaAppEngine Storage: https://github.com/litp/flysystem-sae-storage
  * Gaufrette: https://github.com/jenkoian/flysystem-gaufrette
缓存支持Caching
  * Memory (array caching)
  * Redis (through Predis)
  * Memcached
  * Adapter
  * Stash
##  文件系统挂载 
League\Flysystem\MountManager
```

$ftp = new League\Flysystem\Filesystem($ftpAdapter);
$s3 = new League\Flysystem\Filesystem($s3Adapter);
$local = new League\Flysystem\Filesystem($localAdapter);

// Add them in the constructor
$manager = new League\Flysystem\MountManager([
    'ftp' => $ftp,
    's3' => $s3,
]);

// Or mount them later
$manager->mountFilesystem('local', $local);


// Read from FTP
$contents = $manager->read('ftp://some/file.txt');

// And write to local
$manager->write('local://put/it/here.txt', $contents);

```
##  快速使用 
http://flysystem.thephpleague.com/api/
```

Write Files
$filesystem->write('path/to/file.txt', 'contents');
Update Files
$filesystem->update('path/to/file.txt', 'new contents');
Write or Update Files
$filesystem->put('path/to/file.txt', 'contents');
Read Files
$contents = $filesystem->read('path/to/file.txt');
Check if a file exists
$exists = $filesystem->has('path/to/file.txt');

Delete Files
$filesystem->delete('path/to/file.txt');
Read and Delete
$contents = $filesystem->readAndDelete('path/to/file.txt');
Rename Files
$filesystem->rename('filename.txt', 'newname.txt');
Copy Files
$filesystem->copy('filename.txt', 'duplicate.txt');
Get Mimetypes
$mimetype = $filesystem->getMimetype('path/to/file.txt');
Get Timestamps
$timestamp = $filesystem->getTimestamp('path/to/file.txt');
Get File Sizes
$size = $filesystem->getSize('path/to/file.txt');

Create Directories
$filesystem->createDir('path/to/nested/directory');
Directories are also made implicitly when writing to a deeper path
$filesystem->write('path/to/file.txt', 'contents');
Delete Directories
$filesystem->deleteDir('path/to/directory');

```

元数据获取
```

//$contents = $filesystem->listContents('some/dir', true);
$contents = $filemanager->listContents();
foreach ($contents as $object) {
    echo $object['basename'].' is located at'.$object['path'].' and is a '.$object['type'];
}

$paths = $filemanager->listPaths();

foreach ($paths as $path) {
    echo $path;
}


$listing = $filesystem->listWith(['mimetype', 'size', 'timestamp'], 'optional/path/to/dir', true);

foreach ($listing as $object) {
    echo $object['path'].' has mimetype: '.$object['mimetype'];
}

$info = $filesystem->getWithMetadata('path/to/file.txt', ['timestamp', 'mimetype']);
echo $info['mimetype'];
echo $info['timestamp'];

```
##  Using streams for reads and writes 
```

$stream = fopen('/path/to/database.backup', 'r+');
$filesystem->writeStream('backups/'.strftime('%G-%m-%d').'.backup', $stream);

// Using write you can also directly set the visibility
$filesystem->writeStream('backups/'.strftime('%G-%m-%d').'.backup', $stream, [
    'visibility' => AdapterInterface::VISIBILITY_PRIVATE
]);

if (is_resource($stream)) {
    fclose($stream);
}

// Or update a file with stream contents
$filesystem->updateStream('backups/'.strftime('%G-%m-%d').'.backup', $stream);

// Retrieve a read-stream
$stream = $filesystem->readStream('something/is/here.ext');
$contents = stream_get_contents($stream);
fclose($stream);

// Create or overwrite using a stream.
$putStream = tmpfile();
fwrite($putStream, $contents);
rewind($putStream);
$filesystem->putStream('somewhere/here.txt', $putStream);

if (is_resource($putStream)) {
    fclose($putStream);
}

```
##  缓存适配 
composer require league/flysystem-cached-adapter
###  Memory Caching 
```

use League\Flysystem\Filesystem;
use League\Flysystem\Adapter\Local as Adapter;
use League\Flysystem\Cached\CachedAdapter;
use League\Flysystem\Cached\Storage\Memory as CacheStore;

// Create the adapter
$localAdapter = new Local('/path/to/root');

// Create the cache store
$cacheStore = new CacheStore();

// Decorate the adapter
$adapter = new CachedAdapter($localAdapter, $cacheStore);

// And use that to create the file system
$filesystem = new Filesystem($adapter);

```

###  Predis Caching 
```

use League\Flysystem\Filesystem;
use League\Flysystem\Adapter\Local as Adapter;
use League\Flysystem\Cached\CachedAdapter;
use League\Flysystem\Cached\Storage\Predis as Cache;

$adapter = new CachedAdapter(new Adapter(__DIR__.'/path/to/root'), new Cache);
$filesystem = new Filesystem($adapter);

// Or supply a client
$client = new Predis\Client;
$adapter = new CachedAdapter(new Adapter(__DIR__.'/path/to/root'), new Cache($client));
$filesystem = new Filesystem($adapter);

```

###  Memcached Caching 
```

use League\Flysystem\Filesystem;
use League\Flysystem\Adapter\Local as Adapter;
use League\Flysystem\Cached\CachedAdapter;
use League\Flysystem\Cached\Storage\Memcached as Cache;

$memcached = new Memcached;
$memcached->addServer('localhost', 11211);

$adapter = new CachedAdapter(
    new Adapter(__DIR__.'/path/to/root'),
    new Cache($memcached, 'storageKey', 300)
);
$filesystem = new Filesystem($adapter);
// Storage Key and expire time are optional

```

##  处理复制与文件上传 
###  Plain PHP Upload 
```

$stream = fopen($_FILES[$uploadname]['tmp_name'], 'r+');
$filesystem->writeStream('uploads/'.$_FILES[$uploadname]['name'], $stream);
fclose($stream);

```
###  Symfony Upload 
```

/** @var Symfony\Component\HttpFoundation\Request $request */
/** @var Symfony\Component\HttpFoundation\File\UploadedFile $file */
$file = $request->files->get($uploadname);

if ($file->isValid()) {
    $stream = fopen($file->getRealPath(), 'r+');
    $filesystem->writeStream('uploads/'.$file->getClientOriginalName(), $stream);
    fclose($stream);
}

```

###  Laravel 5 - DI 
```

<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UploadController extends Controller {
    public function store(Request $request, FilesystemInterface $filesystem)
    {
        $file = $request->file('upload');
        $stream = fopen($file->getRealPath(), 'r+');
        $filesystem->writeStream('uploads/'.$file->getClientOriginalName(), $stream);
        fclose($stream);
    }
}

```
##  自定义插件 
```

use League\Flysystem\FilesystemInterface;
use League\Flysystem\PluginInterface;

class MaximusAwesomeness implements PluginInterface
{
    protected $filesystem;

    public function setFilesystem(FilesystemInterface $filesystem)
    {
        $this->filesystem = $filesystem;
    }

    public function getMethod()
    {
        return 'getDown';
    }

    public function handle($path = null)
    {
        $contents = $this->filesystem->read($path);

        return sha1($contents);
    }
}

```
```

use League\Flysystem\Filesystem;
use League\Flysystem\Adapter;

$filesystem = new Filesystem(new Adapter\Local(__DIR__.'/path/to/files/'));
$filesystem->addPlugin(new MaximusAwesomeness);
$sha1 = $filesystem->getDown('path/to/file');

```
##  Creating an adapter 
http://flysystem.thephpleague.com/creating-an-adapter/
需要实现League\Flysystem\AdapterInterface接口
![](/data/dokuwiki/php/pasted/20160419-112202.png)


##  各种Adapter的使用 
请参考：http://flysystem.thephpleague.com
