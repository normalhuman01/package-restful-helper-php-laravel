# Helper de RESTful para Laravel

## Instalación

A través del Composer

``` bash
$ composer require mrjmpl3/laravel-restful-helper
```

## Uso

Este paquete realiza consultas basadas en la solicitud, similar a GraphQL.

### Solicitudes

- **Filtrar datos:** /producto?columna=valor&columna2=valor2
- **Ordenar datos:** /producto?sort=-columna1,columna2
    - Con el prefijo negativo = desc
    - Sin el prefijo negativo = asc
- **Seleccionar campos:** /producto?fields=columna1,columna2,columna3,columna4
- **Paginar y por página:** /producto?paginate=true&per_page=5
- **Incrustar:** /producto?embed=funcionRelacion

### Código

#### A Colección

```
// Crea una instancia simple del modelo donde deseas aplicar las consultas
$modelo = new Producto();
$ayudanteRest = new ApiRestHelper($modelo);
          
// El método 'toCollection' devuelve una colección con todos los datos filtrados
$respuesta = $ayudanteRest->toCollection();
```
      
#### A Modelo

```
// Crea una instancia simple del modelo donde deseas aplicar las consultas
$modelo = new Producto();           
$ayudanteRest = new ApiRestHelper($modelo);
                
// El método 'toModel' devuelve un modelo con todos los datos filtrados
$respuesta = $ayudanteRest->toModel();
```
      
#### De Constructor a Colección

```
// ¡Importante! No cierres la consulta con get() o paginate()
$consulta = Producto::where('estado', '=', 1);
$ayudanteRest = new ApiRestHelper($consulta);
          
// El método 'toCollection' devuelve una colección con todos los datos filtrados
$respuesta = $ayudanteRest->toCollection();
```
      
#### Relaciones

- En el modelo, agrega un array como en el siguiente ejemplo:

    ```
    public $apiAcceptRelations = [
        'post'
    ];
    ```
    Donde 'post' es el nombre de la función de relación.
                
- En los Recursos de la API, utiliza la función incrustar:

    ```
    public function toArray($solicitud) {
        $incrustar = (new ApiRestHelper)->getEmbed();
                
        return [
          'id' => $this->id,
          'nombre' => $this->nombre,
          $this->mergeWhen(array_key_exists('post', $incrustar), [
              'post' => $this->getPostResource($incrustar),
          ]),
          'creado_en' => $this->creado_en,
          'actualizado_en' => $this->actualizado_en,
        ];
    }
            
    private function getPostResource($solicitudIncrustar) {
      $recursoPost = NULL;
                
      if (array_key_exists('local', $incrustar)) {
          $relacionPost = $this->local();                    
          $camposDesdeIncrustar = (new ApiRestHelper($relacionPost->getModel()))->getEmbedField('post');
                    
          if(!empty($camposDesdeIncrustar)) {
              $recursoPost = new PostResource($relacionPost->select($camposDesdeIncrustar)->first());
          } else {
              $recursoPost = new PostResource($relacionPost->first());
          }
      }
            
      return $recursoPost;
    }
    ```
#### Transformadores

- En el modelo, agrega un array como en el siguiente ejemplo:
	
```
public $apiTransforms = [
    'id' => 'codigo'
];
```
		
Donde 'id' es el nombre de la columna de la base de datos y 'codigo' es el nombre de la columna para la respuesta.
	
- En los Recursos de la API, utiliza el array $apiTransforms
	
```
$ayudanteApi = new ApiRestHelper($this);

return [
    $ayudanteApi->getKeyTransformed('id') => $this->id,
    'nombre' => $this->nombre,
    'creado_en' => $this->creado_en,
    'actualizado_en' => $this->actualizado_en,
];
```

- Para utilizar campos en Recursos de API, puedes combinarlos con campos de transformadores
    
```
$ayudanteApi = new ApiRestHelper($this);

return [
    $this->mergeWhen($ayudanteApi->existInFields('id') && !is_null($this->id), [
        $this->transforms['id'] => $this->id
    ]),
    $this->mergeWhen($ayudanteApi->existInFields('nombre') && !is_null($this->nombre), [
        'nombre' => $this->nombre
    ]),
]
```

#### Excluir Campos en el Filtro

- En el modelo, agrega un array como en el siguiente ejemplo:
	
```
public $apiExcludeFilter = [
    'id'
];
```
		
Donde 'id' es el nombre de la columna de la base de datos a excluir.

## Registro de cambios

Consulta [CHANGELOG](CHANGELOG.md) para obtener más información sobre lo que ha cambiado recientemente.

## Seguridad

Si descubres problemas relacionados con la seguridad, envía un correo electrónico a jmpl3.soporte@gmail.com en lugar de usar el seguimiento de problemas.

## Licencia

Licencia MIT (MIT). Consulta [Archivo de licencia](LICENSE.md) para obtener más información.
