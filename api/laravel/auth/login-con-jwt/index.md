# Login con JWT
 
En esta documentación se detalla a un nivel simple las funciones del login y sus requerimientos.
 
---
 
Para poder realizar este proceso se necesitará.
 
* Laravel (De preferencia versiones iguales o superiores a la 5.4, aunque se pueden utilizar versiones anteriores).
 
* Composer.
 
* Visual Studio Code
 
* Saber utilizar la consola de comandos en Visual Studio Code
 
---
 
## Instalación del Auth por Composer
 
---
 
### Primer paso
 
Para poder instalar la última versión del auto de JWT se debe ejecutar el siguiente código.
 
    composer require tymon/jwt-auth
 
---
 
### Añadir proveedor de servicio
 
Una vez se ha realizado el paso anterior se deberá agregar un proveedor de servicios al array "providers" que se encuentra dentro de la ruta "config/app.php".
 
    'providers' => [
 
        ...
 
        Tymon\JWTAuth\Providers\LaravelServiceProvider::class,
    ]
 
Cabe destacar que este proceso solo se debe realizar en proyectos que utilicen Laravel 5.4 o inferior.
 
---
 
### Publicar la configuración
 
Una vez completados los pasos anteriores se debe publicar el paquete de configuración, para poder hacer esto se deberá utilizar el siguiente comando.
 
    php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
 
Esto creará la ruta "config/jwt.php" que servirá para configurar los aspectos básicos del paquete.
 
---
 
### Generar clave secreta
 
Se incluye un comando auxiliar que tiene la utilidad de crear una clave secreta para el usuario
 
    php artisan jwt:secret
 
Esto actualizará el archivo ".env" añadiendo la línea "JWT_SECRET=foobar"
 
Es la clave que se utilizará para firmar los tokens. La forma en que eso suceda exactamente dependerá del algoritmo que elija utilizar.


---

## Guía de Inicio Rápido
 
---
 
### Actualizar el modelo de usuario
 
Primero, se deberá implementar el contrato
 
    Tymon\JWTAuth\Contracts\JWTSubject
 
dentro del modelo de usuario. Una vez que se implementó el contrato se deberán implementar los métodos
 
    getJWTIdentifier()
 
y
 
    getJWTCustomClaims()
 
Todo esto dentro del modelo de usuario igualmente.
 
Abajo se mostrará un ejemplo de cómo se debería ver, obviamente, se deberá actualizar para que se acomode a el uso necesario
 
    <?php
 
    namespace App;
 
    use Tymon\JWTAuth\Contracts\JWTSubject;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
 
    class User extends Authenticatable implements JWTSubject
    {
        use Notifiable;
 
        // Rest omitted for brevity
 
        /**
        * Get the identifier that will be stored in the subject claim of the JWT.
        *
        * @return mixed
        */
        public function getJWTIdentifier()
        {
            return $this->getKey();
        }
 
        /**
        * Return a key value array, containing any custom claims to be added to the JWT.
        *
        * @return array
        */
        public function getJWTCustomClaims()
        {
            return [];
        }
    }
 
---
 
### Configurar Auth guard
 
Nota: Esto solo funcionara en las versiones de Laravel 5.2 hacia arriba
 
Dentro de la ruta "config/auth.php" se deben hacer unos cuantos cambios para permitir a Laravel utilizar el Auth guard.
 
Dichos cambios son:
 
    'defaults' => [
        'guard' => 'api',
        'passwords' => 'users',
    ],
 
    ...
 
    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],
 
Con esto se le informará al guard de la API que debe utilizar el driver de JWT, también se está definiendo el guard de la API como un default dentro de la configuración.
 
---
 
### Añadir rutas de autenticación básicas
 
Se agregaran algunas rutas de autenticación básicas dentro de "routes/api.php"
 
    Route::group([
 
        'middleware' => 'api',
        'prefix' => 'auth'
 
    ], function ($router) {
 
        Route::post('login', 'AuthController@login');
        Route::post('logout', 'AuthController@logout');
        Route::post('refresh', 'AuthController@refresh');
        Route::post('me', 'AuthController@me');
 
    });
 
---
 
### Crear el AuthController
 
Esto se puede hacer de 2 formas, manualmente o utilizando el comando:
 
    php artisan make:controller AuthController
 
Una vez creado el AuthController se deberá agregar lo siguiente:
 
    <?php
 
    namespace App\Http\Controllers;
 
    use Illuminate\Support\Facades\Auth;
    use App\Http\Controllers\Controller;
 
    class AuthController extends Controller
    {
        /**
        * Create a new AuthController instance.
        *
        * @return void
        */
        public function __construct()
        {
            $this->middleware('auth:api', ['except' => ['login']]);
        }
 
        /**
        * Get a JWT via given credentials.
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function login()
        {
            $credentials = request(['email', 'password']);
 
            if (! $token = auth()->attempt($credentials)) {
                return response()->json(['error' => 'Unauthorized'], 401);
            }
 
            return $this->respondWithToken($token);
        }
 
        /**
        * Get the authenticated User.
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function me()
        {
            return response()->json(auth()->user());
        }
 
        /**
        * Log the user out (Invalidate the token).
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function logout()
        {
            auth()->logout();
 
            return response()->json(['message' => 'Successfully logged out']);
        }
 
        /**
        * Refresh a token.
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function refresh()
        {
            return $this->respondWithToken(auth()->refresh());
        }
 
        /**
        * Get the token array structure.
        *
        * @param  string $token
        *
        * @return \Illuminate\Http\JsonResponse
        */
        protected function respondWithToken($token)
        {
            return response()->json([
                'access_token' => $token,
                'token_type' => 'bearer',
                'expires_in' => auth()->factory()->getTTL() * 60
            ]);
        }
    }
 
Una vez hecho lo anterior, se podrá probar el login con credenciales válidas, lo cual deberá mostrar un resultado parecido a:
 
    {
        "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ",
        "token_type": "bearer",
        "expires_in": 3600
    }
