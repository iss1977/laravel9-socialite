# **Laravel login with social credentials**

Login in a Laravel application using login with **google**, **facebook** and **github**

## Developed using: 
-  video tutorial from [Code For You](https://www.youtube.com/channel/UCCGA9iWRQNF0VmWHc14_8Aw/). Link to video [Youtube](https://www.youtube.com/watch?v=jIckLu1cKew)



- Tutorial from [www.tutsmake.com](https://www.tutsmake.com/laravel-9-socialite-login-with-google-account-example/) 

    _and_

- Laravel documentation
---

## Development stepts:

1. Generating fresh application
    - laravel new laravel/laravel
    - composer require laravel/ui 
    - php artisan ui bootstrap --auth
    - npm install
    - npm run dev


2. Configure a Google Cloud App to create Client ID and Client Secret to login using **google**

3. Configure `config/service.php`
```
'google' => [
    'client_id' => 'xxxx',
    'client_secret' => 'xxx',
    'redirect' => 'http://127.0.0.1:8000/callback/google',
], 
```

4. Install Laravel Socialite 
    ```composer require laravel/socialite```
and add it to Service Providers in ```config/app.php```

```php
'providers' => [
    /*
    * Package Service Providers...
    */
    Laravel\Socialite\SocialiteServiceProvider::class,
]
```
Also add the Facade in the same file:

```php
'aliases' => [
    ...
    'Socialite' => Laravel\Socialite\Facades\Socialite::class,
]
```

5. Create migration to add login fields to users table:
```php artisan make:migration add_social_login_field```

```php
<?php
   
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
    
class AddSoicalLoginField extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('users', function ($table) {
            $table->string('social_id')->nullable();
            $table->string('social_type')->nullable();
        });
    }
    
    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('users', function ($table) {
            $table->dropColumn('social_id');
           $table->dropColumn('social_type');
         });
    }
}
```
Modify User Model accordingly:
```php
<?php
 
namespace App\Models;
 
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\TwoFactorAuthenticatable;
use Laravel\Jetstream\HasProfilePhoto;
use Laravel\Sanctum\HasApiTokens;
 
class User extends Authenticatable
{
    use HasApiTokens;
    use HasFactory;
    use HasProfilePhoto;
    use Notifiable;
    use TwoFactorAuthenticatable;
 
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'email',
        'password',
        'social_id',
        'social_type'
    ];
 
    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password',
        'remember_token',
        'two_factor_recovery_codes',
        'two_factor_secret',
    ];
 
    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
 
    /**
     * The accessors to append to the model's array form.
     *
     * @var array
     */
    protected $appends = [
        'profile_photo_url',
    ];
}
```

6. Run the migration

    ```php artisan migrate```

7. Add routes in web.php

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Auth\GoogleSocialiteController;


Route::get('auth/google', [GoogleSocialiteController::class,  'redirectToGoogle']);
Route::get('callback/google', [GoogleSocialiteController::class, 'handleCallback']);
```

8. Make a controller to handle the requests

    ```php artisan make:controller GoogleSocialiteController```


```php
<?php
   
namespace App\Http\Controllers\Auth;
   
use App\Http\Controllers\Controller;
use Socialite;
use Auth;
use Exception;
use App\Models\User;
   
class GoogleSocialiteController extends Controller
{
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function redirectToGoogle()
    {
        return Socialite::driver('google')->redirect();
    }
       
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function handleCallback()
    {
        try {
     
            $user = Socialite::driver('google')->user();
      
            $finduser = User::where('social_id', $user->id)->first();
      
            if($finduser){
      
                Auth::login($finduser);
     
                return redirect('/home');
      
            }else{
                $newUser = User::create([
                    'name' => $user->name,
                    'email' => $user->email,
                    'social_id'=> $user->id,
                    'social_type'=> 'google',
                    'password' => encrypt('my-google')
                ]);
     
                Auth::login($newUser);
      
                return redirect('/home');
            }
     
        } catch (Exception $e) {
            dd($e->getMessage());
        }
    }
}
```

9. Integrate a link to the login route in login blade :

```html
<a href="{{ url('auth/google') }}" style="margin-top: 0px !important;background: green;color: #ffffff;padding: 5px;border-radius:7px;" class="ml-2">
    <strong>Google Login</strong>
</a> 
```

10. Done! Run the server.



