
# 6. Definición de roles y esquema de seguridad para la aplicación en la base de datos relacional.


**Se definieron los siguientes roles dentro del sistema:**

**ADMIN:**
Tiene acceso total al sistema. Puede gestionar usuarios, configurar modelos de scoring y supervisar todas las operaciones.

**ANALYST:**
Encargado de realizar evaluaciones crediticias. Puede consultar información de solicitantes y ejecutar procesos de scoring.

**SUPERVISOR:**
Responsable de revisar y aprobar decisiones crediticias, especialmente en casos que requieren validación adicional.

**AUDITOR:**
Tiene acceso de solo lectura a la información del sistema, con el fin de verificar procesos y garantizar cumplimiento.

**Esquema de seguridad en la base de datos**

**El control de seguridad se implementa mediante las siguientes tablas: **

**app_user:**
Almacena los usuarios del sistema, incluyendo credenciales, estado de la cuenta y el rol asignado.

**role_permission:**
Define los permisos asociados a cada rol, indicando qué acciones puede realizar sobre cada recurso (por ejemplo: READ, WRITE, DELETE).

**token_blacklist:**
Permite invalidar tokens de autenticación (JWT), mejorando la seguridad en procesos como cierre de sesión o bloqueo de usuarios.
 
**audit_log:**
Registra todas las acciones relevantes realizadas en el sistema, permitiendo trazabilidad y auditoría


**Funcionamiento**

Cada usuario tiene un rol asignado, y a partir de este rol se determinan sus permisos dentro del sistema.
Cuando un usuario intenta realizar una acción, el sistema valida si su rol tiene autorización sobre el recurso solicitado.

Este enfoque permite:

- Controlar el acceso a la información
- Restringir operaciones según el tipo de usuario
- Garantizar la seguridad de los datos
- Mantener trazabilidad de las acciones realizada
