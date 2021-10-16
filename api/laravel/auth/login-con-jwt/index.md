# Login con JWT

En esta documentación se detallará a un nivel simple las funciones del login y sus requerimientos.

---

Para poder realizar este proceso se necesitara.

* Laravel (De preferencia versiones iguales o superiores a la 5.4, aunque se pueden utilizar versiones anteriores).

* Composer.

* Visual Studio Code

* Saber utilizar la consola de comandos en Visual Studio Code

---

## Instalacion del Auth por Composer

Para poder instalar la ultima version del auth de JWT se debe ejecutar el siguiente codigo.

    composer require tymon/jwt-auth

## Añadir proveedor de servicio

Una vez se ha realizado el paso anterior se debera agregar un proveedor de servicios al array "providers" que se encuentra dentro de la ruta "config/app.php".

    'providers' => [

        ...

        Tymon\JWTAuth\Providers\LaravelServiceProvider::class,
    ]

Cabe destacar que este proceso solo se debe realizar en proyectos que utilicen Laravel 5.4 o inferior.

---

## Publicar la configuracion

Una vez completados los pasos anteriores se debe publicar el paquete de configuracion, para poder hacer esto se debera utilizar el siguiente comando.

    php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"

Esto creara la ruta "config/jwt.php" que servira para configurar los aspectos basicos del paquete.

---

### Generar clave secreta

Se incluye un comando auxiliar que tiene la utilidad de crear una clave secreta para el usuario