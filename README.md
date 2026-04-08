

# Entregables_BD

Sprint 1

Kelly Julieth Arango Henao
Carlos Andrés Cordoba

Sistema: Motor de Scoring de Riesgo Crediticio

1. Definición de entidades a partir de las reglas de negocio del módulo
   
**Applicant (Solicitante)**
Persona que solicita un crédito dentro del sistema.

**FinancialData (Datos Financieros)**
Información financiera que describe la capacidad económica del solicitante.

**Evaluation (Evaluación)**
Proceso que calcula el nivel de riesgo crediticio de un solicitante.

**EvaluationDetail (Detalle de Evaluación)**
Conjunto de datos que explican cómo se calcula el puntaje de una evaluación.

**CreditDecision (Decisión de Crédito)**
Resultado que determina si un crédito es aprobado, rechazado o enviado a revisión.

**ScoringModel (Modelo de Scoring**)
Estructura que define cómo se calcula el riesgo crediticio.

**ScoringVariable (Variable de Scoring)**
Factor que influye en el cálculo del puntaje de riesgo.

**ModelVariable**
Relación que asigna un peso a cada variable dentro de un modelo de scoring.

**VariableRange**
Rangos que permiten evaluar variables numéricas dentro del modelo.

**VariableCategory**
Categorías que permiten evaluar variables cualitativas dentro del modelo.

**KnockoutRule (Regla de Exclusión)**
Condición que descalifica automáticamente a un solicitante.

**AppUser (Usuario del Sistema)**
Persona que accede y opera el sistema según un rol asignado.

**RolePermission**
Configuración que define qué acciones puede realizar cada rol en el sistema.

**AuditLog**
Rregistro que almacena las acciones realizadas dentro del sistema para auditoría.

**TokenBlacklist**
Conjunto de tokens que han sido invalidados para evitar su reutilización.

**AuthenticationLog**
Registro que documenta los eventos de autenticación de los usuarios.

<img width="4714" height="1740" alt="Diagrama en blanco (1)" src="https://github.com/user-attachments/assets/0f3f8d22-9ef7-4f53-86ee-598456c969d6" />

   
3. Principales consultas identificadas o preguntas a responder del módulo

**Consultas sobre solicitantes**
      
- ¿El solicitante ya existe en el sistema?
Permite validar que no se registren personas duplicadas.

SELECT id
FROM applicant
WHERE identification_hash = :hash;

- ¿Cuáles son los datos de un solicitante?
Se utiliza para consultar la información completa del solicitante.

SELECT *
FROM applicant
WHERE id = :id;

- ¿Qué solicitantes coinciden con una búsqueda?
Permite buscar por nombre o identificación, sin importar mayúsculas o minúsculas y con paginación.

SELECT *
FROM applicant
WHERE LOWER(name) LIKE LOWER('%:criterio%')
   OR identification_hash = :hash
ORDER BY name
LIMIT :limit OFFSET :offset;

 **Consultas sobre datos financieros**
 
- ¿Cuál es el historial financiero de un solicitante?
Permite ver todas las versiones registradas.

SELECT *
FROM financial_data
WHERE applicant_id = :id
ORDER BY version DESC;

- ¿Cuál es el último registro financiero del solicitante?
Se usa para realizar la evaluación más reciente.

SELECT *
FROM financial_data
WHERE applicant_id = :id
ORDER BY version DESC
LIMIT 1;

- ¿Qué solicitantes tienen alto nivel de endeudamiento?
Ayuda a identificar posibles riesgos.

SELECT applicant_id, deuda_total, ingresos_mensuales
FROM financial_data
WHERE (deuda_total / ingresos_mensuales) > 0.6;

**Consultas sobre evaluación de riesgo**

- ¿Qué evaluaciones se han realizado a un solicitante?
Permite ver el historial de evaluaciones.

SELECT *
FROM evaluation
WHERE applicant_id = :id
ORDER BY evaluated_at DESC;

- ¿Cómo se calculó el score de una evaluación?
Permite ver el detalle del cálculo.

SELECT *
FROM evaluation_detail
WHERE evaluation_id = :evaluationId;

- ¿Cómo se distribuyen los clientes por nivel de riesgo?
Se usa para reportes del negocio.

SELECT risk_level, COUNT(*) AS total
FROM evaluation
GROUP BY risk_level;

 **Consultas sobre decisiones de crédito**

- ¿Qué decisiones se han tomado?
Permite ver todas las decisiones registradas.

SELECT *
FROM credit_decision;

- ¿Cuántas solicitudes fueron aprobadas o rechazadas?
Sirve para análisis del negocio.

SELECT decision, COUNT(*) AS total
FROM credit_decision
GROUP BY decision;

** Consultas de seguridad**

- ¿Un usuario tiene permisos para realizar una acción?
Se valida el acceso según el rol.

SELECT allowed
FROM role_permission
WHERE role = :role
AND resource = :resource
AND action = :action;

- ¿El usuario está bloqueado?
Permite controlar intentos de acceso.

SELECT account_locked, failed_login_attempts
FROM app_user
WHERE email = :email;

**Consultas de auditoría**

 - ¿Qué acciones ha realizado un usuario?
Permite revisar su actividad.

SELECT *
FROM audit_log
WHERE actor = :usuario;

- ¿Qué acciones ocurrieron en un periodo de tiempo?
Permite hacer seguimiento por fechas.

SELECT *
FROM audit_log
WHERE created_at BETWEEN :fecha_desde AND :fecha_hasta;

- ¿Qué cambios se hicieron sobre una entidad?
Permite ver historial de modificaciones.

SELECT *
FROM audit_log
WHERE entity_type = 'applicant'
AND entity_id = :id;

- ¿Cuántos intentos fallidos de login han ocurrido?
Ayuda a detectar problemas de seguridad.

SELECT COUNT(*)
FROM audit_log
WHERE action = 'LOGIN_FALLIDO';


5. Definición de modelo lógico con entidades y relaciones definidas (MER)
   
<img width="14991" height="12079" alt="Diagrama en blanco (8)" src="https://github.com/user-attachments/assets/93e113f6-de9c-431b-b715-b077b3b05c6d" />


7. Crear modelo físico con columnas y clave primaria (acorde al alcance definido por el Product Owner). 


<img width="14991" height="11911" alt="Diagrama en blanco (8)" src="https://github.com/user-attachments/assets/fa4fa5dd-7346-408c-bf50-bdad4d32b41e" />

