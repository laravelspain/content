# Laravel Spain. Datos

Repositorio público con los datos que alimentan [laravelspain.com](https://laravelspain.com): eventos, paquetes, ofertas de empleo y artículos del blog del ecosistema Laravel en España y Europa.

Cualquier persona puede contribuir abriendo un Pull Request. Los cambios se reflejan automáticamente en la web una vez aprobados.

## Enlaces

- [Web](https://laravelspain.com)
- [Blog](https://laravelspain.com/blog)
- [Eventos](https://laravelspain.com/eventos)
- [Paquetes](https://laravelspain.com/paquetes)
- [Empleos](https://laravelspain.com/empleos)
- [Recursos](https://laravelspain.com/recursos)

## Estructura del repositorio

```
├── events.json      # Eventos de la comunidad
├── packages.json    # Paquetes destacados
├── jobs.json        # Ofertas de empleo
└── blog/            # Artículos del blog (Markdown + frontmatter YAML)
    ├── mi-primer-post.md
    └── laravel-12-novedades.md
```

## Blog. Formato de los posts

Los archivos `.md` dentro de `/blog` se publican automáticamente en laravelspain.com/blog.

### Nombre del archivo
- El nombre del archivo (sin `.md`) se usa como **slug** de la URL.
- Usar kebab-case, sin tildes ni caracteres especiales.
- Ejemplo: `laravel-12-novedades.md` → `laravelspain.com/blog/laravel-12-novedades`

### Frontmatter obligatorio

```yaml
---
title: "Título del artículo"
date: "2026-02-15"
author: "Nombre Apellido"
description: "Resumen corto del artículo (1-2 frases). Se muestra en el listado."
---
```

### Frontmatter opcional

```yaml
---
tags: ["laravel", "novedades"]    # Se muestran como pills/badges
image: "https://..."              # Imagen para Open Graph (compartir en redes)
---
```

### Cuerpo del post
- Markdown estándar (headings, listas, código, links, imágenes, blockquotes, etc.)
- Se puede usar HTML inline si es necesario
- El contenido se renderiza con estilos `prose` de Tailwind Typography
- Escribir en **español**

### Ejemplo completo

```markdown
---
title: "Novedades de Laravel 12"
date: "2026-02-15"
author: "Taylor Otwell"
description: "Un repaso a las principales novedades que trae Laravel 12."
tags: ["laravel", "novedades"]
---

## Introducción

Laravel 12 llega cargado de novedades...

### Nuevo sistema de routing

El nuevo router es un 40% más rápido:

```php
Route::get('/posts', function () {
    return Post::all();
});
```

> Esta es una cita destacada sobre el framework.

## Conclusión

Laravel sigue evolucionando...
```

## Formato de los archivos JSON

### events.json

Cada entrada representa un evento o conferencia.

| Campo       | Tipo    | Obligatorio | Descripción                                        |
|-------------|---------|-------------|----------------------------------------------------|
| date        | string  | Sí          | Fecha ISO `YYYY-MM-DD` (ej: `"2026-04-20"`)       |
| title       | string  | Sí          | Nombre del evento                                  |
| location    | string  | Sí          | Ciudad y país                                      |
| description | string  | Sí          | 1-2 frases en español                              |
| tag         | string  | Sí          | `"Conferencia"`, `"Presencial"`, `"Online"` o `"Híbrido"` |
| tagColor    | string  | Sí          | Clase CSS según tag (ver tabla de colores)         |
| future      | boolean | No          | Solo `true` en eventos futuros. Omitir en pasados  |

### packages.json

Cada entrada representa un paquete del ecosistema Laravel.

| Campo       | Tipo   | Obligatorio | Descripción                        |
|-------------|--------|-------------|------------------------------------|
| name        | string | Sí          | `vendor/package` como en GitHub    |
| description | string | Sí          | 1-2 frases en español              |
| stars       | number | Sí          | Estrellas aproximadas en GitHub    |
| language    | string | Sí          | Lenguaje principal                 |
| url         | string | Sí          | URL del repositorio en GitHub      |

### jobs.json

Cada entrada representa una oferta de empleo.

| Campo       | Tipo   | Obligatorio | Descripción                                    |
|-------------|--------|-------------|------------------------------------------------|
| title       | string | Sí          | Nombre del puesto                              |
| company     | string | Sí          | Nombre de la empresa                           |
| location    | string | Sí          | Ciudad o `"España"`                            |
| type        | string | Sí          | `"Remoto"`, `"Híbrido"` o `"Presencial"`       |
| typeColor   | string | Sí          | Clase CSS según tipo (ver tabla de colores)    |
| salary      | string | Sí          | Rango salarial (ej: `"35K - 45K €"`)          |
| description | string | Sí          | 1-2 frases en español                          |

## Tabla de colores

Valores válidos para `tagColor` (eventos) y `typeColor` (empleos):

| Valor       | Clase CSS                                             |
|-------------|-------------------------------------------------------|
| Conferencia | `bg-spain-red/10 text-spain-red`                      |
| Presencial  | `bg-green-500/10 text-green-600 dark:text-green-400`  |
| Online      | `bg-blue-500/10 text-blue-600 dark:text-blue-400`     |
| Híbrido     | `bg-purple-500/10 text-purple-600 dark:text-purple-400` |
| Remoto      | `bg-green-500/10 text-green-600 dark:text-green-400`  |

## Cómo contribuir

1. Haz un **fork** de este repositorio.
2. Crea tu archivo `.md` en `/blog` o edita los JSON correspondientes.
3. Abre un **Pull Request** con una descripción breve del cambio.
4. El PR será revisado antes de hacer merge.

> Si no estás seguro de algún campo, consulta las tablas de formato o abre un issue.

## Reglas generales

- Todo el contenido en **español**
- Las fechas siempre en formato ISO: `YYYY-MM-DD`
- No incluir contenido promocional o spam
- Los posts de blog deben estar relacionados con Laravel, PHP o el ecosistema

## Licencia

Los datos de este repositorio son públicos y están disponibles bajo la [licencia MIT](https://opensource.org/licenses/MIT).
