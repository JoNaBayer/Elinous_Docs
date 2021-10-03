# Crud controlador de generación de datos

Para poder utilizar este generador se requerirá:

	-Una base de datos conectada a Laravel que siga las recomendaciones de Laravel para los nombres de tablas y columnas.

	-Las relaciones de las tablas deben estar bien definidas para que el método detecte como relacionar los modelos.

	-Laravel 5.5 o superior.

	-Motor de Base de Datos MySQL.

	-Engine INNODB.

Nota: "Los archivos generados aparecerán en la ruta: storage/app/generated_data"

## Primer paso

Una vez que se tengan los requerimientos anteriores se debera utilizar el siguiente codigo como un inicio para el controlador

	<?php

	namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;
	use Illuminate\Support\Facades\Storage;
	use Illuminate\Support\Facades\DB;

	define('DATABASE_SCHEMA', env('DB_DATABASE', 'default_aca'));
	class DataGeneratorController extends Controller
	{
  		function generate_models()
  		{
    			$tablas = armar_tablas();
    			//crear_controladores($tablas);
    			return crear_modelos($tablas);
  		}
	}

Con esto se determinara la base de datos a utilizar por el controlador.

## Crear controladores

