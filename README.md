## 安装
- [Laravel](#laravel)
- [Lumen](#lumen)

### Laravel

该软件包可用于 Laravel 5.6 或更高版本。

您可以通过 composer 安装软件包：

``` bash
composer require starrysea/multi-auth
```
### Lumen

您可以通过 composer 安装软件包：

``` bash
composer require starrysea/multi-auth
```

## 用法

```php
// config/auth.php

// ...

'guards' => [
    // ...

    'admin' => [
        'driver' => 'session',
        'provider' => 'admins',
    ],

    // ...
],

'providers' => [
    // ...

    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Models\Admin::class,
    ],
    
    // ...
],

// ...
```

```php
// routes/web.php

// 登录及登出
Route::get('login', 'Admin\LoginController@showLoginForm');
Route::post('login', 'Admin\LoginController@login')->name('admin.login');
Route::post('logout', 'Admin\LoginController@logout')->name('admin.logout');

Route::get('/', 'Admin\IndexController@index')->middleware('auth:admin');
```

```php
// app/Exceptions/Handler.php

class Handler extends ExceptionHandler
{
    // ...
    
    /**
     * 重写没有登录时要跳转的登录页面
     *
     * @param \Illuminate\Http\Request $request
     * @param AuthenticationException $exception
     * @return \Illuminate\Http\JsonResponse|\Illuminate\Http\RedirectResponse
     */
    protected function unauthenticated($request, AuthenticationException $exception)
    {
        if ($request->expectsJson()){
            return response()->json(['message' => $exception->getMessage()], 401);
        }elseif (in_array('admin',$exception->guards())){
            return redirect()->guest(route('admin.login'));
        }else{
            return redirect()->guest(route('login'));
        }
    }
}
```

```php
use Illuminate\Foundation\Auth\AuthenticatesUsers;
use Illuminate\Support\Facades\Auth;
use Starrysea\MultiAuth\MultiUsers;

class MultiUsersGatherTest
{
    // 引入处理应用程序的身份验证用户组件
    use AuthenticatesUsers, MultiUsers{
        MultiUsers::logout insteadof AuthenticatesUsers;
        MultiUsers::sendFailedLoginResponse insteadof AuthenticatesUsers;
    }

    // 配置登录成功后重定向地址
    protected $redirectTo = 'admin';

    // 重写登录账号为登录名字段
    public function username()
    {
        return 'username';
    }

    // 创建一个新的控制器实例
    public function __construct()
    {
        $this->middleware('guest:admin')->except('logout');
    }

    // 重写显示应用程序的登录表单
    public function showLoginForm()
    {
        return view('admin.login');
    }

    // 重写验证过程中使用的身份信息
    protected function guard()
    {
        return Auth::guard('admin');
    }
}
```
