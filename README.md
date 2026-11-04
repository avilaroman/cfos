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
git clone https://github.com/avilaroman/cfos.git
cd cfos
npm i -g pnpm
pnpm install --pm-on-fail=ignore
npx cf-os init tuos
```

Este comando clona la plantilla inicial "tuos" que lo puedes cambiar por tu nombre, realiza la instalación, ofrece crear un repositorio privado en GitHub y subir el código, configura el inicio de sesión, Workers y AI Gateway, genera el archivo `deployment.jsonc` y ofrece realizar el despliegue. La sección [Despliegue](#deploy) detalla cada paso, incluyendo el procedimiento manual, mientras que [Personalización](#customization) explica qué elementos se pueden modificar posteriormente. El resto del contenido de esta página es opcional.

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

### Piense en una suite ofimática

La experiencia de usuario básica de Cloudflare OS es similar a la de una suite ofimática en línea, como Google Docs o MS Office. Pero imagine que, en lugar de un conjunto fijo de tipos de archivo (documentos, hojas de cálculo, presentaciones), cada archivo —o «Gadget»— sea, en esencia, su propia aplicación personalizada, creada por una IA para satisfacer exactamente sus necesidades.

Al igual que los documentos de oficina, cada «gadget» es privado por defecto, pero puede compartirse —de forma segura— para colaborar con su equipo o sus amigos.

Al igual que los documentos de oficina, puede tener miles de ellos. Puede crearlos cuando quiera.

Al igual que los documentos de oficina, puede partir de «plantillas», a las que llamamos «Blueprints» (planos). Pero, mientras que una plantilla de oficina es simplemente contenido, un «Blueprint» define una aplicación completa.

Al igual que con los documentos de oficina, puede crear nuevas plantillas («blueprints») a partir de sus propios documentos («gadgets») y compartirlas con otros. Sin embargo, al hacerlo, está compartiendo el código de toda una aplicación.

### Es, en cierto modo, un sistema operativo

El uso del término «sistema operativo» (OS) no es *solo* una estrategia de marketing. A nivel técnico, Cloudflare OS es, de hecho, análogo a un sistema operativo.

| Sistema operativo convencional | Cloudflare OS              |
|--------------------------------|----------------------------|
| kernel (núcleo)                 | packages/workshop-backend  |
| controladores (drivers)          | packages/gatekeeper-*      |
| shell (intérprete de comandos) | packages/workshop-frontend |
| procesos                       | gadgets                    |
| ejecutables                    | blueprints                 |
| usuarios                       | usuarios                   |
| ACL (listas de control de acceso)| permisos compartidos      |
| ??? | agentes                    |

Nuestro «kernel» se encuentra en el paquete `workshop-backend`. Este componente realiza, efectivamente, muchas funciones similares a las de los núcleos de los sistemas operativos reales: conecta a los usuarios con programas y dispositivos (lo que denominamos «Gadgets» y «Gatekeepers»), al tiempo que garantiza la seguridad mediante el aislamiento de aplicaciones (*sandboxing*) y la aplicación de controles de acceso.

En esta analogía, los «Gatekeepers» —que conectan a los usuarios y agentes con servicios externos— funcionan como los controladores (*drivers*), que conectan a los usuarios y programas con dispositivos externos. Hay un aspecto que los sistemas operativos tradicionales no gestionan adecuadamente hoy en día, pero que Cloudflare OS sí aborda: los agentes de IA. Si lo analizamos, se trata de una funcionalidad que, en realidad, falta en los sistemas operativos convencionales. Consideramos que los agentes de IA no pueden tratarse simplemente como usuarios; deben rendir cuentas ante un usuario humano y, al mismo tiempo, contar con sus propios permisos restringidos. Estos agentes operan escribiendo fragmentos de código y ejecutándolos al instante. El modelo de seguridad idóneo para todo esto es la «seguridad basada en capacidades» (*capability-based security*), en lugar de las listas de control de acceso. ¿Se entiende a qué me refiero? Quizás los sistemas operativos tradicionales también deberían ofrecer un tratamiento especial a los agentes de IA.

### Construido sobre Workers, por el equipo de Workers

Cloudflare OS está desarrollado sobre [Cloudflare Workers](https://workers.cloudflare.com) y hace un uso intensivo de [Durable Objects](https://developers.cloudflare.com/durable-objects/), [Dynamic Workers](https://blog.cloudflare.com/dynamic-workers/) y, en particular, [Facets](https://blog.cloudflare.com/durable-object-facets-dynamic-workers/). Cada espacio de trabajo (*workspace*) constituye un *Durable Object* independiente; cada *Gadget* se ejecuta en un *Dynamic Worker Facet*; y los *Gatekeepers* instalan *facets* en cada espacio de trabajo para gestionar el acceso a servicios remotos.

De hecho, Cloudflare OS ha sido creado por las mismas personas que desarrollaron Workers. Utiliza funcionalidades de vanguardia del *runtime* de Workers; es más, *Dynamic Workers*, *Facets* y otras características se añadieron al *runtime* específicamente para dar soporte a Cloudflare OS, con más novedades por llegar. Analizar el código fuente de Cloudflare OS es una excelente forma de comprender cómo el equipo responsable del *runtime* de Workers concibe el uso de esta plataforma.

El hecho de estar construido sobre Workers no implica que Cloudflare OS solo pueda ejecutarse en la infraestructura de Cloudflare. De hecho, [`workerd` —el *runtime* de Cloudflare Workers— es de código abierto](https://github.com/cloudflare/workerd), lo que permite ejecutar Cloudflare OS íntegramente sobre él en sus propios servidores.

Aunque existan conexiones a cuentas externas, los agentes y los *gadgets* no obtienen acceso automático a ellas.

En su lugar, debes «presentar» cada agente (o *gadget*) a los recursos específicos a los que deseas que tenga acceso. Por ejemplo, puedes presentar un repositorio de GitHub pegando su enlace, o bien haciendo clic en «añadir recurso» y seleccionándolo a través de la interfaz de usuario. Asimismo, un agente puede solicitar acceso a un recurso que considere necesario; tú podrás entonces conceder o denegar dicha solicitud.

Esto difiere de la mayoría de los entornos para agentes, en los que los servidores MCP se configuran de antemano, lo que pone a disposición del agente un acceso general a todos tus servicios en cada conversación. El enfoque basado en la concesión de permisos específicos garantiza que cada agente solo tenga acceso a lo que realmente necesita para realizar la tarea en cuestión.
