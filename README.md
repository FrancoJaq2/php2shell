# 🐘 php2shell — Guía divulgativa y de mitigación para no técnicos

> **Caso de estudio:** las vulnerabilidades críticas de PHP de octubre de 2026 (CVE-2026-109618 y CVE-2026-109619)
> **Autor:** estudiante de Técnico en Ciberseguridad
> **Público objetivo:** dueños de sitios web, community managers, emprendedores y cualquier persona que use WordPress u otro CMS sin ser programadora.
> **Última actualización:** 26 de octubre de 2026

## 📥 Descarga el instructivo

**[Instructivo paso a paso en PDF (6 páginas)](docs/php2shell-instructivo.pdf)** — pensado para descargar, imprimir y compartir. Incluye el mensaje listo para enviar a tu proveedor de hosting, cómo verificar si tu sitio fue comprometido y qué hacer si lo fue.

---

## Qué hacer hoy

- El fallo está en **PHP**, el motor que tu hosting pone debajo. Actualizar WordPress, themes o plugins **no lo arregla**.
- **Averigua tu versión de PHP**: en WordPress, *Herramientas > Salud del sitio > Información > Servidor*.
- **Pide el parche a tu proveedor** (o cambia la versión desde tu panel, si te lo permite). Necesitas la versión exacta: 8.5.12, 8.4.27, 8.3.36 u 8.2.35 como mínimo.
- **Pon un WAF** (por ejemplo, el plan gratuito de Cloudflare) mientras esperas. Reduce el riesgo, pero no reemplaza el parche.
- **Revisa si tu sitio ya fue comprometido**, incluso si ahora está parchado. Los detalles están en el [instructivo](docs/php2shell-instructivo.pdf).

---

## Índice

- [¿Por qué existe este repositorio?](#por-qué-existe-este-repositorio)
- [Resumen en 30 segundos](#resumen-en-30-segundos)
- [¿Qué pasó? Los dos fallos](#qué-pasó-los-dos-fallos)
- [¿A quién afecta y qué versiones lo arreglan?](#a-quién-afecta-y-qué-versiones-lo-arreglan)
- [Línea de tiempo](#línea-de-tiempo)
- [Impacto a nivel mundial](#impacto-a-nivel-mundial)
- [Situación en Chile](#situación-en-chile)
- [¿Qué tan grave es?](#qué-tan-grave-es)
- [Para quienes sí administran el servidor](#para-quienes-sí-administran-el-servidor)
- [Mitos y errores comunes](#mitos-y-errores-comunes)
- [Cuidado con las estafas](#cuidado-con-las-estafas)
- [Glosario para no técnicos](#glosario-para-no-técnicos)
- [Fuentes y descargo de responsabilidad](#fuentes-y-descargo-de-responsabilidad)
- [Cómo contribuir](#cómo-contribuir)

> ⚠️ **Sobre los datos marcados con ⚠️:** provienen de reportes que todavía no tienen un enlace verificable en este repositorio. Se mantienen visibles porque son relevantes, pero deben leerse como *por confirmar* hasta que se les añada su fuente.

---

## ¿Por qué existe este repositorio?

Cuando salió la alerta de **php2shell**, casi toda la información técnica estaba escrita para administradores de sistemas: hablar de `php.ini`, de `session.upload_progress`, de *use-after-free*. Nada de eso le sirve a la persona que tiene una tienda online montada en WordPress y que contrató un hosting porque no sabe programar.

Este repositorio nace de esa brecha. La idea es simple:

> **No necesitas saber programar para proteger tu sitio, pero sí necesitas saber qué preguntar y a quién.**

Aquí vas a encontrar qué pasó, por qué es grave, qué se está viendo a nivel mundial y en Chile, y —lo más importante— un [instructivo en PDF](docs/php2shell-instructivo.pdf) que **puedes ejecutar sin tocar una sola línea de código**.

---

## Resumen en 30 segundos

- Se descubrieron **dos fallos críticos en el propio lenguaje PHP** (no en tu WordPress, no en tu theme, no en tus plugins).
- Permiten que un atacante **ejecute código en tu servidor sin necesidad de usuario ni contraseña**.
- PHP se usa en cerca del **70% de las aplicaciones y sitios expuestos a internet**, pero quedan vulnerables solo los que **no han actualizado**.
- **La única solución completa es que tu proveedor de hosting actualice PHP.** Actualizar WordPress no arregla esto.
- Los detalles técnicos **se filtraron antes de que existiera el parche**, y se reportó explotación observada en honeypots el mismo día de la publicación.
- Si estás en Shopify, Wix o Squarespace, el proveedor administra la plataforma: no tienes un PHP propio que parchear.

---

## ¿Qué pasó? Los dos fallos

El equipo de PHP publicó el **21 de octubre de 2026** dos vulnerabilidades que viven en el **intérprete del lenguaje**, es decir, en el "motor" que hace funcionar a PHP.

| CVE | Nombre técnico | Qué significa en simple |
|---|---|---|
| **CVE-2026-109618** | RCE por corrupción tras falta de memoria (*OOM*) | Si se fuerza al servidor a quedarse sin memoria de una forma concreta, PHP queda a medio configurar y la limpieza posterior trata datos inválidos como válidos. |
| **CVE-2026-109619** | RCE por estado global obsoleto (*use-after-free*) | Una petición posterior atendida por el mismo proceso puede usar restos de memoria de una petición anterior que ya debían haberse liberado. |

**Lo importante:** ambas permiten **ejecución remota de código (RCE) sin autenticación**. En palabras simples: alguien desde internet puede hacer que tu servidor ejecute instrucciones que él elige, con los mismos permisos que el proceso de PHP. De ahí el nombre del caso: *php2shell* (de PHP a una shell).

**Software afectado:** PHP-FPM, Apache con `mod_php`, `php-cgi`, FrankenPHP y LiteSpeed. Es decir, prácticamente cualquier forma habitual de servir PHP. Matiz: PHP evalúa el riesgo como mayor en `mod_php`, donde un exploit demostrado dispara el fallo antes de que corra el código de la aplicación; con PHP-FPM la ruta demostrada requiere además una operación adecuada en la aplicación. Aun así, todo sitio sin parchear debe tratarse como expuesto.

---

## ¿A quién afecta y qué versiones lo arreglan?

Afecta a **todas las instalaciones de PHP que sirven peticiones web** y no estén actualizadas.

| Rama | Versión corregida | Observación |
|---|---|---|
| 8.5 | **8.5.12** | Soporte activo hasta dic. 2027; seguridad hasta dic. 2029. |
| 8.4 | **8.4.27** | Soporte activo hasta dic. 2026; seguridad hasta dic. 2028. |
| 8.3 | **8.3.36** | Solo seguridad, hasta dic. 2027. |
| 8.2 | **8.2.35** | Parchada, pero **llega a fin de vida el 31 de dic. 2026**. Si vas a migrar, elige una rama más nueva. |
| **8.1 y anteriores** | — | ❌ **Sin parche oficial.** Solo queda migrar (o que tu proveedor aplique un *backport*). |

> ⚠️ **Dato clave para no técnicos:** si tu hosting dice tener "PHP 8.2" pero no te dice el número completo, **no te está dando la información suficiente**. Necesitas que te confirmen la versión exacta (por ejemplo, `8.3.36`).

**¿Y los CMS?** WordPress, Joomla y Drupal corren sobre PHP, así que dependen de la versión que ponga tu hosting. Shopify, Wix y Squarespace son plataformas administradas por su proveedor: no tienes un PHP propio que parchear.

---

## Línea de tiempo

| Fecha | Qué ocurrió |
|---|---|
| **20 de octubre** | Se **filtran detalles** de las vulnerabilidades antes del anuncio programado. **CERT-EU** emite el aviso 2026-018 recomendando aislar las aplicaciones PHP expuestas a internet. |
| **21 de octubre** | PHP publica los parches de **emergencia**: 8.5.12, 8.4.27, 8.3.36 y 8.2.35. El **NVD** publica CVE-2026-109618 (14:17 UTC). **watchTowr** reporta explotación observada en su red de honeypots. La agencia australiana **ACSC** emite una alerta crítica. |
| **Julio 2026** | Comienza la auditoría de seguridad de PHP dentro del programa "Patch the Planet" de OpenAI, con apoyo de IA. ⚠️ *(fuente por enlazar)* |
| **17 de octubre** | Los honeypots **MadPot de Amazon** habrían detectado el primer intento de explotación, cuatro días antes del anuncio. ⚠️ *(fuente por enlazar)* |
| **21 de octubre** | **ANCI / CSIRT Nacional de Chile** emite la alerta **AVC26-00740**. ⚠️ *(fuente por enlazar)* **CISA**, **FBI** y **NSA** emiten alertas o recomendaciones. ⚠️ *(fuente por enlazar)* |
| **22 de octubre** | **Cloudflare** reporta millones de intentos de explotación a nivel global. ⚠️ *(fuente por enlazar)* |
| **23 de octubre** | Prensa chilena reporta que no hay ataques exitosos confirmados en Chile, aunque sí intentos. ⚠️ *(fuente por enlazar)* |
| **24 de octubre** | El investigador **Adel Karimi** confirma 35 shells ejecutadas en sus honeypots. ⚠️ *(fuente por enlazar)* |
| **26 de octubre** | Se identifica un tercer fallo relacionado, **CVE-2026-115560**, que PHP describiría como dependiente de una configuración específica y no como un *bypass* de los parches. ⚠️ *(fuente por enlazar)* |

> 📌 **Nota de verificación de fuentes:** hubo reportes iniciales contradictorios sobre el momento exacto en que CISA incluyó estas CVE en su catálogo **KEV**. Además, una publicación del 21 de octubre indicaba que aún no se reportaba explotación, mientras que el reporte de MadPot (si se confirma) la ubica el 17. En una crisis de este tipo conviene cruzar fuentes **y fechas** antes de dar un dato por cerrado.

---

## Impacto a nivel mundial

### La reacción preventiva

- **CERT-EU** recomendó **aislar las aplicaciones expuestas a internet** hasta aplicar los parches.
- **Países Bajos y Finlandia** habrían sacado sitios de gobierno de línea por precaución, y **Canadá** habría retirado algunos sitios federales. ⚠️ *(fuente por enlazar)* El **Cyber Centre de Canadá** sí emitió la alerta AL26-026 pidiendo actualizar de inmediato.

### La explotación

- **watchTowr** reportó explotación observada en sus honeypots el 21 de octubre.
- **Bitsight** asigna puntajes de probabilidad de explotación de 9,91 (CVE-2026-109618) y 9,84 (CVE-2026-109619) sobre 10, por la alta conversación en fuentes abiertas y foros de la dark web, y reporta que ya hay reproducciones en laboratorio.
- **Cloudflare**: decenas de miles de intentos en 45 minutos y millones al 22 de octubre. ⚠️ *(fuente por enlazar)*
- **Honeypots de Adel Karimi**: 35 shells confirmadas el 24 de octubre, con cadenas de peticiones `POST multipart`. ⚠️ *(fuente por enlazar)*

### Actores y herramientas (por confirmar)

- Campaña atribuida a **Lazarus** por la firma Cyfirma, con "confianza moderada", y 216 servidores identificados sin parchear o parcialmente protegidos. ⚠️ *(fuente por enlazar)*
- **Wiz** habría demostrado que los fallos pueden encadenarse para lograr RCE sin autenticación en instalaciones por defecto de WordPress, Drupal y Joomla. ⚠️ *(fuente por enlazar)*
- **Sophos** habría detectado un payload llamado "Potassium", derivado de Mirai. ⚠️ *(fuente por enlazar)*
- **Rapid7** advertiría que muchos PoC públicos solo funcionan en combinaciones muy específicas de sistema operativo, versión de PHP y servidor web. **Eso es una ventana para parchear, no una razón para confiarse.** ⚠️ *(fuente por enlazar)*

### El factor IA

Se ha comentado que los fallos existían desde hacía mucho tiempo y que modelos de IA de vanguardia ayudaron a descubrir y encadenar los pasos necesarios. ⚠️ *(fuente por enlazar)* Sea o no el caso, la lección práctica es la misma: el tiempo entre "parche publicado" y "exploit funcional" se ha vuelto muy corto.

---

## Situación en Chile

| Aspecto | Estado |
|---|---|
| **Alerta oficial** | ANCI / CSIRT Nacional habría emitido la alerta **AVC26-00740** el 21 de octubre de 2026, pidiendo actualizar de inmediato. ⚠️ *(fuente por enlazar)* |
| **Actualización de la alerta** | Se habrían incorporado antecedentes de intentos de explotación desde el 17 de octubre y la identificación de CVE-2026-115560. ⚠️ *(fuente por enlazar)* |
| **Ataques exitosos confirmados** | Hasta el 23 de octubre, la prensa no reportaba ataques exitosos en el país. ⚠️ *(fuente por enlazar)* |
| **Riesgo local** | Un académico de la Universidad de Valparaíso advirtió que en Chile hay sitios de gobierno, tiendas online y medios de comunicación montados sobre estos sistemas. ⚠️ *(fuente por enlazar)* |

> 🧠 **Lectura crítica:** que no haya ataques exitosos *reportados* no significa que no hayan ocurrido. Significa que no hay evidencia pública confirmada. La ausencia de reporte no es prueba de ausencia de incidente.

---

## ¿Qué tan grave es?

- **Puntuación CVSS v4.0: 9,2 / 10 para ambas CVE — categoría crítica.**
- **Por qué tan alta:**
  - No requiere credenciales.
  - Se ataca por red.
  - El atacante ejecuta código con los privilegios del proceso PHP.
- **Matiz importante:** el famoso **70%** es la cuota de uso de PHP en internet, **no** el porcentaje de sitios vulnerables. Quedan expuestos los que **no han actualizado**, sobre todo los que usan versiones sin soporte.

---

## Para quienes sí administran el servidor

**El parche es la solución.** Actualiza **todas** las versiones de PHP instaladas, aunque tus sitios no las usen, confirma con `php -v` y reinicia PHP-FPM o Apache. Las actualizaciones automáticas del sistema pueden no incluir PHP si se instaló desde un repositorio aparte. En contenedores, reconstruye la imagen con la versión parchada.

Si hoy **no puedes** parchar, hay mitigaciones temporales. Son **parciales** y no cuentan como arreglo:

| Fallo | Medida temporal | Notas |
|---|---|---|
| **CVE-2026-109618** | Fijar `memory_limit = -1` de forma obligatoria (`php_admin_value`) en todos los *pools* y directorios. Bloquear peticiones multipart en el frontend si no las usas. | Quita el tope de memoria: úsalo solo como medida temporal y vigila el consumo de recursos. |
| **CVE-2026-109619** | `session.upload_progress.enabled = 0` | Debe ir en `php.ini` o en la configuración del servidor, **no** con `ini_set()` ni `.user.ini`. En PHP 8.5, agrega `fatal_error_backtraces = 0`. |

```ini
; php.ini (temporal; no reemplaza el parche)
session.upload_progress.enabled = 0
memory_limit = -1
fatal_error_backtraces = 0     ; solo PHP 8.5

; En un pool de PHP-FPM, para fijar el límite de forma obligatoria:
php_admin_value[memory_limit] = -1
```

Después del cambio es **imprescindible reiniciar PHP-FPM o Apache**.

> ⚠️ **Ojo:** desactivar `session.upload_progress` bloquea **una sola ruta demostrada** de CVE-2026-109619 y **no hace nada** contra CVE-2026-109618. Si sospechas compromiso en un servidor, captura evidencia volátil **antes** de parchear o reiniciar: reiniciar los workers borra el estado en memoria que abusa CVE-2026-109619.

---

## Mitos y errores comunes

| Mito | Realidad |
|---|---|
| "Actualizo WordPress y ya estoy protegido" | ❌ Falso. El fallo está en PHP, no en WordPress. |
| "Mi plugin de seguridad me protege" | ❌ Falso. El fallo se dispara **antes** de que se ejecute el código de tu aplicación. Un plugin no te cubre de esto. |
| "Como no soy un objetivo interesante, no me van a atacar" | ❌ Falso. La mayoría de los intentos son **bots automatizados** que escanean todo internet sin elegir víctimas. |
| "Si no veo nada raro en mi web, estoy bien" | ❌ Falso. El fallo permite ejecutar código **sin dejar rastro visible** en la página. Hay que buscar señales indirectas (ver el [instructivo](docs/php2shell-instructivo.pdf)). |
| "Espero a que mi hosting avise" | ⚠️ Riesgoso. Abre tú el ticket y pide la confirmación por escrito. |
| "El WAF ya me protege, no necesito el parche" | ❌ Falso. El WAF es defensa en profundidad, no un sustituto. |

---

## Cuidado con las estafas

En crisis de seguridad de esta magnitud aparecen estafas. Los ciberdelincuentes aprovechan el caos para enviar correos o mensajes que **suplantan a PHP, a WordPress o a tu proveedor de hosting**, instándote a hacer clic en un enlace para una "actualización de seguridad urgente". Ese enlace suele llevar a **phishing** para robar tus credenciales o descargar malware.

> **Regla de oro:** nunca hagas clic en enlaces de actualizaciones no solicitadas. Ve siempre **directamente** al panel de tu hosting o de WordPress escribiendo la URL tú mismo.

**Advertencia adicional para estudiantes y profesionales:** se han visto **repositorios falsos en GitHub** que se hacen pasar por PoC de php2shell pero contienen malware dirigido a profesionales de seguridad. No ejecutes herramientas de fuentes no verificadas. ⚠️ *(fuente por enlazar)*

---

## Glosario para no técnicos

| Término | Qué significa |
|---|---|
| **PHP** | El lenguaje de programación que usan la mayoría de los sitios web para funcionar. Es el "motor". |
| **Intérprete** | El programa que ejecuta el código PHP en el servidor. |
| **CVE** | Un identificador único y público para una vulnerabilidad conocida. |
| **RCE (ejecución remota de código)** | Cuando un atacante consigue que el servidor ejecute instrucciones que él elige, desde internet. |
| **Sin autenticación** | Que el atacante **no necesita** usuario ni contraseña para intentarlo. |
| **Use-after-free** | Un tipo de fallo de memoria: el programa usa un espacio de memoria que ya debía haber liberado. |
| **CVSS** | Una escala de 0 a 10 para medir la gravedad de una vulnerabilidad. 9,0 o más es "crítico". |
| **Parche** | La actualización que corrige el fallo. |
| **EOL (fin de vida)** | Versión que ya no recibe parches de seguridad. |
| **WAF** | Firewall de Aplicaciones Web: filtra tráfico malicioso antes de que llegue a tu servidor. |
| **Honeypot** | Un servidor "señuelo" puesto a propósito para observar a los atacantes. |
| **PoC (Prueba de Concepto)** | Código que demuestra que una vulnerabilidad se puede explotar. |
| **Backport** | Aplicar el parche de seguridad a una versión antigua, sin cambiar de versión. |
| **KEV** | Catálogo de CISA de vulnerabilidades que se sabe que están siendo explotadas. |

---

## Fuentes y descargo de responsabilidad

**Fuentes enlazadas** (consultadas hasta el 26 de octubre de 2026):

- [Help Net Security — PHP2Shell: Critical PHP vulnerabilities patched](https://helpnetsecurity.com/2026/10/21/php2shell-php-vulnerabilities-patched-cve-2026-109618-cve-2026-109619)
- [Cyber Centre de Canadá — Alerta AL26-026](https://cyber.gc.ca/en/alerts-advisories/al26-026-critical-vulnerabilities-impacting-php-cve-2026-109618-cve-2026-109619)
- [Tanium — Responding to CVE-2026-109618 and CVE-2026-109619](https://www.tanium.com/blog/php2shell-responding-to-cve-2026-109618-and-cve-2026-109619)
- [Armis (ServiceNow) — How to find, prioritize and patch affected systems](https://www.servicenow.com/community/armis-articles/php2shell-how-to-find-prioritize-and-patch-affected-systems/ta-p/3612792)
- [Bitsight — Critical vulnerability alert](https://www.bitsight.com/blog/critical-vulnerability-alert-cve-2026-109618-and-cve-2026-109619-php2shell)
- [HeroDevs — PHP2Shell y PHP en fin de vida](https://www.herodevs.com/blog-posts/php2shell-cve-2026-109618-cve-2026-109619-php-rce)
- [Laravel Forge — PHP security release, octubre 2026](https://laravel.com/forge/docs/knowledge-base/php-security-release-october-2026)
- [Sevalla — Aviso de seguridad](https://docs.sevalla.com/security/advisories/php-cve-2026-109618.md)
- [Cyber Daily — Alerta de la ACSC (Australia)](https://www.cyberdaily.au/security/14334-critical-alert-aussie-cyber-agency-warns-businesses-to-monitor-php-vulnerabilities)
- [Soporte por rama de PHP (8.5.12)](https://pasqualepillitteri.it/en/news/26141/php-8-5-12-released-90-fixes-pdo-dom-soap)

**Pendientes de enlazar** (marcados con ⚠️ en el texto): alerta AVC26-00740 de ANCI, avisos de CISA/FBI/NSA, reportes de Cloudflare y de MadPot, atribución a Lazarus (Cyfirma), shells de Adel Karimi, reportes de Wiz, Sophos y Rapid7, CVE-2026-115560 y programa "Patch the Planet".

**Limitaciones que debes tener en cuenta:**

- La situación **está en evolución**. Las cifras y atribuciones pueden cambiar con nueva información.
- La información sobre víctimas reales de producción es **limitada**: la mayor parte de la evidencia de explotación exitosa proviene de **honeypots**, no de organizaciones confirmadas como comprometidas.
- Algunas atribuciones (por ejemplo, a Lazarus) son de **"confianza moderada"**, no concluyentes.
- Este material tiene **fin educativo y divulgativo**. No sustituye el asesoramiento de un profesional ni las instrucciones oficiales de tu proveedor de hosting.

---

## Cómo contribuir

Si encuentras un error, una fuente mejor o una actualización relevante:

- Abre un **Issue** describiendo el cambio y citando la fuente.
- Si quieres proponer una edición, haz un **Pull Request** explicando qué mejora y por qué.

**Criterio principal:** toda afirmación debe poder rastrearse a una fuente verificable. En seguridad, la trazabilidad es parte de la calidad. Los datos marcados con ⚠️ son la lista de tareas pendientes más útil para empezar.

---

<p align="center">
  <i>"La seguridad no es un producto que se compra, sino una práctica que se mantiene."</i><br>
  <sub>Material divulgativo — Técnico en Ciberseguridad</sub>
</p>
