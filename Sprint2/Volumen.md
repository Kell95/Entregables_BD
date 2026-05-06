


Tabla	Volumen estimado	Observación
app_user	10 – 50	Solo almacena los empleados que usan el sistema (analistas, supervisores, admin), no los clientes. Es una tabla pequeña y casi no cambia.
role_permission	20 – 100	Guarda la combinación de roles, recursos y acciones permitidas. Se crea una sola vez al hacer el despliegue y no cambia en producción.
token_blacklist	500 – 5.000	Guarda los JWT invalidados cuando alguien cierra sesión o cambia de rol. No crece indefinidamente porque los tokens expirados se borran automáticamente con una función en la base de datos.
authentication_log	50.000 – 500.000	Es una de las tablas que más crece. Registra cada intento de login, cierre de sesión y bloqueo de cuenta. Con pocos usuarios pero muchos accesos diarios, en un año puede acumular cientos de miles de filas.
audit_log	10.000 – 100.000+	Registra toda acción importante del sistema: crear un solicitante, ejecutar una evaluación, registrar una decisión. Es inmutable (no se puede borrar) y debe conservarse mínimo 5 años por requisito regulatorio.
applicant	1.000 – 10.000	La tabla principal de clientes. Crece de forma lineal y predecible según la cantidad de solicitantes que ingresen al sistema.
applicant_edit_audit	500 – 5.000	Registra cada campo que se modifica individualmente, no el registro completo. Si un analista cambia el teléfono y el ingreso de un solicitante, se guardan dos filas. Por eso puede crecer más rápido de lo que parece.
financial_data	1.000 – 15.000	Cada vez que se actualizan los datos financieros de un solicitante, no se sobreescribe el registro anterior sino que se crea una versión nueva. Por eso hay más filas que solicitantes.
scoring_variable	10 – 50	Catálogo de variables del modelo (antigüedad laboral, nivel de deuda, etc.). Cambia muy poco. Las variables que se desactivan se conservan igual para no perder el historial.
variable_range	50 – 200	Define los rangos de puntaje para variables numéricas. Por ejemplo: antigüedad 0–6 meses = 10 puntos, 6–24 meses = 40 puntos. Son pocos registros por variable.
variable_category	20 – 100	Similar a la anterior pero para variables categóricas, como el tipo de empleo.
scoring_model	5 – 20	Guarda las versiones del modelo de scoring. Solo una puede estar activa a la vez. Es una tabla pequeña de configuración.
model_variable	50 – 500	Relaciona cada versión del modelo con sus variables y pesos. Incluye una "foto" de los rangos en el momento en que se activó el modelo, para poder reproducir evaluaciones históricas.
knockout_rule	10 – 50	Las reglas de rechazo automático. Pocas reglas, cambian raramente.
evaluation	2.000 – 20.000	La tabla central del negocio. Cada evaluación es inmutable: una vez registrada no se puede modificar ni eliminar.
evaluation_detail	20.000 – 200.000	La más grande del núcleo transaccional. Por cada evaluación se guarda una fila por cada variable evaluada (aproximadamente 8). Eso significa que tiene entre 8 y 10 veces más filas que la tabla evaluation.
evaluation_knockout	10.000 – 100.000	Registra el resultado de cada regla knockout en cada evaluación, tanto las que se activaron como las que no. Con 5 reglas por evaluación, su volumen es comparable al de audit_log. Este es un punto que se subestima fácilmente si solo se cuentan las reglas que dispararon.
credit_decision	2.000 – 20.000	Una decisión por evaluación (relación 1 a 1). No todas las evaluaciones tienen decisión registrada de inmediato.
simulation_scenario	50 – 500	Escenarios de prueba guardados por el administrador del motor. No generan evaluaciones reales ni afectan estadísticas.
<img width="877" height="2247" alt="image" src="https://github.com/user-attachments/assets/04a6954f-21a9-407f-b311-b909188a8966" />
