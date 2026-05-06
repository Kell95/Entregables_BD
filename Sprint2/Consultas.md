
# 4.Crear las consultas identificadas o acciones del módulo para las HU correspondientes al Sprint 1 y 2 vs modelo físico.


**SPRINT 1**


Base del sistema

HU-018 — Autenticación de usuarios

Acciones del módulo:
  
Buscar usuario por email

login(email, password)

**Verificar si está bloqueado**
validateAccountStatus(user_id)

**Incrementar intentos fallidos
handleFailedLogin(user_id)

Bloquear tras 5 intentos
blockAccount(user_id)

Restablecer intentos, last_login
handleSuccessfulLogin(user_id)

Registrar intento de login
logAuthAttempt(...)

Invalidar token en blacklist
logout(jti)

Verificar token en lista negra
validateToken(jti)


Tablas:

- app_user
- authentication_log
- token_blacklist

 Consultas:

- Buscar usuario

```sql
SELECT * FROM app_user
WHERE email = :email
AND enabled = true;
```

- Verificar si token está en blacklist

```sql
SELECT * FROM token_blacklist
WHERE jti = :jti
  AND expires_at > NOW();

```

```sql
Bloquear cuenta (5 intentos)
UPDATE app_user
SET account_locked = true,
    lock_time = NOW()
      + INTERVAL '30 minutes',
    failed_login_attempts = 5
WHERE id = :user_id;
```


<img width="684" height="610" alt="image" src="https://github.com/user-attachments/assets/615c4b1c-5267-4fb0-b7e3-b5662d41a279" />


