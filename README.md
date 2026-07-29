# Alrededor de los EE. UU. — Autenticación de Usuarios (Proyecto 18)

# Descripción del proyecto

Este proyecto añade funcionalidad de autenticación de usuarios (registro, inicio de sesión y cierre de sesión) a la aplicación "Alrededor de los EE. UU.", conectándola al backend externo proporcionado por TripleTen. Solo los usuarios autorizados pueden acceder al contenido principal de la aplicación.

# Funcionalidades implementadas

# Registro de usuarios (/signup):

formulario controlado con validación básica de email y contraseña.

# Inicio de sesión (/signin):

autenticación contra el backend, obtención y almacenamiento del token JWT.

# Cierre de sesión:

implementado en Header.jsx mediante la función signOut, que elimina el token (removeToken), navega a /signin y actualiza el estado isLoggedIn en el contexto global.

# Rutas protegidas:

el componente ProtectedRoute restringe el acceso a la ruta raíz / a usuarios autenticados, y redirige a usuarios ya autenticados fuera de /signup y /signin.

# Persistencia de sesión:

al recargar la página, se valida el token guardado contra el endpoint /users/me para mantener la sesión activa sin pedir credenciales de nuevo.

# Ventana modal InfoTooltip:

notifica al usuario si el registro fue exitoso o falló.

# Backend utilizado

Este sprint conecta la app a un backend externo proporcionado por TripleTen (en un sprint posterior se integrará con la API propia de Express/MongoDB desarrollada en proyectos anteriores):

URL base: https://se-register-api.en.tripleten-services.com/v1

# Endpoints usados:

# Endpoint Método Uso

/signup POST Registro de usuario nuevo
/signin POST Inicio de sesión, devuelve un token JWT
/users/me GET Valida el token y obtiene el email del usuario autenticado

Las peticiones a rutas protegidas incluyen el header:

Authorization: Bearer {token}

# Estructura relevante

src/
├── components/
│ ├── App.js
│ ├── Header/
│ │ └── Header.jsx # incluye el botón y lógica de cierre de sesión
│ ├── Login/
│ │ └── Login.jsx
│ ├── Register/
│ │ └── Register.jsx
│ ├── ProtectedRoute/
│ │ └── ProtectedRoute.jsx
│ └── InfoTooltip/
│ └── InfoTooltip.jsx
├── contexts/
│ └── CurrentUserContext.js
└── utils/
├── auth.js # register, authorize, checkToken
└── token.js # setToken, getToken, removeToken (localStorage)

# Tecnologías

-React
-React Router
-Context API (CurrentUserContext)
-Fetch API
-localStorage para persistencia del token

# Análisis de seguridad (Paso 6)

Se revisó el proyecto contra la lista de comprobación de seguridad del sprint. A continuación, el análisis de cada punto:

1. Encabezados de respuesta (helmet)

No aplica: esta configuración depende del backend, y en este proyecto se utiliza el backend externo de TripleTen, sobre el cual no se tiene control.

2. Validación de datos del usuario

Los formularios de Register y Login son componentes controlados que validan el formato del email y la contraseña antes de enviarlos al backend.

3. Almacenamiento del token (localStorage)

La aplicación guarda el token JWT en localStorage tras iniciar sesión, a través del módulo utils/token.js (setToken, getToken, removeToken). Es una solución simple y funcional, pero localStorage es accesible desde cualquier script de JavaScript que corra en la página, por lo que, si existiera una vulnerabilidad XSS, un atacante podría robar el token directamente.

La alternativa más segura serían las cookies httpOnly, que el navegador maneja automáticamente y que JavaScript no puede leer ni escribir. Sin embargo, esta solución debe implementarse desde el backend, y al usar el backend externo de TripleTen en este sprint, no es posible aplicar este cambio ahora. Quedaría como mejora a futuro si se conecta la app a un backend propio (como el desarrollado en los Proyectos 16 y 17).

4. Protección CSRF

La aplicación no depende de que el navegador envíe cookies automáticamente para autorizar solicitudes. El token se adjunta manualmente en cada petición mediante el header Authorization: Bearer {token} (implementado en utils/auth.js). Esto significa que un sitio malicioso no puede forzar una solicitud autorizada a la API sin que el propio código de la app construya ese header, por lo que el riesgo de CSRF en este flujo no aplica.

5. Protección contra fuerza bruta y DDoS (express-rate-limit)

No aplica: es una medida de backend, y este proyecto no controla el servidor de TripleTen.

6. Auditoría de dependencias (npm audit)

Se ejecutó npm audit sobre las dependencias del frontend. Se encontraron 6 vulnerabilidades (1 baja, 2 moderadas, 3 altas). Se corrigieron 4 de ellas (@babel/core, brace-expansion, postcss, vite) mediante npm audit fix, ya que su corrección era compatible sin cambios que rompieran el proyecto.

Quedó pendiente una vulnerabilidad moderada en react-router / react-router-dom, cuya corrección requiere npm audit fix --force e instalaría una versión con cambios incompatibles (breaking change). Se decidió posponer esta actualización para evaluarla con cuidado en otro momento, evitando arriesgar la estabilidad de las rutas de la aplicación ya funcionando (incluyendo ProtectedRoute).

7. Mejora futura: Content-Security-Policy (CSP)

Se podría implementar una política Content-Security-Policy para restringir los orígenes desde los que la aplicación carga scripts, imágenes y conexiones. Su implementación queda pendiente de un mapeo cuidadoso de todos los recursos externos utilizados por la app antes de aplicarla en producción, para evitar bloquear funcionalidad existente.

# Visita el proyecto en GitHub Pages

https://ericksj91.github.io/web_project_around_auth/

## Autor

Erick Jiménez
