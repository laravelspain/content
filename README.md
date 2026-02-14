# Laravel Spain — Datos

Repositorio público con los datos que alimentan [laravelspain.com](https://laravelspain.com): eventos, paquetes y ofertas de empleo del ecosistema Laravel en España y Europa.

Cualquier persona puede contribuir abriendo un Pull Request. Los cambios se reflejan automáticamente en la web una vez aprobados.

## Enlaces

- [Web](https://laravelspain.com)
- [Eventos](https://laravelspain.com/eventos)
- [Paquetes](https://laravelspain.com/paquetes)
- [Empleos](https://laravelspain.com/empleos)
- [Recursos](https://laravelspain.com/recursos)

## Estructura del repositorio

| Archivo          | Contenido                                                        |
|------------------|------------------------------------------------------------------|
| `events.json`    | Eventos y conferencias relacionados con Laravel, PHP y desarrollo web |
| `packages.json`  | Paquetes destacados del ecosistema Laravel                       |
| `jobs.json`      | Ofertas de empleo Laravel en España                              |

## Cómo contribuir

1. Haz un **fork** de este repositorio.
2. Edita el archivo JSON correspondiente respetando el formato documentado más abajo.
3. Abre un **Pull Request** con una descripción breve del cambio (por ejemplo: "Añadir evento X" o "Actualizar estrellas de paquete Y").
4. El PR será revisado antes de hacer merge.

> Si no estás seguro de algún campo, consulta las tablas de formato o abre un issue.

## Formato de los archivos

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

## Licencia

Los datos de este repositorio son públicos y están disponibles bajo la [licencia MIT](https://opensource.org/licenses/MIT).
