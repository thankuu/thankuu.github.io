---
title: "Laravel 5.3中使用Laravel Passport构建后端授权认证API"
tags:
    - laravel
    - php

---
Laravel在5.3中引入了新的官方OAuth扩展Laravel Passport，之前在5.1/5.2时一直是用dingo+jwt这一套来构建后端api，最近正好要构建新项目，想着试试官方这一个拓展如何。
## 安装
官方文档中有完整的安装调用过程，使用composer：

    composer require laravel/passport
像使用其他组件一样，我们需要在`config/app/php`的`providers`数组中注册Passport：

    Laravel\Passport\PassportServiceProvider::class,
然后我们需要执行`migrate`创建相关的客户端数据表和令牌数据表,和`passport:install`来生成一些加密秘钥等，这些在官方文档中都有详细介绍。
## 构建认证函数
官方文档中针对如何简单使用做了初步介绍，下面我试着构造一个完整的客户端授权流程，首先创建`ApiController`：

    php artisen make:controller Apicontroller
在`ApiController`中引入`AuthenticatesUsers`模块：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Foundation\Auth\AuthenticatesUsers;
    use Illuminate\Http\Request;

    class ApiController extends Controller
    {
        use  AuthenticatesUsers;

    public function __construct()
    {
        $this->middleware('api');
    }
我们的客户端需要通过密码授权的方式来认证，我们需要在`ApiController`中重写`AuthenticatesUsers`部分功能函数来实现整个完整的授权流程,在这里我们调用Passport提供的`oauth/token`接口：

    //调用认证接口获取授权码
    protected function authenticateClient(Request $request)
    {
        $credentials = $this->credentials($request);

        $data = $request->all();

        $request->request->add([
            'grant_type' => $data['grant_type'],
            'client_id' => $data['client_id'],
            'client_secret' => $data['client_secret'],
            'username' => $credentials['phone'],
            'password' => $credentials['password'],
            'scope' => ''
        ]);

        $proxy = Request::create(
            'oauth/token',
            'POST'
        );

        $response = \Route::dispatch($proxy);

        return $response;
    }
    //以下为重写部分
    protected function authenticated(Request $request)
    {
        return $this->authenticateClient($request);
    }

    protected function sendLoginResponse(Request $request)
    {
        $this->clearLoginAttempts($request);

        return $this->authenticated($request);
    }

    protected function sendFailedLoginResponse(Request $request)
    {
        $msg = $request['errors'];
        $code = $request['code'];
        return $this->failed($msg,$code);
    }
在这里我们会遇到一个问题，官方文档中没有提及，就是我们要如何使用自定的用户名进行授权，找寻源码，其实在`Laravel\Passport\Bridge\UserRepository.php`的`getUserEntityByUserCredentials()`函数中会看到这段代码：

    if (method_exists($model, 'findForPassport')) {
        $user = (new $model)->findForPassport($username);
    } else {
        $user = (new $model)->where('email', $username)->first();
    }
我们只需要在我们配置在config/auth.php的模型中添加这段代码就可以完成自定义授权用户名：

     public function findForPassport($username) {
        return $this->where('phone', $username)->first();
    }
这样一个授权流程基本完成，接下来创建一个`Api/LoginController`:

    <?php

    namespace App\Http\Controllers\Api;

    use App\Http\Controllers\ApiController;
    use Illuminate\Http\Request;
    use App\Models\User;                \\User.php我移动到了Models目录
    use Validator;

    class LoginController extends ApiController
    {
        // 登录用户名标示为phone字段
        public function username()
        {
            return 'phone';
        }
        //登录接口，调用了ApiController中一些其他函数succeed\failed，上文未提及，用于接口格式化输出
        public function login(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'phone'    => 'required|exists:users',
            'password' => 'required|between:6,32',
            ]);

            if ($validator->fails()) {
            $request->request->add([
                'errors' => $validator->errors()->toArray(),
                'code' => 401,
                ]);                     
            return $this->sendFailedLoginResponse($request);
        }

        $credentials = $this->credentials($request);

        if ($this->guard('api')->attempt($credentials, $request->has('remember'))) {
            return $this->sendLoginResponse($request);
        }

        return $this->failed('login failed',401);
        }
    }
## 构建路由    
最后`routes/api.php`中加入我们需要的路由：

    Route::group([
        'prefix'=>'/v1',
        'middleware' => ['api']
    ], function () {
        Route::post('/user/login','Api\LoginController@login');
    });
## 测试    
最后在postmen中调用接口：
![image_1b2543doiq521a5c17lh1h3k14ve9.png-81.7kB][1]


  [1]: http://static.zybuluo.com/thankuu/gcptj01cj4f3zk3mtrnbm82k/image_1b2543doiq521a5c17lh1h3k14ve9.png
  结果正确返回。
