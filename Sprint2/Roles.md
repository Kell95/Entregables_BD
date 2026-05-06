
# 6. Definición de roles y esquema de seguridad para la aplicación en la base de datos relacional.


**Se definieron los siguientes roles dentro del sistema:**

Se definieron cuatro roles dentro del sistema, almacenados con restricción CHECK en la tabla app_user:

**ADMIN:** Tiene acceso total al sistema. Es el único rol que puede crear y gestionar cuentas de usuario, configurar permisos y supervisar todas las operaciones. No se puede eliminar el último administrador del sistema.

**RISK_MANAGER:** Administrador del motor de riesgos. Gestiona la configuración del modelo de scoring: variables, pesos, rangos y reglas de rechazo automático (knockout). Accede a reportes estratégicos pero no puede modificar datos de solicitantes ni gestionar usuarios.

**CREDIT_SUPERVISOR:** Supervisor de crédito. Supervisa las evaluaciones realizadas por los analistas, accede a reportes operativos y al log de auditoría. Puede crear evaluaciones y tiene responsabilidad sobre las decisiones escaladas, que debe resolver en un plazo máximo de 48 horas.

**ANALYST:** Analista de crédito. Es el usuario operativo del sistema: registra solicitantes, carga datos financieros, ejecuta evaluaciones de scoring y registra las decisiones crediticias. Solo puede consultar su propio historial de actividad en el log de auditoría.

**Esquema de seguridad en la base de datos**

El control de seguridad se implementa mediante las siguientes tablas:

- app_user: Almacena los usuarios del sistema, incluyendo credenciales hasheadas con bcrypt (factor de costo 12), estado de la cuenta, rol asignado y control de intentos de login fallidos. La cuenta se bloquea automáticamente tras cinco intentos fallidos consecutivos.

- role_permission: Define la matriz de permisos asociados a cada rol, indicando qué acciones puede realizar sobre cada recurso del sistema (CREATE, READ, UPDATE, DELETE). Esta tabla se siembra durante el despliegue inicial y no se modifica desde la interfaz, ya que los roles son predefinidos.

- token_blacklist: Permite invalidar tokens JWT, mejorando la seguridad en procesos como cierre de sesión o cambio de rol. Los tokens expirados se limpian automáticamente mediante la función cleanup_expired_tokens().
authentication_log: Registra cada evento de autenticación: logins exitosos, intentos fallidos, cierres de sesión y bloqueos de cuenta, incluyendo la dirección IP de origen.

- audit_log: Registra todas las acciones relevantes realizadas en el sistema (creación, modificación, evaluación, decisión), con el estado anterior y posterior de cada cambio. Es inmutable, no puede modificarse ni eliminarse, y su retención mínima es de cinco años por requerimiento regulatorio.

**Funcionamiento**

Cada usuario tiene exactamente un rol asignado. Al autenticarse, el sistema genera un JWT que contiene el identificador del usuario, su rol y la fecha de expiración. En cada petición al API, un middleware de autorización verifica que el rol del token tenga permiso sobre el recurso solicitado consultando la tabla role_permission. Si no tiene autorización, el sistema responde con HTTP 403. Los cambios de rol toman efecto de forma inmediata, invalidando el token activo del usuario.

Este enfoque permite controlar el acceso a la información, restringir operaciones según el tipo de usuario, mantener trazabilidad completa de las acciones realizadas y cumplir con requisitos regulatorios de auditoría.

- Restringir operaciones según el tipo de usuario
- Garantizar la seguridad de los datos
- Mantener trazabilidad de las acciones realizada
