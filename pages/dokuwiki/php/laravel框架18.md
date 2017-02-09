title: laravel框架18 

#  Laravel之加密与哈希 
注意：加密与哈希是不同的概念：加密是双向算法，哈希是单向算法。
##  加密 

在使用 Laravel 的加密器前，你应该先设置 ` config/app.php ` 配置文件中的 **key 选项**，设置值需要是** 32 个字符**的随机字符串。如果没有适当地设置这个值，所有被 Laravel 加密的值都将是不安全的。
**加密一个值#**
你可以借助 ` Crypt facade ` 来加密一个值。这些值都会使用 OpenSSL 与 AES-256-CBC 来进行加密。此外，所有加密过后的值都会被签署文件消息验证码 (MAC)，以检测加密字符串是否被篡改过。
例如，我们可以使用 encrypt 方法加密机密信息，并把它保存在 Eloquent 模型中：
```

<?php
namespace App\Http\Controllers;

use Crypt;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 保存用户的机密消息。
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function storeSecret(Request $request, $id)
    {
        $user = User::findOrFail($id);

        $user->fill([
            'secret' => Crypt::encrypt($request->secret)
        ])->save();
    }
}

```
**解密一个值#**
当然，你可以使用 Crypt facade 上的 decrypt 方法来解密值。如果该值无法被适当地解密，例如文档消息验证码无效等因素，将会抛出一个 Illuminate\Contracts\Encryption\DecryptException 异常：
```

use Illuminate\Contracts\Encryption\DecryptException;

try {
    $decrypted = Crypt::decrypt($encryptedValue);
} catch (DecryptException $e) {
    //
}

```
##  哈希 
Laravel 通过 ` Hash facade ` 提供** Bcrypt** 加密来保存用户密码。**如果你在当前应用使用了 AuthController 控制器，它将` 自动 `使用 Bcrypt 加密来进行注册跟验证。**
由于 Bcrypt 的 「加密系数（word fator）」可以任意调整，这使它成为最好的加密选择。这代表每一次加密的时间可以随着硬件设备的升级而加长。
你可以通过调用 Hash facade 的 make 方法加密一个密码：
```

<?php
namespace App\Http\Controllers;

use Hash;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 为用户更新密码。
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function updatePassword(Request $request, $id)
    {
        $user = User::findOrFail($id);

        // 验证新密码的长度...

        $user->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();
    }
}


```
另外，你也可以使用 bcrypt 辅助方法：
```

bcrypt('plain-text');

```
**根据哈希值验证密码#**
` check 方法 `允许你通过一个指定的纯字符串跟哈希值进行验证。**如果你目前正使用 Laravel 内含的 AuthController，你可能不需要直接使用该方法，它已经包含在控制器当中并且会被自动调用。**
```

if (Hash::check('plain-text', $hashedPassword)) {
    // The passwords match...
}

```
**验证密码是否须重新加密#**
` needsRehash 函数 `允许你检查已加密的密码所使用的加密系数是否被修改：
```

if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}

```
