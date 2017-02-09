title: phplearn06 

#  PHP学习之与命令行和外部程序交互 
##  方式一、执行操作符与命令行交互 
执行操作符实际上是**一对反向单引号（` `）**.PHP将试着将反向单引号里面的命令**当作服务器端的命令行来执行**。表达式的值就是命令执行的结果。
使用反引号运算符“`”的效果与函数 ` shell_exec() ` 相同。string shell_exec ( string $cmd )
例如：linux下：
```

echo `ls -la`;

```
windows下:(windows下命令行输出默认为gbk编码，所以需要通过` iconv()函数进行字符编码转换 `)
```

echo iconv('GB2312','UTF-8',`dir c:`);

```
##  方式二、使用exec()函数运行服务器命令 
string exec ( string $command [, array &$output [, int &$return_var ]] )exec() 执行 command 参数所指定的命令。
  * output如果提供了 output 参数， **那么会用命令执行的输出填充此数组， 每行输出填充数组中的一个元素。** 数组中的数据不包含行尾的空白字符，例如 \n 字符。 请注意，如果数组中已经包含了部分元素，exec() 函数会在数组末尾追加内容。如果你不想在数组末尾进行追加， 请在传入 exec() 函数之前 对数组使用 unset() 函数进行重置。
  * return_var如果同时提供 output 和 return_var 参数， **命令执行后的返回状态会被写入到此变量。**
返回命令执行结果的**最后一行内容。**
##  方式三、使用passthru — 执行外部程序并且显示原始输出 
void passthru ( string $command [, int &$return_var ] )
同 exec() 函数类似， passthru() 函数 也是用来执行外部命令（command）的。 当所执行的 Unix 命令输出二进制数据，** 并且需要直接传送到浏览器的时候，** 需要用此函数来替代 exec() 或 system() 函数。 常用来执行诸如 pbmplus 之类的可以直接输出图像流的命令。 通过设置 Content-type 为 image/gif， 然后调用 pbmplus 程序输出 gif 文件， 就可以从 PHP 脚本中直接输出图像到浏览器。

##  方式四、system — 执行外部程序，并且显示输出 
string system ( string $command [, int &$return_var ] ) 本函数执行 command 参数所指定的命令， 并且输出执行结果。
成功则返回命令输出的最后一行， 失败则返回 FALSE
如何程序使用此函数启动，为了能保持在后台运行，此程序必须将输出重定向到文件或其它输出流。否则会导致 PHP 挂起，直至程序执行结束。
使用` escapeshellcmd() `可以阻止恶意命令。
system (escapeshellcmd($cmd)）

##  方式五、proc_open启动外部进程，并且打开用来输入/输出的文件指针
resource proc_open ( string $cmd , array $descriptorspec , array &$pipes [, string $cwd [, array $env [, array $other_options ]]] )
返回表示进程的资源类型， 当使用完毕之后，**请调用 proc_close() 函数来关闭此资源**。 如果失败，返回 FALSE。
如果你只需要单向的进程管道， 使用 ` popen() ` 函数会更加简单。
参数说明:
descriptorspec:**一个索引数组**。 数组的键表示描述符( 0 表示标准输入（stdin），1 表示标准输出（stdout），2 表示标准错误（stderr）)，数组元素值表示 PHP 如何将这些描述符传送至子进程。
pipes:**将被置为索引数组**， 其中的元素是被执行程序创建的**管道对应到 PHP 这一端的文件指针**。
cwd:**要执行命令的初始工作目录。 必须是 绝对 路径**， 设置此参数为 NULL 表示使用默认值（当前 PHP 进程的工作目录）。
env：**要执行的命令所使用的环境变量**。 设置此参数为 NULL 表示使用和当前 PHP 进程相同的环境变量。
```

<?php
$descriptorspec = array(
   0 => array("pipe", "r"),  // 标准输入，子进程从此管道中读取数据
   1 => array("pipe", "w"),  // 标准输出，子进程向此管道中写入数据
   2 => array("file", "/tmp/error-output.txt", "a") // 标准错误，写入到一个文件
);

$cwd = '/tmp';
$env = array('some_option' => 'aeiou');

$process = proc_open('php', $descriptorspec, $pipes, $cwd, $env);

if (is_resource($process)) {
    // $pipes 现在看起来是这样的：
    // 0 => 可以向子进程标准输入写入的句柄
    // 1 => 可以从子进程标准输出读取的句柄
    // 错误输出将被追加到文件 /tmp/error-output.txt

    fwrite($pipes[0], '<?php print_r($_ENV); ?>');
    fclose($pipes[0]);

    echo stream_get_contents($pipes[1]);
    fclose($pipes[1]);
    

    // 切记：在调用 proc_close 之前关闭所有的管道以避免死锁。
    $return_value = proc_close($process);

    echo "command returned $return_value\n";
}
?>

```