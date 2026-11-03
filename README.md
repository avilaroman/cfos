<p align="center">
<img src="docs/assets/cloudflareOS.svg" alt="Cloudflare OS" width="480">
</p>

<h1 align="center">Personalizado para tu empresa</h1>

<p align="center">
Implementa una versión específica (fijada) de Cloudflare OS, con control total sobre la identidad visual, el inicio de sesión, las integraciones, las rutas y las actualizaciones.
</p>

<p align="center">
<a href="https://developers.cloudflare.com/workers/"><img alt="Cloudflare Workers" src="https://img.shields.io/badge/Cloudflare-Workers-F6821F?logo=cloudflare&logoColor=white"></a>
<a href="https://nodejs.org/"><img alt="Node.js 24.19+" src="https://img.shields.io/badge/Node.js-24.19+-5FA04E?logo=nodedotjs&logoColor=white"></a>
<a href="https://pnpm.io/"><img alt="pnpm 11.17" src="https://img.shields.io/badge/pnpm-11.17-F69220?logo=pnpm&logoColor=white"></a>
<a href="https://www.typescriptlang.org/"><img alt="TypeScript 7" src="https://img.shields.io/badge/TypeScript-7-3178C6?logo=typescript&logoColor=white"></a>
<a href="https://github.com/cloudflare/cloudflare-os"><img alt="Cloudflare OS upstream" src="https://img.shields.io/badge/upstream-Cloudflare_OS-24292F?logo=github"></a>
</p>

> [!IMPORTANT]
> Cloudflare OS es un software en fase de acceso anticipado. Fija las versiones provenientes del repositorio original (*upstream*), revisa los cambios y verifica los límites de confianza antes de realizar cualquier actualización en un entorno de producción. ## Inicio rápido

Con [Node.js 24.19 o una versión posterior](https://nodejs.org/) y [pnpm 11.17](https://pnpm.io/installation) instalados:

```sh
npx cf-os init my-os
```

Este comando clona la plantilla inicial, realiza la instalación, ofrece crear un repositorio privado en GitHub y subir el código, configura el inicio de sesión, Workers y AI Gateway, genera el archivo `deployment.jsonc` y ofrece realizar el despliegue. La sección [Despliegue](#deploy) detalla cada paso, incluyendo el procedimiento manual, mientras que [Personalización](#customization) explica qué elementos se pueden modificar posteriormente. El resto del contenido de esta página es opcional.

## Visión general

Este repositorio añade controles de despliegue sobre una versión específica (fijada) de [Cloudflare OS](https://github.com/cloudflare/cloudflare-os), sin modificar el código fuente original.

| Control | Elementos bajo tu gestión |
| --- | --- |
| Identidad visual | Nombre del sitio, logotipo y color de acento; se pueden modificar en [`/admin`](docs/customization.md#branding) sin necesidad de realizar un nuevo despliegue. |
| Identidad | Método de inicio de sesión y lista de administradores permitidos; esta plantilla configura el modo [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/access-controls/applications/http-apps/self-hosted-public-app/). |
| Enrutamiento | Un [dominio personalizado](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/) para producción o una ruta `workers.dev` para pruebas. |
| Datos | Recursos KV/R2 existentes o mediante [aprovisionamiento automático](https://developers.cloudflare.com/workers/wrangler/configuration/#automatic-provisioning). |
| Integraciones | *Gatekeepers* y enlaces a servicios gestionados por el *wrapper*, sin necesidad de modificar el código fuente original. |
| IA | Catálogo de modelos de [Workers AI](https://developers.cloudflare.com/workers-ai/) a través de [AI Gateway](https://developers.cloudflare.com/ai-gateway/), listo para usar y sin necesidad de tokens de API; incluye la configuración de proveedores y pasarelas (*gateways*). |
| Operaciones | [Registros estructurados, seguimientos (traces), informes de errores explícitos](docs/observability.md), validación, orden de despliegue y actualizaciones |

### Arquitectura

<img src="docs/assets/architecture.svg" alt="Arquitectura de despliegue de Cloudflare OS: los usuarios acceden a una ruta pública gestionada por un Worker «router», el cual sirve el frontend y actúa como proxy para dirigir las peticiones a `/api` hacia el backend de Workshop, y las peticiones a `/gatekeeper/<nombre>` hacia el Gatekeeper correspondiente. Detrás de este se encuentra la versión específica de Cloudflare OS, que contiene el kernel de Workshop, Gadgets, Blueprints y los Gatekeepers predeterminados. Los «service bindings» (enlaces de servicio) conectan este sistema con los Workers y recursos que forman parte de este repositorio: AI Gateway (sin token de API), Gatekeepers personalizados, el componente de reporte de errores (Error Reporter), y almacenamiento en KV y R2.">

El despliegue consta de hasta siete Workers, además de los Gatekeepers opcionales que decidas incluir. Un **router** gestiona la ruta pública y sirve el frontend, actuando como proxy para dirigir las peticiones a `/api` hacia el backend de Workshop, y las peticiones a `/gatekeeper/<nombre>` hacia el Gatekeeper cuyo nombre coincida con el enlace (binding). Componentes como Workshop, Context, Scheduler, Web Search, los Gatekeepers personalizados y el Error Reporter se sitúan detrás, sin rutas propias y accesibles únicamente a través de «service bindings».

El comando de despliegue genera archivos temporales de Wrangler a partir de configuraciones base originales, compila el frontend en modo «Cloudflare Access», despliega el Error Reporter (privado), los Gatekeepers y Workshop —antes de desplegar el router que los vincula—, y elimina los archivos generados, incluso en caso de fallo. Los secretos (credenciales) nunca se incluyen en la configuración bajo control de versiones.

### Si solo buscas personalizar la identidad de marca (branding)

Existe un flujo de despliegue gestionado que implementa la versión original (upstream) directamente en tu cuenta de Cloudflare, sin necesidad de este repositorio. No requiere compilación local, configura automáticamente el inicio de sesión y tus correos electrónicos de administrador, y permite personalizar toda la interfaz `/admin`: nombre del sitio, logotipo, color de acento, anuncios, instrucciones para los agentes, blueprints destacados y los conectores a los que pueden acceder tus usuarios. Los Gatekeepers integrados, como los de GitHub o Google, pueden configurarse utilizando tus propias credenciales OAuth.

<a href="https://os.cloudflare.app/d
