---
title: "Novedades de Laravel 12: Todo lo que necesitas saber"
date: "2026-02-15"
author: "Laravel Spain"
description: "Un repaso completo a las principales novedades, mejoras de rendimiento y nuevas herramientas que trae Laravel 12."
tags: ["laravel", "novedades", "php"]
---

## Laravel 12 ya está aquí

La nueva versión mayor del framework PHP más popular del mundo ha llegado, y viene cargada de novedades que van a mejorar tu día a día como desarrollador.

## Nuevo starter kit con React, Vue e Inertia

Laravel 12 ha renovado completamente sus starter kits. Ahora puedes elegir entre **React**, **Vue** o **Livewire**, todos con un diseño moderno basado en componentes de [shadcn](https://ui.shadcn.com/) y **Tailwind CSS v4**.

```bash
laravel new mi-proyecto --react
laravel new mi-proyecto --vue
laravel new mi-proyecto --livewire
```

Los kits incluyen autenticación completa, gestión de perfil, modo oscuro y están listos para producción.

## WorkOS AuthKit como alternativa

Una de las opciones más interesantes es la integración con **WorkOS AuthKit**, que te da autenticación social (Google, GitHub, Apple), passkeys y SSO empresarial sin configuración adicional.

```php
// config/services.php
'workos' => [
    'client_id' => env('WORKOS_CLIENT_ID'),
    'secret' => env('WORKOS_API_KEY'),
],
```

## Mejoras internas

Aunque Laravel 12 se centra en los starter kits, también incluye mejoras bajo el capó:

- **Actualizaciones de dependencias**. Todas las dependencias de primera parte han sido actualizadas
- **Soporte continuo para PHP 8.2+**. Compatible con las últimas versiones de PHP
- **Mejoras en el testing**. Herramientas más potentes para tests de integración

## ¿Cómo actualizar?

Si vienes de Laravel 11, la actualización es sencilla. Laravel sigue el versionado semántico, pero el equipo ha trabajado para minimizar los breaking changes:

```bash
composer require laravel/framework:^12.0
```

> Recuerda siempre revisar la [guía de actualización](https://laravel.com/docs/12.x/upgrade) oficial antes de actualizar en producción.

## La comunidad Laravel en España

Desde **Laravel Spain** seguimos organizando meetups y charlas para compartir conocimiento sobre el ecosistema. Si quieres presentar algo sobre Laravel 12 en uno de nuestros eventos, no dudes en contactarnos.

## Conclusión

Laravel 12 consolida la dirección del framework: **experiencia de desarrollador excepcional** con herramientas modernas y flexibles. Los nuevos starter kits hacen que arrancar un proyecto sea más rápido que nunca.

¿Ya has probado Laravel 12? Cuéntanos tu experiencia en nuestro [Discord](https://discord.gg/laravelspain).
