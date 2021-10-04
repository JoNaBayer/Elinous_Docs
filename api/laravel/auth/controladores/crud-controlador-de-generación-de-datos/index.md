# Controlador de generación de datos

En esta documentación se detallará a un nivel simple las funciones del controlador y sus requerimientos.

---

Para poder utilizar este generador se requerirá:

* Una base de datos conectada a Laravel que siga las recomendaciones de Laravel para los nombres de tablas y columnas.

* Las relaciones de las tablas deben estar bien definidas para que el método detecte como relacionar los modelos.

* Laravel 5.5 o superior.

* Motor de Base de Datos MySQL.

* Engine INNODB.

Nota: "Los archivos generados aparecerán en la ruta: storage/app/generated_data"

---

## Primer paso

Una vez que se tengan los requerimientos anteriores se debera utilizar el siguiente codigo como un inicio para el controlador

Con esto se determinará la base de datos a utilizar por el controlador.

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

---

## Función "crear_controladores"

Mediante esta funcion se crearán los distintos CRUD que necesitara el modelo, como puede ser el mostrar los datos de cierta tabla mediante el uso de IDs o "Claves Primarias".

Tambien se encuentran otros metodos CRUD como eliminar, actualizar e insertar.

    function crear_controladores($tablas)
    {
    $controladores = '';
    $routes = '';
    foreach ($tablas as $key => $tabla) {
        $table_name = $key;
        $model_name = model_name($table_name);
        $plural_model = plural($model_name);
        $controlador = "<?php
        \nnamespace App\Http\Controllers;
        \nuse Illuminate\Http\Request;
        \nuse App\\$model_name;
        \nclass {$model_name}Controller extends Controller
        {
        /**
        * GET all
        */
        public function index()
        {
            return response()->json(['message' => '$plural_model obtenidos', 'data' => $model_name::all()], 200);
        }\n
        /**
        * POST
        */
        public function store(Request \$request)
        {
            \$data = \$request->all();" . ($tabla['auth'] ? "\n        \$data['password'] = Hash::make(\$data['password']);" : "") . "
            \${$table_name} = $model_name::create(\$data);
            return response()->json(['message' => '$model_name creado', 'data' => \${$table_name}], 200);
        }\n
        /**
        * GET 1
        */
        public function show(\$id)
        {
            \${$table_name} = $model_name::findOrFail(\$id);
            return response()->json(['message' => '$model_name obtenido', 'data' => \${$table_name}], 200);
        }\n
        /**
        * PUT
        */
        public function update(Request \$request, \$id)
        {
            \${$table_name} = $model_name::findOrFail(\$id);
            \${$table_name}->update(\$request->all());
            return response()->json(['message' => '$model_name editado', 'data' => \${$table_name}], 200);
        }\n
        /**
        * DELETE
        */
        public function destroy(\$id)
        {
            \${$table_name} = $model_name::findOrFail(\$id);
            \${$table_name}->delete();
            return response()->json(['message' => '$model_name eliminado'], 200);
        }\n}";
        $filename = "{$model_name}Controller.php";
        $path = 'generated_data/app/Http/Controllers/' . $filename;
        Storage::disk('local')->put($path, $controlador);
        $controladores .= $controlador;
        $routes .= "Route::resource('$table_name', '{$model_name}Controller', [\n    'except' => ['edit', 'create']\n]);\n\n";
    }
    $path = 'generated_data/app/Http/Controllers/routes.txt';
    Storage::disk('local')->put($path, $routes);
    return $controladores;
    }

---

## Función "crear_modelos"

La siguiente función sirve para crear modelos de datos, tomando las tablas y las relaciones como los atributos a utilizar.

    function crear_modelos($tablas)
    {
    $crear_relaciones = function ($tabla) {
        return crear_relaciones($tabla);
    };
    $fillable = function ($tabla) {
        return fillable($tabla);
    };
    $if = function ($condition, $true, $false) {
        return $condition ? $true : $false;
    };
    $modelos = '';
    foreach ($tablas as $key => $tabla) {
        $table_name = $key;
        $model_name = model_name($table_name);
        $modelo = <<<EOT
            <?php
            
            namespace App;
            
            use Illuminate\Database\Eloquent\Model;
            {$if($tabla['auth'], "use Illuminate\Foundation\Auth\User as Authenticatable;\n", "")}
            class $model_name extends {$if($tabla['auth'], "Authenticatable", "Model")}
            {
                protected \$table = '$table_name';
                
                protected \$fillable = [{$fillable($tabla)}
                ];
            {$if(!$tabla['tiene_pk'], "\n    protected \$primaryKey = null;\n\n    public \$incrementing = false;", '')}
            {$if($tabla['auth'], "\n    protected \$hidden = ['password'];", '')}
            {$if(!$tabla['timestamps'], "\n    public \$timestamps = false;\n", '')}
            {$crear_relaciones($tabla)}
            }
            
            EOT;

        $filename = "$model_name.php";
        $path = 'generated_data/app/' . $filename;
        Storage::disk('local')->put($path, $modelo);
        $modelos .= $modelo;
    }
    return $modelos;
    }

---

## Función "fillable"

La utilidad de esta función es principalmente el llenar las columnas de las tablas con los atributos correspondientes dentro de la base de datos, un ejemplo de esto podria ser el agregar todos los atributos que se encuentren en la columna "nombre" de una tabla X.

    function fillable($tabla)
    {
    $not_in = [
        "id",
        "created_at",
        "updated_at"
    ];
    $fillable = "";
    foreach ($tabla['columnas'] as $col) {
        if (!in_array($col["COLUMN_NAME"], $not_in)) {
        $fillable .= <<<EOT
                \n        '{$col["COLUMN_NAME"]}',
                EOT;
        }
    }
    return $fillable;
    }

---

## Función "crear_relaciones"

Esta funcion crear las relaciones que tendran las tablas, revisando si las relaciones dentro de la base de datos es de "Uno a muchos", "Muchos a muchos" o "Uno a uno".

Tambien revisara si es que la relacion esta vinculada a una tabla intermedia, igualmente traspasara las claves foraneas necesarias para las relaciones.

    function crear_relaciones($tabla)
    {
    $relaciones = "";
    foreach (($tabla['relaciones'] + $tabla['relaciones_inversas']) as $relacion) {
        $referenced_table = $relacion['REFERENCED_TABLE_NAME'];
        $model_name = model_name($referenced_table);
        $tipo = $relacion['TIPO'];
        $nombre_relacion = ($tipo == 'hasMany') ? pluralizar($referenced_table) : $referenced_table;
        $relaciones .= <<<EOT
                public function $nombre_relacion()
                {
                    return \$this->$tipo('App\\$model_name');
                }\n

            EOT;
    }
    foreach ($tabla['pivots'] as $relacion) {
        $referenced_table = $relacion['REFERENCED_TABLE_NAME'];
        $pivot = $relacion['PIVOT'];
        $model_name = model_name($referenced_table);
        $plural = pluralizar($referenced_table);
        $relaciones .= <<<EOT
                public function $plural()
                {
                    return \$this->belongsToMany('App\\$model_name', '$pivot');
                }

            EOT;
    }
    return $relaciones;
    }