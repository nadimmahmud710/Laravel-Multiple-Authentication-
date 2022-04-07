step 1: Setup  database (DB) and set the credentials in environment file (.env) in installed laravel application.
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE="your_database"
DB_USERNAME="your_username"
DB_PASSWORD="your_password"

Step 2: Go to your application and open the folder by following: app/Providers/AppServiceProvider.php, open this file and add the below code
use Illuminate\Support\Facades\Schema;

public function boot()
{
    Schema::defaultStringLength(191);
}

Step 3: Now, lets set the login and registration system using laravel auth command by below code:

php artisan make:auth

Step 4: All the setup for login and register is successfully done. Now, let's edit user table which is available in following folder: database / migrations /
$table->tinyInteger('role_as')->default('0'); 0 for user, 1 for admin.
Step 5: Let's migrate this table in database by following commad:
php artisan migrate

Step 6: Now, Let's Create a custom middleware to check is the Logged_In user is admin or not.! The command for generating middleware as following:

php artisan make:middleware AdminMiddleware

after middleware generated go to AdminMiddleware as follows: app/Http/Middleware/AdminMiddleware.php and paste the below code:


    use Illuminate\Support\Facades\Auth;

    public function handle($request, Closure $next)
    {
        if(Auth::check())
        {
            if(Auth::user()->role_as == 'admin')
            {
                return $next($request);
            }
            else
            {
                return redirect('/home')->with('status','Access Denied! as you are not as admin');
            }
        }
        else
        {
            return redirect('/home')->with('status','Please Login First');
        }
    }
    

Step 7: Now Register Middleware just created named AdminMiddleware with the following path: app/Http/Kernel.php and goto the routeMiddleware and paste the below code:


    protected $routeMiddleware = [

        'isAdmin' => \App\Http\Middleware\AdminMiddleware::class, //Add to the end
    ];
    

step 8: Set Redirection when logging as user or admin or any other role provided. Goto path - app/Http/Controller/Auth/LoginController.php
 protected function authenticated()
 use Illuminate\Support\Facades\Auth;
    {
        if(Auth::user()->role_as == '1') //1 = Admin Login
        {
            return redirect('dashboard')->with('status','Welcome to your dashboard');
        }
        elseif(Auth::user()->role_as == '0') // Normal or Default User Login
        {
            return redirect('/')->with('status','Logged in successfully');
        }
    }
    
    
    Step 9: Lets Setup the Route for this Process of multiple authentication as give below:
    Route::group(['middleware' => ['auth','isAdmin']], function () {

   Route::get('/dashboard', function () {
      return view('admin.dashboard');
   });

});

