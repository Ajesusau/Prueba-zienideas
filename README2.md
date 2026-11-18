## Proceso de Análisis y Planteamiento de la Solución

1. **Comprensión del ecosistema:** En primer lugar, analicé el funcionamiento de **Microsoft Graph** para entender cómo actúa como puerta de enlace unificada hacia los datos del tenant de Microsoft 365 y qué alcance real tenía la prueba.
2. **Exploración de capacidades y permisos:** Revisé los endpoints y permisos de solo lectura disponibles para definir una base viable y segura, descartando cualquier acción de escritura o modificación sobre el entorno.
3. **Selección de indicadores clave (Seguridad + Negocio):** De entre todas las métricas posibles, decidí centrar el análisis en tres pilares con alto impacto directo para la empresa:
   * **Seguridad de acceso:** Detección de usuarios sin doble factor de autenticación (**MFA**) activo.
   * **Higiene de cuentas:** Identificación de cuentas con **inactividad** prolongada (más de 90 días sin iniciar sesión).
   * **Optimización de costes:** Cruce entre **licencias contratadas y asignadas** para detectar gasto innecesario en licencias sin uso.
4. **Diseño de la aplicación en Java:** Con estos requisitos definidos, planteé la arquitectura de un programa backend en Java que consume la API REST de Microsoft Graph, procesa los datos obtenidos y emite un informe final priorizando los puntos críticos tanto a nivel de ciberseguridad como de eficiencia económica.

---

### Nota sobre el alcance y gestión del tiempo
Debido a la restricción temporal de la prueba (2 horas) y a la amplitud de la documentación oficial de Microsoft, prioricé comprender a fondo el flujo de autenticación, los permisos necesarios y las métricas de mayor valor para el negocio, plasmando la solución en un **esquema técnico y funcional sólido (MVP)** antes que presentar código incompleto o sin validar.
