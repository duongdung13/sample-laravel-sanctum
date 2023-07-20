# sample-laravel-sanctum
Sample Laravel 8.x with package authentication Sanctum

#Giới thiệu
Laravel Sanctum cung cấp một hệ thống xác thực cho:
- SPA (ứng dụng đơn trang) 
- App mobile
- API base token

Sanctum cho phép user tạo nhiều API tokens cho tài khoản của họ. 
Các tokens này có khả năng chỉ định các action mà user đó có thể thực hiện

<b>API Tokens</b>

Sanctum sẽ sinh ra API tokens cho người dùng mà không cần đến sự phức tạp của Oauth. Nó giống như tính năng "access tokens" của GitHub. Hãy tưởng tượng ứng dụng của bạn trong phần cài đặt tài khoản có màn hình nơi user có thể tạo API token cho tài khoản của họ. Bạn có thể sử dụng Sanctum để tạo và quản lý các tokens này. Những tokens này thường có thời gian hết hạn rất dài (tính bằng năm) nhưng user có thể hủy bỏ chúng bất cứ lúc nào.

Laravel Sanctum cung cấp tính năng này bằng cách lưu trữ API tokens của user trong một bảng cơ sở dữ liệu duy nhất và xác thực các requests thông qua API token hợp lệ được gắn trên Authorization header.

<b>SPA Authentication</b>

Sanctum cung cấp một cách đơn giản để xử lý việc xác thực trong các SPA cần giao tiếp với API được hỗ trợ bởi Laravel. Các SPA này có thể tồn tại trong cùng một kho lưu trữ như ứng dụng Laravel của bạn hoặc có thể là một kho lưu trữ hoàn toàn riêng biệt, chẳng hạn như SPA được tạo bằng Vue CLI.

Đối với tính năng này, Sanctum không sử dụng bất kỳ loại token nào. Thay vào đó, Sanctum sử dụng các dịch vụ xác thực phiên dựa trên cookie được tích hợp sẵn của Laravel. Điều này cung cấp các lợi ích của CSRF protection, session authentication, cũng như bảo vệ chống rò rỉ thông tin xác thực qua XSS. Sanctum sẽ chỉ cố gắng xác thực bằng cookie khi request bắt nguồn từ giao diện người dùng SPA của bạn

#Cài đặt Sanctum

Yêu cầu laravel version 7.x trở lên 

Cài đặt package
```
composer require laravel/sanctum
```

Publish file config và migration của Sanctum
```
 php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

Chạy migrate tạo bảng và tạo dữ liệu mẫu
```
php artisan migrate --seed
```

Nếu muốn sử dụng Sanctum để xác thực một trang SPA, cần thêm Sanctum's middleware vào nhóm phần mềm api middleware trong tệp app/Http/Kernel.php của ứng dụng:

```
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

Thêm thư viện vào model để sử dụng, ví dụ mình đang sử dụng cho model User

```
use Laravel\Sanctum\HasApiTokens;
 
class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

#Tạo controller và route để sử dụng

Tạo route ở file routes/api.php

```
Route::post('/login', 'UserController@login');
Route::middleware('auth:sanctum')->get('/users', 'UserController@index');
```

Auth bằng sanctum sử dụng token type Bearer
```
Authorization = Bearer <API-Token>
```

Tạo file UserController.php action login
```
$request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (!$user || !Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    return response()->json([
        'status' => 200,
        'message' => 'Login Success',
        'response' => [
            'type' => 'Bearer',
            'access_token' => $user->createToken($request->device_name)->plainTextToken
        ],
    ]);
```

Chúng ta có thể thu hồi những token này bằng cách xóa trong database nhưng tất nhiên là sử dụng method có sẵn:

```
// Revoke all tokens...
$user->tokens()->delete();
 
// Revoke the token that was used to authenticate the current request...
$request->user()->currentAccessToken()->delete();
 
// Revoke a specific token...
$user->tokens()->where('id', $tokenId)->delete();
```

Action index()

```
return response()->json([
    'status' => 200,
    'message' => 'Success',
    'response' => User::get(),
]);
```
#Thử nghiệm bằng postman

Login tài khoản:

<img src="https://i.imgur.com/o5aj0Wx.png"/>

Sử dụng token trả về sau khi login, truyền token trên header type "Bearer"

<img src="https://i.imgur.com/dlt9fAh.png"/>


<b>Nguồn tham khảo</b>

https://laravel.com/docs/8.x/sanctum

https://viblo.asia/p/tao-api-va-authenticate-nhanh-chong-voi-package-laravel-sanctum-eW65G1EJZDO
