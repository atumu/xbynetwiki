title: laravel框架6 

#  Laravel之文件上传与下载 
##  上传文件 
**获取上传文件**
你可以使用 ` Illuminate\Http\Request ` 实例中的`  file 方法 `获取上传的文件。file 方法返回的对象是 ` Symfony\Component\HttpFoundation\File\UploadedFile ` 类的实例，**该类继承了 PHP 的 SplFileInfo 类**并提供了许多和文件交互的方法：
```

$file = $request->file('photo');

```
**确认文件是否有上传**
```

if ($request->hasFile('photo')) {
    //
}

```
**确认上传的文件是否有效**
```

if ($request->file('photo')->isValid()) {
    //
}

```
**移动上传的文件**
```

$request->file('photo')->move($destinationPath);
$request->file('photo')->move($destinationPath, $fileName);

```
**其他上传文件的方法**
UploadedFile 的实例还有许多可用的方法，可以到该对象的 [API](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html) 文档了解这些方法的详细信息。

##  文件下载 
Response的` download 方法 `可以用于产生强制让用户的浏览器下载指定路径文件的响应。download 方法接受文件名称作为方法的第二个参数，此名称为用户下载文件时看见的文件名称。最后，你可以传递一个 HTTP 标头的数组作为第三个参数传入该方法：
```

return response()->download($pathToFile);
return response()->download($pathToFile, $name, $headers);

```
**注意：管理文件下载的扩展包 Symfony HttpFoundation，要求下载文件必须是 ASCII 文件名。**

