# Auditoría Automatizada de Seguridad en Microsoft 365 (Microsoft Graph)

## 1. Objetivo y Enfoque de la Solución
El objetivo de este proyecto es diseñar una herramienta backend en **Java** capaz de conectarse de forma segura (solo lectura) a un tenant de Microsoft 365 mediante la API de **Microsoft Graph**. El sistema extrae metadatos críticos de identidad y licenciamiento, los evalúa bajo reglas de riesgo y genera un informe estructurado para facilitar la toma de decisiones del equipo de ciberseguridad.

---

## 2. Flujo de Trabajo de la Herramienta (Arquitectura)

El funcionamiento del sistema se divide en 4 fases secuenciales:

1. **Autenticación (OAuth 2.0 - Client Credentials):** 
   La aplicación Java utiliza las credenciales proporcionadas (`Tenant ID`, `Client ID` y `Client Secret`) para solicitar un *Access Token* al endpoint de identidad de Microsoft (`login.microsoftonline.com`). Este token permite realizar consultas sin intervención humana.
2. **Extracción de Datos (Consumo de API REST):** 
   Utilizando el cliente HTTP nativo de Java (`java.net.http.HttpClient`), se lanzan peticiones `GET` con permisos estrictos de solo lectura (`User.Read.All`, `AuditLog.Read.All`, `RoleManagement.Read.Directory`, `Organization.Read.All`) hacia los endpoints clave de Graph:
   * `/v1.0/users` (Listado general y cuentas de invitados `Guest`).
   * `/v1.0/users?$select=displayName,signInActivity` (Detección de inactividad).
   * `/v1.0/reports/authenticationMethods/userRegistrationDetails` (Estado del MFA).
   * `/v1.0/directoryRoles` (Identificación de administradores privilegiados).
   * `/v1.0/subscribedSkus` (Auditoría de licencias contratadas vs. asignadas).
3. **Motor de Reglas y Priorización (Lógica en Java):**
   Los JSON recibidos se mapean a objetos Java y pasan por un evaluador de reglas puras que asignan un nivel de severidad (**Crítico, Alto, Medio, Informativo**).
4. **Generación de Informe:**
   Se exporta un resumen ejecutivo en consola y un fichero JSON/CSV con los hallazgos ordenados por gravedad para su remediación.

---

## 3. Criterios de Evaluación y Severidad (Memoria de Decisiones)

### ¿Qué se considera crítico y por qué?
* **CRÍTICO – Administrador sin MFA:** Si un usuario tiene el rol *Global Administrator* y no tiene doble factor registrado (`isMfaRegistered == false`), el riesgo de compromiso total del tenant es inmediato.
* **ALTO – Usuario estándar sin MFA o Cuentas Inactivas (>90 días):** Las cuentas abandonadas son vectores habituales de ataque; si además tienen licencias asignadas, suponen un gasto innecesario.
* **MEDIO – Invitados (`Guest`) sin actividad reciente:** Usuarios externos que conservan acceso a recursos internos sin necesitarlo.
* **INFORMATIVO – Licencias sin asignar:** Optimización de costes (ej. licencias pagadas en `subscribedSkus` que no están vinculadas a ningún usuario activo).

### Prevención de Falsos Positivos
* **Cuentas de emergencia (*Break-glass*):** Las organizaciones suelen mantener 1 o 2 cuentas de administrador de respaldo excluidas de las políticas habituales de MFA para evitar bloqueos totales. El motor debe permitir configurar una lista blanca (*allowlist*) para no marcarlas como alerta roja errónea.
* **Cuentas de servicio:** Usuarios no humanos utilizados para automatizaciones que no inician sesión de forma interactiva.

### Privacidad y Seguridad de los Datos
* **Principio de mínimo privilegio:** La aplicación opera exclusivamente con permisos `.Read.All`, garantizando que no puede alterar configuraciones ni leer contenido privado (como correos o archivos de OneDrive).
* **Gestión de secretos:** Las credenciales nunca se escriben en el código fuente; se cargan mediante variables de entorno (`.env` excluido en `.gitignore`).
* **Anonimización:** En los informes exportados para análisis general, los identificadores personales (UPN/correos) se enmascaran parcialmente (ej. `j***@empresa.com`).

### Acción Recomendada del Analista ante Alertas Rojas
1. **Ante Admin sin MFA:** Contactar inmediatamente con el titular, forzar el registro de MFA desde Entra ID y revisar los logs de inicio de sesión de las últimas 48 horas para descartar accesos anómalos.
2. **Ante Cuenta Inactiva >90 días:** Bloquear el inicio de sesión (`accountEnabled = false`), revocar sesiones activas y retirar las licencias asignadas tras confirmar con Recursos Humanos.

---

## 4. Uso de Inteligencia Artificial y Validación Técnica
Se ha utilizado IA como apoyo para comprender la arquitectura de Microsoft Graph y estructurar las peticiones HTTP en Java, aplicando siempre validación manual sobre la documentación oficial:

* **Error detectado 1 (Licenciamiento de API):** Inicialmente la IA sugirió consultar directamente `signInActivity` y los reportes de MFA sin advertir restricciones. Al contrastar con la documentación de Microsoft Graph, se verificó que estos endpoints requieren que el tenant disponga de licencias **Entra ID P1/P2** (devolviendo error `403 Forbidden` en tenants básicos), por lo que el diseño contempla capturar este error de forma controlada y omitir esos bloques sin detener la auditoría.
* **Error detectado 2 (Permisos delegados vs. Aplicación):** La IA propuso inicialmente usar el permiso `User.Read` (que solo lee el perfil propio en sesiones interactivas). Se corrigió a `User.Read.All` de tipo *Application*, indispensable para que un backend en Java pueda auditar a todos los usuarios de la organización sin intervención humana.

---

## 5. Próximos Pasos (Si se dispusiera de más tiempo)
* Implementar la ejecución completa en Java utilizando Spring Boot o Maven con tests unitarios (JUnit) para validar cada regla de severidad.
* Añadir paginación (`@odata.nextLink`) para soportar tenants con más de 100 usuarios.