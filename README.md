# Laravel 8 Multi Auth (Authentication) Tutorial
In this example, you will learn laravel 8 multi auth. i would like to share with you laravel 8 multiple auth. This post will give you simple example of laravel 8 multiple authentication.

**Step 1: Install Laravel 8**

first of all we need to get fresh Laravel 8 version application using bellow command, So open your terminal OR command prompt and run bellow command:
```
composer create-project --prefer-dist laravel/laravel blog
```

**Step 2: Database Configuration**

In second step, we will make database configuration for example database name, username, password etc for our crud application of laravel 8. So let's open .env file and fill all details like as bellow:

**.env**

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=here your database name(blog)
DB_USERNAME=here database username(root)
DB_PASSWORD=here database password(root)
```

**Step 3: Update Migration and Model**<br>
In this step, we need to add new row "is_admin" in users table and model. than we need to run migration. so let's change that on both file.<br>
<code>database/migrations/000_create_users_table.php</code>
```
<?php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
class CreateUsersTable extends Migration {
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up() {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email');
            $table->timestamp('email_verified_at')->nullable();
            $table->boolean('is_admin')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }
    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down() {
        Schema::dropIfExists('users');
    }
}

```
<code>app/Models/User.php</code>
```
<?php
namespace App\Models;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
class User extends Authenticatable {
    use HasFactory, Notifiable;
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name', 'email', 'password', 'is_admin'];
    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = ['password', 'remember_token', ];
    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = ['email_verified_at' => 'datetime', ];
}
```
Now we need to run migration.<br>
so let's run bellow command:
```
php artisan migrate
```
**Step 4: Create Auth using scaffold**
Now, in this step, we will create auth scaffold command to create login, register and dashboard. so run following commands:<br>
<code>Laravel 8 UI Package</code>
```
composer require laravel/ui 
```
**Generate auth**
```
php artisan ui bootstrap --auth 
```
```
npm install
```
```
npm run dev
```
**Step 5: Create IsAdmin Middleware**
In this step, we require to create admin middleware that will allows only admin access users to that routes. so let's create admin user with following steps.<br>
```
php artisan make:middleware IsAdmin
```
<code>app/Http/middleware/IsAdmin.php</code>
```
<?php
namespace App\Http\Middleware;
use Closure;
class IsAdmin {
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next) {
        if (auth()->user()->is_admin == 1) {
            return $next($request);
        }
        return redirect(‘home’)->with(‘error’, "You don't have admin access.");
    }
}
```
<code>app/Http/Kernel.php</code>
```
....

protected $routeMiddleware = [

    'auth' => \App\Http\Middleware\Authenticate::class,

    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,

    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,

    'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,

    'can' => \Illuminate\Auth\Middleware\Authorize::class,

    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,

    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,

    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,

    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,

    'is_admin' => \App\Http\Middleware\IsAdmin::class,

];

....
```
**Step 6: Create Route**
Here, we need to add one more route for admin user home page so let's add that route in web.php file.<br>
<code>routes/web.php</code>
```
<?php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\HomeController;
/*

|--------------------------------------------------------------------------

| Web Routes

|--------------------------------------------------------------------------

|

| Here is where you can register web routes for your application. These

| routes are loaded by the RouteServiceProvider within a group which

| contains the "web" middleware group. Now create something great!

|

*/
Route::get('admin/home', [HomeController::class, 'adminHome'])->name('admin.home')->middleware('is_admin');
```
**Step 7: Add Method on Controller**
Here, we need add adminHome() method for admin route in HomeController. so let's add like as bellow:<br>
<code>app/Http/Controllers/HomeController.php</code>
```
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
class HomeController extends Controller {
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct() {
        $this->middleware('auth');
    }
    /**
     * Show the application dashboard.
     *
     * @return \Illuminate\Contracts\Support\Renderable
     */
    public function index() {
        return view('home');
    }
    /**
     * Show the application dashboard.
     *
     * @return \Illuminate\Contracts\Support\Renderable
     */
    public function adminHome() {
        return view('adminHome');
    }
}
```
**Step 8: Create Blade file**
In this step, we need to create new blade file for admin and update for user blade file. so let's change it.
<code>resources/views/home.blade.php</code>
```
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">Dashboard</div>
                <div class="card-body">
                    You are normal user.
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```
<code>resources/views/adminHome.blade.php</code>
```
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">Dashboard</div>
                <div class="card-body">
                    You are Admin.
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```
**Step 9: Update on LoginController**
In this step, we will change on LoginController, when user will login than we redirect according to user access. if normal user than we will redirect to home route and if admin user than we redirect to admin route. so let's change.
<code>app/Http/Controllers/Auth/LoginController.php</code>
```
<?php
namespace App\Http\Controllers\Auth;
use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\AuthenticatesUsers;
use Illuminate\Http\Request;
class LoginController extends Controller {
    /*
    
    |--------------------------------------------------------------------------
    
    | Login Controller
    
    |--------------------------------------------------------------------------
    
    |
    
    | This controller handles authenticating users for the application and
    
    | redirecting them to your home screen. The controller uses a trait
    
    | to conveniently provide its functionality to your applications.
    
    |
    
    */
    use AuthenticatesUsers;
    /**
     * Where to redirect users after login.
     *
     * @var string
     */
    protected $redirectTo = '/home';
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct() {
        $this->middleware('guest')->except('logout');
    }
    public function login(Request $request) {
        $input = $request->all();
        $this->validate($request, ['email' => 'required|email', 'password' => 'required', ]);
        if (auth()->attempt(array('email' => $input['email'], 'password' => $input['password']))) {
            if (auth()->user()->is_admin == 1) {
                return redirect()->route('admin.home');
            } else {
                return redirect()->route('home');
            }
        } else {
            return redirect()->route('login')->with('error', 'Email-Address And Password Are Wrong.');
        }
    }
}
```
**Step 10: Create Seeder**
We will create seeder for create new admin and normal user. so let's create seeder using following command:
```
php artisan make:seeder CreateUsersSeeder
```
<code>database/seeds/CreateUsersSeeder.php</code>
```
<?php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;

class CreateUsersSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $user = [

            [
                'name' => 'Admin',
                'email' => 'admin@example.com',
                'is_admin' => '1',
                'password' => bcrypt('123456'),

            ],
            [

                'name' => 'User',
                'email' => 'user@example.com',
                'is_admin' => '0',
                'password' => bcrypt('123456'),

            ],

        ];

        foreach ($user as $key => $value) {

            User::create($value);

        }
    }
}
```
Now let's run seeder:
```
php artisan db:seed --class=CreateUsersSeeder
```
Ok, now we are ready to run.<br>
So let's run project using this command:
```
php artisan serve
```
**Admin User**
```
Email: admin@example.com
Password: 123456
```

**User User**
```
Email: user@example.com
Password: 123456
```
