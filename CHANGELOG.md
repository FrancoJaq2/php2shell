# Historial de cambios

Todos los cambios relevantes de este repositorio se registran aquí, con la entrada más reciente arriba. Como la situación de PHP2Shell evoluciona, cada entrada indica la fecha para que quien lea sepa qué tan actualizada está la información.

## 2026-10-26

### Añadido
- Instructivo en PDF (`docs/php2shell-instructivo.pdf`) con el paso a paso para personas no técnicas: cómo averiguar la versión de PHP, mensaje listo para el proveedor de hosting, WAF, verificación de compromiso y plan de respuesta.
- Bloque "Qué hacer hoy" al inicio del README.
- Fuentes enlazadas al final del README.
- Marcas ⚠️ *(fuente por enlazar)* en los datos que aún no tienen una fuente verificable.
- Archivos `LICENSE` (CC BY 4.0) y `CHANGELOG.md`.

### Cambiado
- El instructivo completo salió del README y ahora se enlaza al PDF.
- Mitigaciones temporales separadas por CVE: `session.upload_progress.enabled = 0` solo cubre una ruta de CVE-2026-109619; para CVE-2026-109618 se indica `memory_limit = -1` forzado y bloquear peticiones multipart. Se aclara que estos valores van en `php.ini` o en la configuración del servidor, no en `.user.ini`.
- Tabla de versiones con notas de soporte: PHP 8.2 llega a fin de vida el 31 de diciembre de 2026.
- Aclaración de que el riesgo es mayor con `mod_php` que con PHP-FPM.
- Aclaración de que Shopify, Wix y Squarespace no requieren que el usuario parche PHP.
- Línea de tiempo reordenada: primero los hechos con fuente enlazada.
- Títulos de sección sin emojis para que funcione el índice en GitHub.

### Corregido
- Error de redacción ("fine" por "fin").
- Fila duplicada "WAF / KEV" en el glosario.
- Se dejó de llamar "fuerza bruta" a los intentos de explotación.

## 2026-10-26 (versión inicial)

- Primera publicación de la guía divulgativa sobre las vulnerabilidades CVE-2026-109618 y CVE-2026-109619.
