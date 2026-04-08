

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
4. Definición de modelo lógico con entidades y relaciones definidas (MER) 
5. Crear modelo físico con columnas y clave primaria (acorde al alcance definido por el Product Owner). 
