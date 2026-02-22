---
title: "Cómo implementar el patrón Action en Laravel"
date: "2026-02-22"
author: "Edu Lázaro"
description: "Guía práctica del paquete Laractions para organizar la lógica de negocio en clases Action reutilizables, con soporte para colas, modelos, trazabilidad y testing."
tags: ["laravel", "arquitectura", "paquetes", "laractions", "php"]
---

Si llevas tiempo con Laravel, seguro que te has encontrado con controladores que crecen sin control. Validación, lógica de negocio, notificaciones, interacción con servicios externos... todo acaba en el mismo método `store()` de 200 líneas. El patrón Action lleva años siendo una solución popular en la comunidad, y [Laractions](https://github.com/edulazaro/laractions) lo implementa con una arquitectura completa para Laravel.

## ¿Qué es una Action?

Una Action es una clase con una única responsabilidad: ejecutar una operación de negocio concreta. En vez de meter toda la lógica en un controlador, la encapsulas en clases independientes y reutilizables.

```php
// ❌ Controlador inflado
class OrderController extends Controller
{
    public function store(Request $request)
    {
        // Validar...
        // Calcular impuestos...
        // Crear pedido...
        // Descontar stock...
        // Enviar email...
        // Notificar al almacén...
        // 200 líneas después...
    }
}

// ✅ Controlador limpio con Actions
class OrderController extends Controller
{
    public function store(Request $request)
    {
        $order = CreateOrderAction::create()->run(
            $request->user(),
            $request->validated()
        );

        return redirect()->route('orders.show', $order);
    }
}
```

La ventaja principal es que `CreateOrderAction` se puede reutilizar desde un controlador, un comando Artisan, un Job, un test o cualquier otro punto de entrada.

## Instalación

```bash
composer require edulazaro/laractions
```

El paquete se auto-registra con Laravel. No necesitas configuración adicional.

## Crear tu primera Action

```bash
php artisan make:action SendWelcomeEmailAction
```

Esto genera una clase en `app/Actions/` que extiende la clase base `Action`. Toda la lógica va dentro del método `handle()`:

```php
namespace App\Actions;

use EduLazaro\Laractions\Action;

class SendWelcomeEmailAction extends Action
{
    public function handle(string $email, string $nombre)
    {
        Mail::to($email)->send(new WelcomeMail($nombre));
    }
}
```

Para ejecutarla, usas `create()` para resolver la instancia a través del Service Container y `run()` para lanzarla. Los parámetros de `run()` se pasan directamente al método `handle()`:

```php
SendWelcomeEmailAction::create()->run('usuario@ejemplo.com', 'Carlos');
```

## Inyección de dependencias

Como `create()` resuelve la Action a través del Service Container, las dependencias del constructor se inyectan automáticamente:

```php
class CreateInvoiceAction extends Action
{
    protected TaxCalculator $calculator;

    public function __construct(TaxCalculator $calculator)
    {
        $this->calculator = $calculator;
    }

    public function handle(User $user, array $items): Invoice
    {
        $subtotal = collect($items)->sum('price');
        $tax = $this->calculator->calculate($subtotal);

        return Invoice::create([
            'user_id' => $user->id,
            'subtotal' => $subtotal,
            'tax' => $tax,
            'total' => $subtotal + $tax,
        ]);
    }
}
```

La ejecución es igual de simple:

```php
$invoice = CreateInvoiceAction::create()->run($user, $items);
```

## Parámetros dinámicos con `with()`

Cuando tienes muchos parámetros, puedes usar `with()` para pasarlos como atributos en vez de argumentos posicionales:

```php
SendWelcomeEmailAction::create()->with([
    'email' => 'usuario@ejemplo.com',
    'nombre' => 'Carlos',
])->run();
```

Dentro de la Action, accedes a ellos como propiedades: `$this->email`, `$this->nombre`.

## Actions vinculadas a modelos

Esta es una de las funcionalidades más potentes de Laractions. Puedes vincular Actions directamente a un modelo Eloquent, y el modelo se inyecta automáticamente en la Action.

### Generar una Action para un modelo

```bash
php artisan make:action SendEmailAction --model=User
```

Esto crea la Action dentro de `app/Actions/User/` con una propiedad tipada del modelo. Cuando se ejecuta desde una instancia del modelo, la propiedad se rellena automáticamente:

```php
namespace App\Actions\User;

use EduLazaro\Laractions\Action;
use App\Models\User;

class SendEmail extends Action
{
    protected User $user; // Se inyecta automáticamente

    public function handle(string $subject, string $message)
    {
        Mail::to($this->user->email)->send(
            new GenericMail($subject, $message)
        );
    }
}
```

### Registrar Actions en el modelo

Añade el trait `HasActions` y define las Actions disponibles en el array `$actions`:

```php
use EduLazaro\Laractions\Concerns\HasActions;

class User extends Model
{
    use HasActions;

    protected array $actions = [
        'send_email' => \App\Actions\User\SendEmail::class,
        'suspend' => \App\Actions\User\Suspend::class,
    ];
}
```

Ahora puedes invocarlas desde cualquier instancia del modelo:

```php
$user = User::find(1);

// Por nombre registrado
$user->action('send_email')->run('Bienvenido', 'Hola, bienvenido a la plataforma');

// O directamente por clase (sin necesidad de registrar en $actions)
$user->action(SendEmail::class)->run('Bienvenido', 'Hola, bienvenido a la plataforma');
```

En ambos casos, `$this->user` dentro de la Action apuntará a la instancia `$user` desde la que se invocó.

## Ejecución asíncrona con colas

Laractions permite despachar Actions como Jobs sin necesidad de crear una clase Job separada. Usa `dispatch()` en vez de `run()`:

```php
SendWelcomeEmailAction::create()
    ->queue('emails')
    ->delay(10)
    ->retry(3)
    ->dispatch('usuario@ejemplo.com', 'Carlos');
```

También puedes definir la configuración de cola como propiedades de la clase:

```php
class SendWelcomeEmailAction extends Action
{
    protected int $tries = 3;
    protected ?int $delay = 30;
    protected ?string $queue = 'emails';

    public function handle(string $email, string $nombre)
    {
        Mail::to($email)->send(new WelcomeMail($nombre));
    }
}
```

Laractions genera el Job automáticamente — tú solo defines la lógica en `handle()`.

## Actores y trazabilidad

Para auditoría, Laractions permite registrar quién ejecutó cada Action y sobre qué modelo.

### Configurar actores

Añade el trait `IsActor` al modelo que realiza acciones (típicamente `User`):

```php
use EduLazaro\Laractions\Concerns\IsActor;

class User extends Model
{
    use IsActor;
}
```

### Ejecutar con trazabilidad

Con `IsActor` puedes usar el método `act()` para ejecutar Actions como ese actor, `on()` para indicar sobre qué modelo actúa, y `trace()` para activar el registro:

```php
$admin = User::find(1);
$pedido = Order::find(42);

$admin->act(CancelOrderAction::class)
    ->on($pedido)
    ->trace()
    ->run();
```

Esto registra que `$admin` ejecutó `CancelOrderAction` sobre `$pedido`.

También puedes activar la trazabilidad en Actions standalone:

```php
SendEmailAction::create()
    ->actor($user)
    ->on($targetModel)
    ->trace()
    ->run($params);
```

## Logging

Activa el logging para cualquier Action:

```php
SendWelcomeEmailAction::create()
    ->enableLogging()
    ->run('usuario@ejemplo.com', 'Carlos');
```

Los logs se escriben en los canales de log configurados en Laravel.

## Mocking en tests

Laractions facilita el testing permitiendo hacer mock de Actions vinculadas a modelos:

```php
$user->mockAction(SendEmailAction::class, new class {
    public function run()
    {
        return 'Mocked!';
    }
});

// La Action real no se ejecuta
$result = $user->action(SendEmailAction::class)->run();
// $result === 'Mocked!'
```

## Listar Actions registradas

Para ver todas las Actions disponibles en tu aplicación:

```bash
php artisan list:actions
```

## ¿Cuándo usar Actions?

Las Actions encajan bien cuando:

- **El controlador crece demasiado**. Si un método tiene más de 20-30 líneas de lógica de negocio, es candidato a extraer en una Action.
- **Reutilizas operaciones**. La misma lógica se necesita en un controlador web, un endpoint API y un comando Artisan.
- **Quieres tests unitarios limpios**. Es más fácil testear `CreateInvoiceAction` aislada que un controlador completo.
- **Necesitas auditoría**. La trazabilidad registra quién hizo qué y sobre qué modelo.

Y no las necesitas cuando la lógica es trivial (un simple CRUD sin reglas de negocio) o cuando un Event + Listener ya cubre el caso.

## Conclusión

Laractions implementa el patrón Action con una API fluida que incluye generación por Artisan, vinculación a modelos con inyección automática, colas, trazabilidad, logging y mocking. Si te frustran los controladores gordos y la lógica de negocio dispersa, dale una oportunidad.

- **Repositorio**: [github.com/edulazaro/laractions](https://github.com/edulazaro/laractions)

Desde **Laravel Spain** seguiremos compartiendo paquetes y patrones de la comunidad. Si conoces otros paquetes interesantes, pásate por nuestro [Discord](https://discord.gg/laravelspain).
