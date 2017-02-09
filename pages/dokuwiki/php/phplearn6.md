title: phplearn6 

#  PHP学习之文件上传与下载 
##  文件上传 
php.ini配置：
假设要上传一个90M的大文件。配置 php.ini 如下：
```

file_uploads = On
upload_tmp_dir = "d:/fileuploadtmp"
upload_max_filesize = 90M
post_max_size = 100M
max_file_uploads=10
max_execution_time = 3600 ;秒，一个小时
memory_limit=1048576000);Byte 1000 兆，即 1G 

```
提示：**除此之外max_execution_time,memory_limit也有影响**，需要保持 memory_limit > post_max_size > upload_max_filesize
max_execution_time也可通过` set_time_limit() ` — 设置脚本最大执行时间
当此函数被调用时，**set_time_limit()会从零开始重新启动超时计数器。**换句话说，如果超时默认是30秒，在脚本运行了了25秒时调用 set_time_limit(20)，那么，脚本在超时之前可运行总时间为45秒。
` 当php运行于安全模式时，此功能不能生效。除了关闭安全模式或改变php.ini中的时间限制，没有别的办法。 `


使用 PHP 可以很方便的实现文件上传，其具体流程如下：
表单选择文件 -> 检查文件大小及类型 -> 生成临时文件 -> 移动临时文件至文件存储目录 -> 记录文件信息以便于管理。
在文件上传功能中，需要考虑以下几个问题：
  * 限定上传文件的大小
  * 限定上传文件的类型
  * 只允许可信任的用户上传文件，防止远程提交
  * 服务器端文件存储目录
  * 对文件上传后的管理
```

<form action="upload_file.php" method="post"
enctype="multipart/form-data">
<label for="file">Filename:</label>
<input type="file" name="file" id="file" /> 
<br />
<input type="submit" name="submit" value="Submit" />
</form>

```
创建上传脚本
"upload_file.php" 文件含有供上传文件的代码：
```

<?php
if ($_FILES["file"]["error"] > 0)
  {
  echo "Error: " . $_FILES["file"]["error"] . "<br />";
  }
else
  {
  echo "Upload: " . $_FILES["file"]["name"] . "<br />";
  echo "Type: " . $_FILES["file"]["type"] . "<br />";
  echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
  echo "Stored in: " . $_FILES["file"]["tmp_name"];
  }
?>

```
当文件被上传时，该文件将保存在临时目录中，钟乳石通过php.ini文件的upload_tmp_dir指令设置的。如果没有显式设置，默认使用服务器上的主临时目录。在脚本执行完之前临时文件会被删除。通过使用 PHP 的全局数组 $_FILES，你可以从客户计算机向远程服务器上传文件。
第一个参数是表单的 input name，第二个下标可以是 "name", "type", "size", "tmp_name" 或 "error"。就像这样：
  * $_FILES["file"]["name"] - 被上传文件的名称
  * $_FILES["file"]["type"] - 被上传文件的类型
  * $_FILES["file"]["size"] - 被上传文件的大小，以字节计
  * $_FILES["file"]["tmp_name"] - 存储在服务器的文件的临时副本的名称
  * $_FILES["file"]["error"] - 由文件上传导致的错误代码，为0表示上传成功。
UPLOAD_ERR_OK: 0 正常，上传成功 
UPLOAD_ERR_INI_SIZE: 1 上传文件大小超过服务器允许上传的最大值，php.ini中设置upload_max_filesize选项限制的值 
UPLOAD_ERR_FORM_SIZE: 2 上传文件大小超过HTML表单中隐藏域MAX_FILE_SIZE选项指定的值 
UPLOAD_ERR_PARTIAL: 3 文件只有部分被上传 
UPLOAD_ERR_NO_TMP_DIR: 6 没有找不到临时文件夹 
UPLOAD_ERR_CANT_WRITE: 7 文件写入失败 
UPLOAD_ERR_EXTENSION: 8 php文件上传扩展没有打开 
这是一种非常简单文件上传方式。基于安全方面的考虑，您应当增加有关什么用户有权上传文件的限制。
上传限制
在这个脚本中，我们增加了对文件上传的限制。用户只能上传 .gif 或 .jpeg 文件，文件大小必须小于 20 kb：
```

<?php

if ((($_FILES["file"]["type"] == "image/gif")
|| ($_FILES["file"]["type"] == "image/jpeg")
|| ($_FILES["file"]["type"] == "image/pjpeg"))
&& ($_FILES["file"]["size"] < 20000))
  {
  if ($_FILES["file"]["error"] > 0)
    {
    echo "Error: " . $_FILES["file"]["error"] . "<br />";
    }
  else
    {
    echo "Upload: " . $_FILES["file"]["name"] . "<br />";
    echo "Type: " . $_FILES["file"]["type"] . "<br />";
    echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
    echo "Stored in: " . $_FILES["file"]["tmp_name"];
    }
  }
else
  {
  echo "Invalid file";
  }

?>

```
```

<html>
<head>
  <title>Uploading...</title>
</head>
<body>
<h1>Uploading file...</h1>
<?php

//Check to see if an error code was generated on the upload attempt
  if ($_FILES['userfile']['error'] > 0) //文件是否上传成功
  {
    echo 'Problem: ';
    switch ($_FILES['userfile']['error']) //上传错误原因分析
    {
      case 1:	echo 'File exceeded upload_max_filesize';
	  			break;
      case 2:	echo 'File exceeded max_file_size';
	  			break;
      case 3:	echo 'File only partially uploaded';
	  			break;
      case 4:	echo 'No file uploaded';
	  			break;
	  case 6:   echo 'Cannot upload file: No temp directory specified.';
	  			break;
	  case 7:   echo 'Upload failed: Cannot write to disk.';
	  			break;
    }
    exit;
  }

  // Does the file have the right MIME type?
  if ($_FILES['userfile']['type'] != 'text/plain') //限制mimetype
  {
    echo 'Problem: file is not plain text';
    exit;
  }

  // put the file where we'd like it
  $upfile = '/uploads/'.$_FILES['userfile']['name']; //文件目标路径

  if (is_uploaded_file($_FILES['userfile']['tmp_name'])) //判断是否真的有上传文件。这是必须的。因为客户端可以伪造，这是防止这种情形的发生。
  {
     if (!move_uploaded_file($_FILES['userfile']['tmp_name'], $upfile)) //移动临时文件到目标路径。也就是保存上传的文件。
     {
        echo 'Problem: Could not move file to destination directory';
        exit;
     }
  } 
  else 
  {
    echo 'Problem: Possible file upload attack. Filename: ';
    echo $_FILES['userfile']['name'];
    exit;
  }
  echo 'File uploaded successfully<br><br>'; 
?>
</body>
</html>

```

###  保存被上传的文件 
上面的例子在服务器的 PHP **临时文件夹**创建了一个被上传文件的**临时副本**。
这个**临时的复制文件会在脚本结束时消失**。要保存被上传的文件，我们需要把它拷贝到另外的位置：
```

    if (file_exists("upload/" . $_FILES["file"]["name"]))
      {
      echo $_FILES["file"]["name"] . " already exists. ";
      }
    else
      {
      move_uploaded_file($_FILES["file"]["tmp_name"],
      "upload/" . $_FILES["file"]["name"]);
      echo "Stored in: " . "upload/" . $_FILES["file"]["name"];
      }
    }

```
上面的脚本检测了是否已存在此文件，如果不存在，则把文件拷贝到指定的文件夹。
注释：这个例子把文件保存到了名为 "upload" 的新文件夹。

###  PHP5.4文件上传进度支持 
http://www.laruence.com/2011/10/10/2217.html
文件上传进度反馈, 这个需求在当前是越来越普遍, 比如大附件邮件. 
**在PHP5.4以前, 我们可以通过APC提供的功能来实现. 或者使用PECL扩展uploadprogress来实现.**
虽然说, 它们能很好的解决现在的问题, 但是也有很明显的不足:
1. 他们都需要额外安装(我们并没有打算把APC加入PHP5.4)
2. 它们都使用本地机制来存储这些信息, APC使用共享内存, 而uploadprogress使用文件系统(不考虑NFS), 这在多台前端机的时候会造成麻烦.
从PHP的角度来说, 最好的储存这些信息的地方应该是SESSION, 首先它是PHP原生支持的机制. 其次, 它可以被配置到存放到任何地方(支持多机共享).
**这个新特性, 提供了一些新的INI配置**, 他们和APC的相关配置很类似:
  * session.upload_progress.enabled[=1] : 是否启用上传进度报告(默认开启)
  * session.upload_progress.cleanup[=1] : 是否在上传完成后及时删除进度数据(默认开启, 推荐开启).
  * session.upload_progress.prefix[=upload_progress_] : 进度数据将存储在_SESSION[session.upload_progress.prefix . _POST[session.upload_progress.name]]
  * session.upload_progress.name[=PHP_SESSION_UPLOAD_PROGRESS] : 如果_POST[session.upload_progress.name]没有被设置, 则不会报告进度.
  * session.upload_progress.freq[=1%] : 更新进度的频率(已经处理的字节数), 也支持百分比表示’%’.
  * session.upload_progress.min_freq[=1.0] : 更新进度的时间间隔(秒级)
对于如下的上传表单:
```

<form action="upload.php" method="POST" enctype="multipart/form-data">
 <input type="hidden"
     name="<?php echo ini_get("session.upload_progress.name"); ?>" value="laruence" />
 <input type="file" name="file1" />
 <input type="file" name="file2" />
 <input type="submit" />
</form>

```
如果我们上传一个足够大的文件(网速要是足够慢就更好:P), **我们就可以从_SESSION中, 得到类似下面的进度信息**:
```

$_SESSION["upload_progress_laruence"] = array(
 "start_time" => 1234567890,   // 请求时间
 "content_length" => 57343257, // 上传文件总大小
 "bytes_processed" => 453489,  // 已经处理的大小
 "done" => false,              // 当所有上传处理完成后为TRUE
 "files" => array(
  0 => array(
   "field_name" => "file1",       // 表单中上传框的名字
   // The following 3 elements equals those in $_FILES
   "name" => "foo.avi",
   "tmp_name" => "/tmp/phpxxxxxx",
   "error" => 0,
   "done" => true,                // 当这个文件处理完成后会变成TRUE
   "start_time" => 1234567890,    // 这个文件开始处理时间
   "bytes_processed" => 57343250, // 这个文件已经处理的大小
  ),
  // An other file, not finished uploading, in the same request
  1 => array(
   "field_name" => "file2",
   "name" => "bar.avi",
   "tmp_name" => NULL,
   "error" => 0,
   "done" => false,
   "start_time" => 1234567899,
   "bytes_processed" => 54554,
  ),
 )
);

```
###  示例1-图片上传 
通过$_FILES函数，获得临时文件名，文件类型，文件尺寸，文件名等信息
用 ` is_uploaded_file 函数 `判断，用户是否上传了图片，然后用mkdir创建文件夹，
使用$newfile=date('YmdHis'); $filename=$dir."/".$newfile.$ext; 自定义上传的文件名
最后，用` move_uploaded_file函数 `来实现把文件从临时区移动到指定的文件夹
```

<?  
header('Content-Type:text/html; charset=utf-8');  
include('function.php');  
$error=$_FILES['pic']['error'];  
$name=$_FILES['pic']['name'];  
$tmp_name=$_FILES['pic']['tmp_name'];  
$type=$_FILES['pic']['type'];  
$size=$_FILES['pic']['size'];  
if($name<>"")  
{  
    $ext=substr($name,-4);  
    if($ext!='.jpg' && $ext!='.bmp' && $ext!='.gif' && $ext!='.png' && $ext!='jpeg')  
    {  
        echo "<script language='javascript'>alert('您选择的图片格式不正确');history.go(-1);</script>";  
    }  
    else 
    {  
          
        if(is_uploaded_file($tmp_name))  
        {  
            $dir=date('Y-m-d');  
            mk($dir);  
            $newfile=date('YmdHis');  
            $filename=$dir."/".$newfile.$ext;  
             if(!move_uploaded_file($tmp_name,$filename))  
             {  
                 echo "<script language='javascript'>alert('对不起，文件移动失败');history.go(-1);</script>";  
                 exit();  
             }  
             else 
             {  
                 echo "<script language='javascript'>alert('文件上传成功');location.href='upfile.php';</script>";  
             }  
              
        }  
    }  
}  
else 
{  
    echo "<script language='javascript'>alert('请选择文件');history.go(-1);</script>";  
}  
?> 

```

##  文件下载 
http://www.laruence.com/2012/05/02/2613.html
```

<?php
    $file = "/tmp/dummy.tar.gz";
    header("Content-type: application/octet-stream");
    header('Content-Disposition: attachment; filename="' . basename($file) . '"');
    header("Content-Length: ". filesize($file));
    readfile($file);

```
但是这个有一个问题, 就是如果文件是中文名的话, 有的用户可能下载后的文件名是乱码.
于是, 我们做一下修改(参考: :
```

<?php
    $file = "/tmp/中文名.tar.gz";
    $filename = basename($file);
    header("Content-type: application/octet-stream");
 
    //处理中文文件名
    $ua = $_SERVER["HTTP_USER_AGENT"];
    $encoded_filename = rawurlencode($filename);
    if (preg_match("/MSIE/", $ua)) {
     header('Content-Disposition: attachment; filename="' . $encoded_filename . '"');
    } else if (preg_match("/Firefox/", $ua)) {
     header("Content-Disposition: attachment; filename*=\"utf8''" . $filename . '"');
    } else {
     header('Content-Disposition: attachment; filename="' . $filename . '"');
    }
    header("Content-Length: ". filesize($file));
    readfile($file);

```
