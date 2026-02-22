---
title: "Laractions: cómo implementar el patrón Action en Laravel"
date: "2026-02-22"
author: "Laravel Spain"
description: "Guía práctica del paquete Laractions para organizar la lógica de negocio en clases Action reutilizables, con soporte para colas, modelos, trazabilidad y testing."
tags: ["laravel", "arquitectura", "paquetes", "laractions", "php"]
---

Si llevas tiempo con Laravel, seguro que te has encontrado con controladores que crecen sin control. Validación, lógica de negocio, notificaciones, interacción con servicios externos... todo acaba en el mismo método `store()` de 200 líneas. El patrón Action lleva años siendo una solución popular en la comunidad, y [Laractions](https://github.com/edulazaro/laractions) lo lleva al siguiente nivel con una arquitectura completa.

## ¿Qué es una Action?

Una Action es una clase con una única responsabilidad: ejecutar una operación de negocio concreta. En vez de tener un controlador que hace de todo, extraes cada operación a su propia clase.

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
    public function store(Request $request, CreateOrderAction $action)
    {
        $order = $action->handle($request->user(), $request->validated());
        return redirect()->route('orders.show', $order);
    }
}
```

La ventaja principal es que `CreateOrderAction` se puede reutilizar desde un controlador, un comando Artisan, un Job, un test o cualquier otro punto de entrada.

## Instalación de Laractions

```bash
composer require edulazaro/laractions
```

El paquete se auto-registra con Laravel. No necesitas configuración adicional.

## Crear tu primera Action

```bash
php artisan make:action SendWelcomeEmailAction
```

Esto genera una clase en `app/Actions/` que extiende la clase base `Action`:

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

Para ejecutarla, usas el patrón `create()->run()`:

```php
SendWelcomeEmailAction::create()->run('usuario@ejemplo.com', 'Carlos');
```

## Inyección de dependencias

Las Actions se resuelven a través del Service Container de Laravel, así que puedes inyectar dependencias en el constructor:

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

## Actions vinculadas a modelos

Esta es una de las funcionalidades más potentes de Laractions. Puedes vincular Actions directamente a un modelo Eloquent.

### Generar una Action para un modelo

```bash
php artisan make:action SendEmailAction --model=User
```

Esto crea la Action dentro de `app/Actions/User/` con el modelo ya inyectado:

```php
namespace App\Actions\User;

use EduLazaro\Laractions\Action;
use App\Models\User;

class SendEmail extends Action
{
    protected User $user;

    public function handle(string $subject, string $message)
    {
        Mail::to($this->user->email)->send(
            new GenericMail($subject, $message)
        );
    }
}
```

### Registrar Actions en el modelo

Añade el trait `HasActions` y define las Actions disponibles:

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

// O directamente por clase
$user->action(SendEmail::class)->run('Bienvenido', 'Hola, bienvenido a la plataforma');
```

## Parámetros dinámicos con `with()`

Puedes pasar parámetros como atributos en vez de argumentos de `run()`:

```php
SendWelcomeEmailAction::create()->with([
    'email' => 'usuario@ejemplo.com',
    'nombre' => 'Carlos',
])->run();
```

Dentro de la Action, accedes a ellos como propiedades: `$this->email`, `$this->nombre`.

## Ejecución asíncrona con colas

Laractions permite despachar Actions como Jobs de forma transparente:

```php
SendWelcomeEmailAction::create()
    ->queue('emails')
    ->delay(10)
    ->retry(3)
    ->dispatch('usuario@ejemplo.com', 'Carlos');
```

También puedes definir la configuración de cola directamente en la clase:

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

No necesitas crear un Job separado — Laractions genera la clase Job automáticamente.

## Actores y trazabilidad

Para auditoría, Laractions permite rastrear quién ejecutó cada Action y sobre qué modelo.

### Configurar actores

Añade el trait `IsActor` al modelo que ejecuta acciones:

```php
use EduLazaro\Laractions\Concerns\IsActor;

class User extends Model
{
    use IsActor;
}
```

### Ejecutar con trazabilidad

```php
$admin = User::find(1);
$pedido = Order::find(42);

$admin->act(CancelOrderAction::class)
    ->on($pedido)
    ->trace()
    ->run();
```

Esto registra que `$admin` ejecutó `CancelOrderAction` sobre `$pedido`, útil para logs de auditoría.

## Mocking en tests

Laractions facilita el testing permitiendo hacer mock de las Actions:

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
- **Quieres tests unitarios limpios**. Es más fácil testear `CreateInvoiceAction` que testear un controlador completo con HTTP.
- **Necesitas auditoría**. La trazabilidad de Laractions registra quién hizo qué y sobre qué modelo.

Y no las necesitas cuando la lógica es trivial (un simple CRUD sin reglas de negocio) o cuando un Event + Listener ya cubre el caso.

## Conclusión

El patrón Action no es nuevo, pero Laractions lo eleva con una implementación completa que incluye generación por Artisan, vinculación a modelos, colas, trazabilidad y mocking. Si te frustran los controladores gordos y la lógica de negocio dispersa, dale una oportunidad.

- **Repositorio**: [github.com/edulazaro/laractions](https://github.com/edulazaro/laractions)
- **Instalación**: `composer require edulazaro/laractions`

Desde **Laravel Spain** seguiremos compartiendo paquetes y patrones de la comunidad. Si conoces otros paquetes interesantes, pásate por nuestro [Discord](https://discord.gg/laravelspain).
