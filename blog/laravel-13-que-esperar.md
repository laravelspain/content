---
title: "Qué esperar de Laravel 13: todo lo que sabemos"
date: "2026-02-15"
author: "Laravel Spain"
description: "Repaso completo a las novedades confirmadas de Laravel 13: requisitos, nuevas funcionalidades, mejoras internas y todo lo que los desarrolladores necesitan saber."
tags: ["laravel", "novedades", "php", "laravel-13"]
---

## Laravel 13 está en camino

Siguiendo el ciclo anual de versiones mayores, Laravel 13 está previsto para el **primer trimestre de 2026**. Como ya pasó con Laravel 12, esta versión prioriza la estabilidad y la modernización interna sobre cambios revolucionarios. Pero eso no significa que no traiga cosas interesantes.

Vamos a repasar todo lo que se sabe hasta ahora, basándonos en los cambios que ya están en la rama de desarrollo.

## Requisitos: PHP 8.3 obligatorio

El cambio más importante a nivel de infraestructura es que **Laravel 13 elimina el soporte para PHP 8.2**. El requisito mínimo pasa a ser PHP 8.3, con compatibilidad también para PHP 8.4.

Esto permite al equipo de Laravel eliminar polyfills obsoletos y código de compatibilidad hacia atrás, dejando un framework más limpio y aprovechando las mejoras de rendimiento de PHP 8.3. En benchmarks, las aplicaciones Laravel sobre PHP 8.3 manejan alrededor de 445 peticiones por segundo en endpoints API, una mejora de entre un 5% y un 10% respecto a PHP 8.2 gracias a las mejoras en JIT y la reducción del consumo de memoria en procesos de larga duración.

## Compatibilidad con Symfony 7.4 y 8.0

Laravel 13 actualiza sus dependencias para ser compatible con **Symfony 7.4 y 8.0**, afectando a componentes clave como HTTP, Console y Routing. También se han resuelto deprecaciones de Symfony Console para garantizar compatibilidad futura.

## Nuevo método Cache::touch()

Una de las novedades más prácticas. El nuevo método `Cache::touch()` (y `Store::touch()`) permite **extender el TTL de un elemento en caché sin necesidad de recuperar su valor**.

```php
// Extiende el TTL del elemento 'user:123' sin leer su contenido
Cache::touch('user:123', now()->addMinutes(30));
```

Esto es especialmente útil en aplicaciones de alto tráfico donde quieres mantener un elemento en caché "vivo" sin el coste de deserializarlo y volver a guardarlo.

## Prioridad en rutas de subdominio

Las rutas vinculadas a subdominios específicos ahora se **registran antes que las rutas sin dominio**. Esto garantiza que las rutas de subdominio tengan prioridad durante el matching de la petición, evitando conflictos que antes podían ser difíciles de depurar.

```php
// Esta ruta ahora se registra primero internamente
Route::domain('api.ejemplo.com')->group(function () {
    Route::get('/users', [UserController::class, 'index']);
});

// Esta se registra después
Route::get('/users', [WebUserController::class, 'index']);
```

## Restricciones en el boot de modelos Eloquent

Laravel 13 **impide crear nuevas instancias de modelos Eloquent durante el método `boot()`**. Este cambio previene efectos secundarios inesperados y cascadas de inicialización que podían causar bugs difíciles de rastrear.

```php
class User extends Model
{
    protected static function boot()
    {
        parent::boot();

        // Esto ahora lanzará una excepción en Laravel 13
        // $settings = new Setting();
    }
}
```

## Mejoras en el cliente HTTP

Dos cambios relevantes en el cliente HTTP:

- **Pool con concurrencia por defecto**. `PendingRequest::pool()` ahora usa un valor de concurrencia de 2 por defecto, en vez de `null`. Esto evita el problema de que los desarrolladores asuman que las peticiones en pool se ejecutan en paralelo cuando en realidad, sin concurrencia definida, se ejecutaban en serie.

- **Request::get() alineado con Symfony**. El comportamiento del método se ha ajustado para mayor consistencia con los componentes de Symfony.

## Mejoras en MySQL: DELETE con JOIN

Las consultas `DELETE...JOIN` en MySQL ahora compilan correctamente las cláusulas `ORDER BY` y `LIMIT`. Antes, estas cláusulas se omitían silenciosamente, lo que podía causar que se eliminaran más registros de los esperados.

## Mejoras en el sistema de colas

El evento `JobAttempted` ahora expone el **objeto Throwable completo** en vez de un simple flag booleano. Esto da a los listeners mucho más contexto sobre los fallos.

```php
Event::listen(JobAttempted::class, function ($event) {
    if ($event->exception) {
        // Ahora tienes acceso al objeto completo de la excepción
        Log::error($event->exception->getMessage());
    }
});
```

## Nombres de tablas pivot polimórficas en plural

Los nombres de tablas pivot generados automáticamente para relaciones polimórficas ahora usan la **convención en plural**, alineándose con lo que la documentación siempre sugirió.

## Mejoras en testing

Las factories de `Str` ahora se **resetean entre cada test case**, lo que previene fugas de estado entre tests que podían causar fallos intermitentes difíciles de diagnosticar.

## Driver de base de datos para Reverb

Laravel 13 incluye un **driver de base de datos para Reverb** que permite escalado horizontal sin necesidad de Redis. Una opción interesante para aplicaciones que quieren WebSockets sin añadir Redis a su stack.

## Control granular de reintentos en colas

Se añade soporte para propiedades como `maxExceptions` con mayor granularidad, permitiendo un control más fino sobre cuándo un job debe dejar de reintentarse.

## Otras mejoras menores

- **Manager driver closures**. Los callbacks de drivers personalizados reciben instancias consistentes del manager
- **Soporte para prefijos con guiones**. Mejor manejo de prefijos con guiones en varias áreas del framework
- **Notificaciones**. Mejoras en la capitalización de asuntos de email para verificación y reset de contraseña
- **Limpieza de CI**. Mejoras en la fiabilidad del entorno de integración continua
- **Corrección de scopes anidados**. Se eliminan edge cases problemáticos con scopes globales en queries anidadas
- **Metadatos de Composer sincronizados** a `^13.0` para prevenir desajustes de dependencias

## Soporte y ciclo de vida

| Versión    | Corrección de bugs | Parches de seguridad |
|------------|-------------------|---------------------|
| Laravel 12 | Hasta agosto 2026 | Hasta febrero 2027  |
| Laravel 13 | Hasta Q3 2027     | Hasta Q1 2028       |

## Cómo probarlo ahora

Si quieres experimentar con Laravel 13 antes de su lanzamiento oficial:

```bash
# Con el instalador de Laravel
laravel new mi-app --dev

# Con Composer
composer create-project --prefer-dist laravel/laravel mi-app dev-master
```

## Conclusión

Laravel 13 sigue la línea de consolidación que vimos en Laravel 12. No es una revolución, sino una evolución enfocada en limpiar el framework, modernizar sus cimientos y pulir la experiencia del desarrollador. El salto a PHP 8.3 como mínimo, las mejoras en caché, routing y el cliente HTTP, y las correcciones de edge cases hacen que valga la pena la actualización.

Desde **Laravel Spain** seguiremos informando a medida que se confirmen más cambios. Si quieres compartir tu experiencia probando la versión de desarrollo, pásate por nuestro [Discord](https://discord.gg/laravelspain).
