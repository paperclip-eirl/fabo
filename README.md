# Fabo - PHP

Librería de acceso a la API de [fabo.dev](https://fabo.dev/) para PHP 8.0 o mayor.

## Instalación

La instalación se realiza vía [Composer](https://packagist.org/packages/paperclip/fabo):

```bash
composer require paperclip/fabo
```

## Uso

El constructor de la clase `Paperclip\Fabo\Facturador` requiere dos parámetros:

* `$token`, el token recibido cuando creas el acceso a la API de Fabo.
* `$url`, la URL para acceder a la API de Fabo, también recibida cuando creas el acceso.

Al instanciar un objecto `Paperclip\Fabo\Facturador`, puedes ejecutar un [comando de la API](https://docs.paperclip.com.pe/api-facturación/#comandos) como método del objeto:

* [`Facturador::hola(...$parámetros):array`](https://docs.paperclip.com.pe/api-facturación/documentación/comando-hola/): Comando para realizar pruebas de comunicación.
* [`Facturador::emitir(...$parámetros):array`](https://docs.paperclip.com.pe/api-facturación/documentación/comando-emitir/): Genera un nuevo comprobante, ya sea factura, boleta, o sus notas correspondientes.
* [`Facturador::baja(...$parámetros): array`](https://docs.paperclip.com.pe/api-facturación/documentación/comando-baja/): Solicita la baja (anulación) un comprobante.
* [`Facturador::correo(...$parámetros):array`](https://docs.paperclip.com.pe/api-facturación/documentación/comando-correo/): Envia el PDF y el XML por correo electrónico a los destinatarios especificados.
* [`Facturador::consultarRuc(...$parámetros):array`](https://docs.paperclip.com.pe/api-facturación/documentación/comando-consultarruc/): Obtiene información sobre un RUC, o un DNI con empresa.
* [`Facturador::consultarTicket(...$parámetros):array`](https://docs.paperclip.com.pe/api-facturación/documentación/comando-consultarticket/): Consulta el estado de un ticket de una transacción diferida de la SUNAT.

Cada método debes llamarlos con los parámetros requeridos por cada comando, usando parámetros con nombre.

## Excepciones

Cuando la API retorna un error, se genera varias excepciones según el tipo de error. Todas las excepciones extienden `\Exception`:

* **`Paperclip\Fabo\ExcepciónAutorización`**: Excepción lanzada cuando hubo un error en la fase de autorización, como un UUID o token inválido.
* **`Paperclip\Fabo\ExcepciónNegociación`**: Excepción lanzada cuando hubo un error en la fase de negociaciación con la API, como error en la cabecera HTML del formato a usar, etc.
* **`Paperclip\Fabo\ExcepciónParámetros`**: Excepción lanzada cuando hubo un error en los parámetros enviados a la API.
* **`Paperclip\Fabo\ExcepciónFatal`**: Excepción lanzada cuando hubo un error no previsto en la API o en esta librería.

## Ejemplo

Este código emite una factura electrónica:

```php
<?php

use Paperclip\Fabo\Facturador;

// Autocargador de Composer. Más información en https://getcomposer.org/doc/01-basic-usage.md#autoloading
require __DIR__ . '/vendor/autoload.php';

// Creamos el objecto facturador con el token y la URL proporcionada al crear el acceso
// a la API. Debes colocar el token y URL de tu cuenta para que funcione.
$facturador = new Facturador(
    '5cfccd27b2bb0f0d3fc860c7a1fb7231d9484c0e59195a5da55af9b7b2b7b703',
    'https://api-testing.fabo.dev/ebcf891b-9ec9-4409-a95f-cb8c3c803b17'
);

// Parámetros para la emisión de una factura
$parámetros = [

    // Estos datos son ficticios.
    'numero_doc' => 20123456786,
    'razon_social' => 'ABC CONSULTORES S.A.C',

    // f = Factura electrónica.
    'tipo' => 'f',
    'serie' => 'F001',
    'numero' => 21,
    'fecha' => '2021-01-26',
    'hora' => '07:23',

    // r = RUC
    'tipo_doc' => 'r',

    // PEN = Soles
    'moneda' => 'PEN',
    'total_gravado' => 76.48,
    'total_igv' => 13.77,
    'total_icbper' => 0.4,
    'items' => [
        [
			// NIU = Unidad genérica de bienes
            'unidad' => 'NIU',
            'cantidad' => 1,
            'codigo' => '98765',
            'descripcion' => 'ESCUDO DE VIBRANIUM',
            'precio_unitario' => 41,
            'valor_unitario' => 34.75,
            'valor_venta' => 34.75,
            'igv' => 6.25,

            // 10 = Gravado - operación onerosa
            'afectacion_igv' => 10,
        ],
        [
            'unidad' => 'NIU',
            'cantidad' => 1,
            'codigo' => '43210',
            'descripcion' => 'LAWGIVER MK II',
            'precio_unitario' => 49,
            'valor_unitario' => 41.53,
            'valor_venta' => 41.53,
            'igv' => 7.48,
            'afectacion_igv' => 10,
        ],
        [
            'unidad' => 'NIU',
            'cantidad' => 1,
            'codigo' => 'BP001',
            'descripcion' => 'BOLSA DE PLÁSTICO GRANDE',
            'precio_unitario' => 0.2,
            'valor_unitario' => 0.17,
            'igv' => 0.04,
            'afectacion_igv' => 10,
            'icbper' => 0.4,
        ]
    ],
];

try {
    // Emitimos el comprobante
    $resultado = $facturador->emitir(...$parámetros);
} catch (Exception $e) {
    // Si falla, generará una excepción. Obtenemos el resultado
    echo get_class($e) . ': ' . $e->getMessage() . PHP_EOL . PHP_EOL;

    // $resultado tendrá información del error.
    $resultado = $facturador->obtenerRespuesta();

    var_dump($resultado);
    exit;
}

// En este punto, el comprobante fue recibido satisfactoriamente por la API.
// El resultado del comando 'emitir' estará en la variable $resultado.
//
// Puedes ver más información sobre las variables del resultado en la URL
// https://docs.paperclip.com.pe/api-facturación/documentación/comando-emitir/#variables-de-retorno
//
echo "Comprobante emitido." . PHP_EOL . PHP_EOL;
var_dump($resultado);
```

Al ejecutar este código con un `$token` y `$url` válido, mostrará algo similiar a:

```
Comprobante emitido.

array(7) {
  ["estado"]=>
  string(2) "ok"
  ["valor_resumen"]=>
  string(28) "AKXZBYFhI4idLnFLAnUvSPra95o="
  ["codigo_descarga"]=>
  string(40) "4c05b054213fb1663588a2281de81219cc8bbeae"
  ["codigo_documento"]=>
  string(22) "20123456781-01-F001-21"
  ["sunat_respuesta"]=>
  string(1) "0"
  ["sunat_descripcion"]=>
  string(43) "La Factura numero F001-21, ha sido aceptada"
  ["sunat_observacion"]=>
  array(0) {
  }
}
```
