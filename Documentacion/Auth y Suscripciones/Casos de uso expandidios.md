
![Diagrama1](Diagramas/Diagrama-Primera-descomposicion-Auth.png)

![Diagrama1](Diagramas/Diagrama-Primera-descomposicion-Auth.png)

## CU-USER-01: Ingresar Datos

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-01 |
| **Módulo** | Usuario / Autenticación |
| **Actor(es) principal(es)** | Usuario Final (Visitante) |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Alta |
| **Tipo** | Principal |
| **Precondiciones** | 1. El visitante ha accedido al formulario de registro de Quetzal TV.<br>2. El API Gateway está disponible y enruta hacia el microservicio de autenticación. |
| **Postcondiciones** | • **Éxito:** Los datos ingresados pasan al flujo de validación y el sistema mantiene la información en estado temporal (staging).<<br>• **Fallo:** Se descartan los datos temporales; el sistema solicita corrección. |
| **Flujo Principal** | 1. El visitante accede a la opción "Crear cuenta".<<br>2. El sistema presenta el formulario de registro: correo electrónico, nombre completo, país de residencia y contraseña.<br>3. El visitante ingresa los datos solicitados.<br>4. El sistema ejecuta «include» **CU-USER-02: Validar Datos**.<br>5. El sistema almacena los datos validados en estado temporal.<br>6. El sistema ejecuta «include» **CU-USER-03: Notificar Registro**.<br>7. El sistema muestra mensaje de confirmación: "Revisa tu correo para verificar tu cuenta". |
| **Flujos Alternativos** | **FA-1: Registro federado (OAuth 2.0)**<<br>1. El visitante selecciona "Continuar con Google/GitHub".<<br>2. El sistema redirige al Identity Provider (OAuth).<<br>3. El Identity Provider retorna código de autorización.<br>4. El sistema intercambia el código por token de identidad.<br>5. El sistema extrae: email, nombre y avatar.<br>6. El sistema salta al paso 4 del flujo principal (Validar Datos) usando la información del IdP.<br><br>**FA-2: Autocompletar país por IP**<<br>1. El sistema detecta la IP del visitante.<br>2. Sugiere el país de residencia en el desplegable.<br>3. El visitante confirma o modifica la sugerencia; continúa en el paso 3. |
| **Flujos de Excepción** | **FE-1: Campos obligatorios vacíos**<<br>1. El visitante intenta continuar sin completar campos obligatorios.<br>2. El sistema resalta los campos faltantes y muestra: "Todos los campos son obligatorios".<<br>3. El flujo retorna al paso 2.<br><br>**FE-2: Formato de correo inválido**<<br>1. El visitante ingresa un correo sin formato válido.<br>2. El sistema muestra: "Ingresa un correo electrónico válido".<<br>3. El flujo retorna al paso 2. |
| **Reglas de Negocio** | • El correo electrónico debe ser único en la plataforma.<br>• La contraseña debe tener mínimo 8 caracteres, 1 mayúscula, 1 minúscula, 1 número y 1 símbolo.<br>• El país de residencia determina la moneda local por defecto en suscripciones. |
| **Requerimientos asociados** | • RF-AUTH-01<br>• RF-AUTH-05 (OAuth)<br>• RNF-AUTH-01 (hash de credenciales) |

---

## CU-USER-02: Validar Datos

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-02 |
| **Módulo** | Usuario / Autenticación |
| **Actor(es) principal(es)** | Sistema (Microservicio Auth) |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Alta |
| **Tipo** | Secundario (<<include>>) |
| **Precondiciones** | 1. El visitante ha ingresado datos en el formulario de registro o login.<br>2. El microservicio de autenticación está operativo. |
| **Postcondiciones** | • **Éxito:** Los datos cumplen todas las reglas de negocio; el flujo principal continúa.<br>• **Fallo:** Se rechazan los datos; se notifica el motivo específico al usuario. |
| **Flujo Principal** | 1. El sistema recibe los datos ingresados por el actor principal o secundario.<br>2. El sistema verifica que el correo no exista previamente en la tabla de usuarios (consulta a DB).<<br>3. El sistema valida la fortaleza de la contraseña contra la política de seguridad.<br>4. El sistema verifica que el país seleccionado pertenezca a la lista de países operativos.<br>5. El sistema confirma la validación exitosa y retorna control al caso de uso invocador. |
| **Flujos Alternativos** | **FA-1: Validación en login (credenciales existentes)**<<br>1. El sistema recibe correo y contraseña de login.<br>2. Consulta el hash almacenado (bcrypt/Argon2).<<br>3. Compara la contraseña ingresada con el hash.<br>4. Si coinciden, retorna éxito; si no, lanza excepción de credenciales inválidas. |
| **Flujos de Excepción** | **FE-1: Correo ya registrado**<<br>1. La consulta a DB encuentra el correo activo.<br>2. El sistema retorna: "Este correo ya está asociado a una cuenta. ¿Deseas iniciar sesión?".<<br>3. El flujo se cancela.<br><br>**FE-2: Contraseña débil**<<br>1. La contraseña no cumple la política de complejidad.<br>2. El sistema muestra: "La contraseña debe contener al menos 8 caracteres, una mayúscula, una minúscula, un número y un símbolo".<<br>3. El flujo retorna al ingreso de datos.<br><br>**FE-3: Límite de intentos de validación**<<br>1. El visitante excede 5 intentos fallidos de validación en 15 minutos.<br>2. El sistema bloquea temporalmente la IP/cuenta por 15 minutos (RNF-AUTH-04). |
| **Reglas de Negocio** | • El correo electrónico es único e insensible a mayúsculas/minúsculas en la validación de duplicados.<br>• La contraseña nunca se valida en texto plano contra una lista de contraseñas comunes (Have I Been Pwned opcional).<<br>• La validación de existencia de correo debe responder en ≤ 100 ms (p95). |
| **Requerimientos asociados** | • RF-AUTH-01<br>• RNF-AUTH-01 (almacenamiento seguro)<br>• RNF-AUTH-04 (rate limiting) |

---

## CU-USER-03: Notificar Registro

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-03 |
| **Módulo** | Usuario / Notificaciones |
| **Actor(es) principal(es)** | Sistema (Microservicio Auth) |
| **Actor(es) secundario(s)** | Proveedor de Correo |
| **Prioridad** | Alta |
| **Tipo** | Secundario (<<include>>) |
| **Precondiciones** | 1. Los datos del usuario han sido validados exitosamente.<br>2. El microservicio de notificaciones o el adaptador SMTP está configurado. |
| **Postcondiciones** | • **Éxito:** El correo de confirmación/verificación ha sido encolado o enviado; se registra el intento de envío.<br>• **Fallo:** Se registra el fallo en logs; la cuenta queda en estado "pendiente de verificación" pero los datos persisten. |
| **Flujo Principal** | 1. El sistema genera un token de verificación único (UUID/JWT de un solo uso con TTL de 24 horas).<<br>2. El sistema construye el correo electrónico con enlace de verificación: `https://quetzal.tv/verify?token=<token>`.<<br>3. El sistema invoca al **Proveedor de Correo** mediante API interna (gRPC o HTTP) para el despacho.<br>4. El Proveedor de Correo acepta la petición y retorna código 202/200.<br>5. El sistema almacena el token de verificación asociado al usuario en DB con estado "pendiente". |
| **Flujos Alternativos** | **FA-1: Reenvío de notificación**<<br>1. El usuario solicita "Reenviar correo de confirmación" desde la pantalla de espera.<br>2. El sistema invalida el token anterior.<br>3. El sistema genera un nuevo token y repite el flujo principal desde el paso 2.<br><br>**FA-2: Notificación de recibo (post-compra)**<<br>1. Este mismo caso de uso puede ser invocado por **CU-SUBS-02-01** (Contratar plan) para enviar el recibo de compra. |
| **Flujos de Excepción** | **FE-1: Proveedor de Correo no disponible**<<br>1. La petición al Proveedor de Correo falla o excede timeout.<br>2. El sistema encola el mensaje en una cola de reintentos (si aplica arquitectura asíncrona) o registra el fallo.<br>3. El sistema muestra al usuario: "Tu cuenta fue creada. Reenviaremos el correo de confirmación en unos minutos".<<br><br>**FE-2: Dirección de correo rechazada (bounce)**<<br>1. El Proveedor de Correo reporta dirección inexistente.<br>2. El sistema marca la cuenta como "correo inválido" y solicita actualización de email en el próximo login. |
| **Reglas de Negocio** | • El token de verificación expira en 24 horas.<br>• El enlace de verificación debe ser de un solo uso; una vez consumido, se invalida.<br>• El correo debe incluir branding de Quetzal TV y opción de contacto a soporte. |
| **Requerimientos asociados** | • RF-AUTH-01 (registro)<br>• RF-SUBS (recibos)<br>• RNF-PORT-01 (configuración por .env del SMTP) |

---

![Diagrama1](Diagramas/Diagrama-Expandido-Sesion.png)

## CU-USER-04: Iniciar Sesión

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-04 |
| **Módulo** | Usuario / Autenticación |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Alta |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario posee una cuenta registrada y verificada.<br>2. El microservicio de autenticación y el API Gateway están operativos. |
| **Postcondiciones** | • **Éxito:** El usuario obtiene JWT válido, Session Cookie segura y acceso al panel de perfiles.<br>• **Fallo:** No se genera sesión; se incrementa contador de intentos fallidos. |
| **Flujo Principal** | 1. El usuario ingresa correo y contraseña en el formulario de login.<br>2. El sistema ejecuta «include» **CU-USER-02: Validar Datos** (validación de credenciales contra hash).<<br>3. El sistema genera un **Access Token** (JWT RS256, exp 15 min) con claims: `sub`, `email`, `roles`, `iat`, `exp`, `jti`.<<br>4. El sistema genera un **Refresh Token** (exp 7 días) y lo almacena de forma segura.<br>5. El sistema establece la **Session Cookie** segura (`HttpOnly`, `Secure`, `SameSite=Strict`, `Path=/`).<<br>6. El sistema registra el evento de login en la tabla de auditoría (IP, timestamp, user agent).<<br>7. El sistema redirige al usuario al selector de perfiles. |
| **Flujos Alternativos** | **FA-1: Inicio de sesión con OAuth 2.0**<<br>1. El usuario selecciona "Continuar con [Proveedor]".<<br>2. El sistema redirige al Authorization Endpoint del Identity Provider con `state` y `nonce`.<<br>3. El Identity Provider retorna código de autorización al callback de Quetzal TV.<br>4. El sistema valida el `state` para prevenir CSRF.<br>5. El sistema intercambia el código por `id_token` y valida la firma contra JWKS.<br>6. Si el usuario no existe, se crea cuenta vinculada; si existe, se vincula sesión.<br>7. Continúa en el paso 3 del flujo principal.<br><br>**FA-2: Recuerdame (Remember Me)**<<br>1. El usuario marca la opción "Mantener sesión iniciada".<<br>2. El sistema extiende la duración de la Session Cookie a 30 días.<br>3. El Refresh Token se extiende proporcionalmente. |
| **Flujos de Excepción** | **FE-1: Credenciales inválidas**<<br>1. La validación de datos falla (correo no existe o contraseña no coincide).<<br>2. El sistema incrementa el contador de intentos fallidos.<br>3. Si el contador < 5, muestra: "Correo o contraseña incorrectos".<<br>4. Si el contador ≥ 5, aplica bloqueo temporal de 15 minutos (RNF-AUTH-04).<<br><br>**FE-2: Cuenta no verificada**<<br>1. El usuario existe pero el estado es "pendiente de verificación".<<br>2. El sistema muestra: "Verifica tu correo antes de iniciar sesión" y ofrece reenvío de correo.<br><br>**FE-3: Cuenta cancelada/inactiva**<<br>1. El usuario existe pero su estado es `inactive` o `banned`.<<br>2. El sistema muestra: "Tu cuenta no está disponible. Contacta a soporte". |
| **Reglas de Negocio** | • El JWT access token expira en 15 minutos; el refresh token en 7 días.<br>• La Session Cookie debe incluir `HttpOnly`, `Secure` (en producción) y `SameSite=Strict`.<<br>• Cada login exitoso genera un `jti` único para trazabilidad.<br>• La comunicación service-to-service debe propagar el JWT en el header `Authorization: Bearer`. |
| **Requerimientos asociados** | • RF-AUTH-02<br>• RF-AUTH-03 (JWT service-to-service)<br>• RF-AUTH-04 (Session Cookies)<br>• RNF-AUTH-02 (RS256, expiraciones)<br>• RNF-AUTH-03 (flags de cookie)<br>• RNF-AUTH-04 (rate limiting) |

---

## CU-USER-05: Cerrar Sesión

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-05 |
| **Módulo** | Usuario / Autenticación |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Media |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario tiene una sesión activa (JWT válido y Session Cookie presente).<<br>2. El API Gateway ha validado la cookie/JWT en la petición. |
| **Postcondiciones** | • **Éxito:** El JWT queda invalidado (denylist), la cookie se destruye y la sesión del cliente finaliza.<br>• **Fallo:** La sesión del cliente puede quedar parcialmente activa; se registra anomalía. |
| **Flujo Principal** | 1. El usuario selecciona "Cerrar sesión" en la interfaz.<br>2. El API Gateway intercepta la petición y extrae el JWT de la cookie.<br>3. El sistema envía el `jti` del JWT a una lista de denegación (Redis/DB) con TTL igual al tiempo restante de expiración del token.<br>4. El sistema destruye la Session Cookie del cliente (set cookie con `Max-Age=0` y valores vacíos).<<br>5. El sistema registra el evento de logout en auditoría.<br>6. El sistema redirige al usuario a la página de login. |
| **Flujos Alternativos** | **FA-1: Cierre de sesión remota (todas las sesiones)**<<br>1. Desde el panel de administración de cuenta, el usuario selecciona "Cerrar sesión en todos los dispositivos".<<br>2. El sistema invalida todos los refresh tokens activos del usuario.<br>3. El sistema agrega todos los JWT activos a la denylist.<br>4. Se destruyen todas las cookies asociadas en los clientes en su próxima petición.<br><br>**FA-2: Cierre automático por expiración**<<br>1. El JWT access token expira tras 15 minutos de inactividad.<br>2. El sistema rechaza la petición con 401.<br>3. El cliente debe usar el refresh token para renovar; si también expiró, se fuerza logout implícito. |
| **Flujos de Excepción** | **FE-1: Token ya invalidado**<<br>1. El sistema detecta que el `jti` ya existe en la denylist.<br>2. El sistema destruye la cookie de todas formas y redirige a login.<br>3. Se registra como posible reutilización de token (alerta de seguridad opcional).<<br><br>**FE-2: Fallo de conexión a Redis (denylist)**<<br>1. El microservicio no puede contactar Redis para almacenar la invalidación.<br>2. El sistema recurre a invalidación por rotación de secret de firma (emergencia) o almacena en DB.<br>3. Se dispara alerta al equipo de operaciones. |
| **Reglas de Negocio** | • Un JWT invalidado debe rechazarse en todas las peticiones posteriores, incluso si aún no ha expirado técnicamente.<br>• La cookie debe eliminarse tanto en el cliente como en el registro del servidor.<br>• El cierre de sesión debe ser idempotente (múltiples clicks no generan error). |
| **Requerimientos asociados** | • RF-AUTH-06 (logout seguro)<br>• RNF-AUTH-02 (invalidación JWT)<br>• RF-AUTH-03 (propagación de identidad) |

---

## CU-USER-06: Verificar Cuenta

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-06 |
| **Módulo** | Usuario / Autenticación |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Alta |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario ha completado el registro (CU-USER-01).<<br>2. El usuario ha recibido el correo con el enlace de verificación (CU-USER-03).<<br>3. El token de verificación no ha expirado. |
| **Postcondiciones** | • **Éxito:** La cuenta cambia de estado "pendiente" a "activa"; el usuario puede iniciar sesión.<br>• **Fallo:** La cuenta permanece en estado "pendiente"; se ofrece reenvío de correo. |
| **Flujo Principal** | 1. El usuario accede al enlace de verificación desde su correo electrónico.<br>2. El API Gateway recibe la petición GET con el parámetro `token`.<<br>3. El microservicio de autenticación valida el token contra la — (existencia y no expiración).<<br>4. El sistema actualiza el estado de la cuenta a `active`.<<br>5. El sistema invalida el token de verificación (eliminación lógica o física).<<br>6. El sistema muestra pantalla de éxito: "Cuenta verificada. Ahora puedes iniciar sesión". |
| **Flujos Alternativos** | **FA-1: Verificación desde móvil con deep link**<<br>1. El usuario abre el enlace desde la app móvil.<br>2. El sistema intercepta el deep link y extrae el token.<br>3. El flujo continúa desde el paso 3. |
| **Flujos de Excepción** | **FE-1: Token expirado**<<br>1. El sistema detecta que el token excedió las 24 horas de vigencia.<br>2. El sistema muestra: "El enlace ha expirado. Solicita uno nuevo".<<br>3. El sistema ofrece botón para reejecutar CU-USER-03 (Notificar Registro).<<br><br>**FE-2: Token inválido o ya consumido**<<br>1. El token no existe en la — o ya fue marcado como usado.<br>2. El sistema muestra: "Enlace no válido. Verifica tu correo o contacta a soporte".<<br><br>**FE-3: Cuenta ya verificada**<<br>1. El token es válido pero la cuenta ya está en estado `active`.<<br>2. El sistema muestra: "Tu cuenta ya está verificada" y ofrece redirección a login. |
| **Reglas de Negocio** | • El token de verificación es de un solo uso; una vez consumido, se invalida permanentemente.<br>• La verificación es obligatoria para iniciar sesión por primera vez.<br>• El estado de la cuenta debe ser atómico (`pending` → `active`); no se permiten estados intermedios. |
| **Requerimientos asociados** | • RF-AUTH-01 (registro completo)<br>• RNF-AUTH-02 (tokens con TTL) |

---

![Diagrama1](Diagramas/Diagrama-Expandido-Administrar-cuenta.png)

## CU-USER-07: Administrar Historial

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-07 |
| **Módulo** | Usuario / Administración de Cuenta |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Media |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario ha iniciado sesión (CU-USER-04).<<br>2. El usuario accede al panel de "Seguridad y Privacidad" de su cuenta. |
| **Postcondiciones** | • **Éxito:** El usuario visualiza el historial de actividad de su cuenta.<br>• **Fallo:** Se muestra mensaje de error y no se expone información. |
| **Flujo Principal** | 1. El usuario accede a la opción "Historial de actividad" dentro del panel de administración.<br>2. El sistema consulta la tabla de auditoría (alimentada por triggers y logs de sesión).<<br>3. El sistema presenta la lista cronológica: inicios de sesión, cambios de contraseña, cambios de correo, contrataciones de plan, modificaciones de suscripción.<br>4. El usuario puede filtrar por tipo de evento y rango de fechas.<br>5. El usuario selecciona un evento para ver detalle (IP, dispositivo, timestamp). |
| **Flujos Alternativos** | **FA-1: Exportar historial**<<br>1. El usuario selecciona "Exportar".<<br>2. El sistema genera archivo CSV/JSON con los eventos filtrados.<br>3. El sistema inicia descarga del archivo. |
| **Flujos de Excepción** | **FE-1: Sin eventos registrados**<<br>1. La consulta retorna conjunto vacío.<br>2. El sistema muestra: "No hay actividad reciente en tu cuenta".<<br><br>**FE-2: Fallo de consulta a DB**<<br>1. La tabla de auditoría no responde.<br>2. El sistema muestra: "No fue posible cargar tu historial. Intenta más tarde". |
| **Reglas de Negocio** | • El historial debe mostrar los últimos 90 días por defecto; máximo 2 años disponibles.<br>• Los eventos de cambio de credenciales deben mostrar el timestamp exacto y la IP de origen.<br>• El usuario solo puede ver su propio historial (aislamiento por `sub` del JWT). |
| **Requerimientos asociados** | • RF-ADMIN-02 (panel de cuenta)<br>• RF-DB-02 (trigger de auditoría)<br>• RNF-AUTH-02 (trazabilidad JWT `jti`) |

---

## CU-USER-08: Administrar Preferencias

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-08 |
| **Módulo** | Usuario / Administración de Cuenta |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Media |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario ha iniciado sesión.<br>2. El usuario accede al panel de "Configuración de cuenta". |
| **Postcondiciones** | • **Éxito:** Las preferencias se actualizan y aplican inmediatamente a la cuenta.<br>• **Fallo:** Las preferencias previas se mantienen; se notifica el error. |
| **Flujo Principal** | 1. El usuario accede a "Preferencias de cuenta".<<br>2. El sistema muestra las opciones configurables: idioma de la plataforma, país de facturación, preferencias de notificación por correo (marketing, nuevos lanzamientos, recibos), moneda de visualización preferida.<br>3. El usuario modifica una o varias preferencias.<br>4. El usuario confirma los cambios.<br>5. El sistema valida que el país seleccionado esté en la lista de países operativos.<br>6. El sistema almacena las preferencias en la —.<br>7. El sistema confirma: "Preferencias actualizadas correctamente". |
| **Flujos Alternativos** | **FA-1: Cambio de país con impacto en moneda**<<br>1. El usuario cambia el país de residencia/facturación.<br>2. El sistema ejecuta «include» consulta a FX-Service para mostrar precios en la nueva moneda.<br>3. El sistema advierte: "Los precios de tu suscripción se mostrarán en [nueva moneda]". |
| **Flujos de Excepción** | **FE-1: País no operativo**<<br>1. El país seleccionado no está en la lista de mercados activos de Quetzal TV.<br>2. El sistema muestra: "Quetzal TV aún no está disponible en este país".<<br>3. El sistema descarta el cambio y mantiene el país anterior.<br><br>**FE-2: Fallo de actualización**<<br>1. La escritura en DB falla.<br>2. El sistema muestra: "No se pudieron guardar los cambios. Intenta nuevamente". |
| **Reglas de Negocio** | • El idioma por defecto se deriva del país seleccionado durante el registro, pero el usuario puede sobrescribirlo.<br>• Las preferencias de notificación marketing deben respetar el opt-in/opt-out (cumplimiento normativo).<<br>• El cambio de país puede requerir re-verificación de método de pago si hay suscripción activa. |
| **Requerimientos asociados** | • RF-ADMIN-01 (panel de administración)<br>• RNF-PORT-01 (configuración por entorno) |

---

## CU-USER-09: Eliminar Historial

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-09 |
| **Módulo** | Usuario / Administración de Cuenta |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Baja |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario ha iniciado sesión.<br>2. El usuario tiene al menos un evento en su historial de actividad (CU-USER-07). |
| **Postcondiciones** | • **Éxito:** Los eventos seleccionados se eliminan de la vista del usuario (soft delete o anonimización según política).<<br>• **Fallo:** El historial permanece intacto. |
| **Flujo Principal** | 1. El usuario accede a "Historial de actividad" (CU-USER-07).<<br>2. El usuario selecciona uno o varios eventos del historial.<br>3. El usuario selecciona "Eliminar seleccionados".<<br>4. El sistema solicita confirmación: "¿Estás seguro? Esta acción no se puede deshacer".<<br>5. El usuario confirma.<br>6. El sistema marca los registros como `deleted_by_user = true` (soft delete para cumplimiento normativo).<<br>7. El sistema refresca la vista y muestra: "Historial eliminado". |
| **Flujos Alternativos** | **FA-1: Eliminar todo el historial**<<br>1. El usuario selecciona "Eliminar todo mi historial de actividad".<<br>2. El sistema solicita re-autenticación (CU-USER-04 con contraseña actual).<<br>3. Al confirmar, se marcan todos los eventos del usuario como eliminados por solicitud del titular. |
| **Flujos de Excepción** | **FE-1: Eventos de auditoría obligatorios**<<br>1. El usuario intenta eliminar eventos que son de cumplimiento legal (cambios de credenciales, compras).<<br>2. El sistema muestra: "Algunos eventos no pueden eliminarse por requerimientos de auditoría".<<br>3. El sistema elimina solo los eventos permitidos y notifica cuáles fueron retenidos.<br><br>**FE-2: Fallo de actualización masiva**<<br>1. La transacción de soft delete falla a mitad de proceso.<br>2. El sistema realiza rollback total.<br>3. Se muestra: "No se pudo completar la eliminación. Intenta nuevamente". |
| **Reglas de Negocio** | • Los eventos de compra y cambio de credenciales deben conservarse en backend por 5 años para auditoría, aunque el usuario los elimine de su vista (anonimización parcial).<<br>• La eliminación requiere confirmación explícita para prevenir borrado accidental.<br>• La acción de eliminar todo requiere re-autenticación por seguridad. |
| **Requerimientos asociados** | • RF-ADMIN-02 (gestión de cuenta)<br>• RF-DB-02 (auditoría y retención)<br>• RNF-AUTH-01 (protección de datos) |

---

![Diagrama1](Diagramas/Diagrama-Expandido-Perfil.png)

## CU-USER-10: Agregar Perfil

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-10 |
| **Módulo** | Usuario / Perfiles |
| **Actor(es) principal(es)** | Usuario Final (Administrador de la cuenta) |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Alta |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario ha iniciado sesión (CU-USER-04).<<br>2. La cuenta tiene menos de 5 perfiles activos.<br>3. El usuario se encuentra en la pantalla de selección/gestión de perfiles. |
| **Postcondiciones** | • **Éxito:** El nuevo perfil queda creado, asociado a la cuenta y disponible para selección.<br>• **Fallo:** No se crea el perfil; se mantiene el estado anterior. |
| **Flujo Principal** | 1. El usuario selecciona "Agregar perfil".<<br>2. El sistema presenta el formulario: nombre del perfil, avatar/imagen (opcional), clasificación por edades (todos los públicos, 7+, 13+, 18+).<<br>3. El usuario ingresa la información.<br>4. El sistema ejecuta «include» **CU-USER-12: Validar Perfil**.<br>5. El sistema almacena el perfil en la — con `account_id` y `profile_id` único.<br>6. El sistema muestra el nuevo perfil en la grilla de selección.<br>7. El sistema ofrece seleccionar el perfil recién creado para iniciar navegación. |
| **Flujos Alternativos** | **FA-1: Avatar por defecto**<<br>1. El usuario no selecciona avatar.<br>2. El sistema asigna un avatar generado automáticamente basado en las iniciales del nombre o un ícono aleatorio de la biblioteca.<br><br>**FA-2: Perfil infantil (restricciones)**<<br>1. El usuario selecciona clasificación "7+" o perfil infantil.<br>2. El sistema activa automáticamente controles parentales: bloqueo de contenido +18, desactivación de calificaciones públicas.<br>3. El flujo continúa en el paso 4. |
| **Flujos de Excepción** | **FE-1: Límite de 5 perfiles alcanzado**<<br>1. La validación detecta que la cuenta ya posee 5 perfiles activos.<br>2. El sistema muestra: "Has alcanzado el límite máximo de 5 perfiles. Elimina uno para continuar".<<br>3. Se deshabilita el botón "Agregar perfil".<<br><br>**FE-2: Nombre duplicado dentro de la cuenta**<<br>1. Ya existe un perfil con el mismo nombre en la cuenta.<br>2. El sistema sugiere: "Ya existe un perfil con este nombre. Usa un nombre diferente".<<br><br>**FE-3: Fallo de almacenamiento**<<br>1. La — no responde durante la inserción.<br>2. El sistema muestra: "No se pudo crear el perfil. Intenta nuevamente". |
| **Reglas de Negocio** | • Máximo 5 perfiles por cuenta; el límite es rígido e innegociable.<br>• El nombre del perfil debe tener entre 1 y 50 caracteres alfanuméricos.<br>• Cada perfil debe tener un identificador único (`profile_id`) aislado del `user_id` de la cuenta.<br>• El perfil principal (owner) se crea automáticamente al registrar la cuenta y no cuenta para el límite de 5 (o sí, según diseño del grupo; aquí asumo que los 5 incluyen el principal). |
| **Requerimientos asociados** | • RF-PROF-01 (máximo 5 perfiles)<br>• RF-PROF-02 (aislamiento)<br>• RF-PROF-03 (nombre, avatar, clasificación) |

---

## CU-USER-11: Eliminar Perfil

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-11 |
| **Módulo** | Usuario / Perfiles |
| **Actor(es) principal(es)** | Usuario Final (Administrador de la cuenta) |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Media |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario ha iniciado sesión.<br>2. La cuenta tiene al menos 2 perfiles (uno principal + al menos uno secundario).<<br>3. El usuario se encuentra en la gestión de perfiles. |
| **Postcondiciones** | • **Éxito:** El perfil secundario queda eliminado junto con sus datos asociados (historial, preferencias).<<br>• **Fallo:** El perfil permanece activo. |
| **Flujo Principal** | 1. El usuario selecciona el perfil secundario que desea eliminar.<br>2. El usuario selecciona la opción "Eliminar perfil".<<br>3. El sistema valida que el perfil no sea el perfil principal (owner).<<br>4. El sistema muestra advertencia: "Se eliminarán todas las preferencias y progreso asociados a [Nombre del perfil]. ¿Continuar?".<<br>5. El usuario confirma.<br>6. El sistema ejecuta eliminación en cascada: perfil → preferencias → historial de reproducción del perfil (delegado al microservicio correspondiente vía gRPC si aplica).<<br>7. El sistema actualiza el contador de perfiles de la cuenta.<br>8. El sistema redirige a la grilla de perfiles actualizada. |
| **Flujos Alternativos** | **FA-1: Transferir datos antes de eliminar**<<br>1. El sistema ofrece opción "Transferir historial a otro perfil" antes de confirmar.<br>2. El usuario selecciona perfil destino.<br>3. El sistema migra los datos y luego ejecuta la eliminación.<br><br>**FA-2: Eliminación desde panel admin (autogestión)**<<br>1. El usuario accede a "Administrar perfiles" dentro del panel de cuenta.<br>2. El flujo continúa desde el paso 1. |
| **Flujos de Excepción** | **FE-1: Perfil principal protegido**<<br>1. El usuario intenta eliminar el perfil principal (owner).<<br>2. El sistema muestra: "No puedes eliminar el perfil principal. Para eliminarlo, debes cancelar tu cuenta".<<br>3. Se cancela la operación.<br><br>**FE-2: Perfil con sesión activa**<<br>1. El perfil seleccionado tiene una sesión de reproducción activa en otro dispositivo.<br>2. El sistema fuerza el cierre de esa sesión y luego procede a eliminar.<br><br>**FE-3: Fallo de eliminación en cascada**<<br>1. La transacción de eliminación falla a mitad de proceso.<br>2. El sistema realiza rollback total.<br>3. Se muestra: "No se pudo eliminar el perfil. Intenta nuevamente". |
| **Reglas de Negocio** | • El perfil principal (owner) es indeleble mientras la cuenta exista.<br>• La eliminación de perfil debe ser atómica: o se elimina todo el árbol de datos del perfil, o no se elimina nada.<br>• Se debe conservar un registro de auditoría de la eliminación (quién, cuándo, qué perfil) por 1 año. |
| **Requerimientos asociados** | • RF-PROF-01 (límite 5)<br>• RF-PROF-04 (eliminar perfil secundario)<br>• RF-DB-02 (auditoría) |

---

## CU-USER-12: Validar Perfil

| Campo | Detalle |
|---|---|
| **Código** | CU-USER-12 |
| **Módulo** | Usuario / Perfiles |
| **Actor(es) principal(es)** | Sistema (Microservicio Auth/Perfiles) |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Alta |
| **Tipo** | Secundario (<<include>>) |
| **Precondiciones** | 1. El usuario ha solicitado crear (CU-USER-10) o editar un perfil.<br>2. El microservicio de perfiles está operativo. |
| **Postcondiciones** | • **Éxito:** Los datos del perfil cumplen todas las reglas de negocio; el flujo invocador continúa.<br>• **Fallo:** Se rechaza la operación; se notifican los errores específicos. |
| **Flujo Principal** | 1. El sistema recibe los datos del perfil: nombre, avatar, clasificación por edades.<br>2. El sistema verifica que la cuenta no exceda el límite de 5 perfiles activos (consulta a DB).<<br>3. El sistema valida que el nombre tenga entre 1 y 50 caracteres y no contenga palabras reservadas/ofensivas.<br>4. El sistema valida que el avatar, si se proporcionó, sea una imagen válida (formatos: JPG, PNG; tamaño máx. 2 MB).<<br>5. El sistema valida que la clasificación por edades sea un valor permitido del enum: `all`, `7+`, `13+`, `18+`.<<br>6. El sistema confirma validación exitosa y retorna control al caso de uso invocador. |
| **Flujos Alternativos** | **FA-1: Validación en edición de perfil existente**<<br>1. El sistema detecta que es una edición, no creación.<br>2. Omite la validación de límite de 5 perfiles (paso 2).<<br>3. Valida que el `profile_id` pertenezca a la cuenta del usuario autenticado.<br>4. Continúa desde el paso 3. |
| **Flujos de Excepción** | **FE-1: Límite de perfiles excedido**<<br>1. La cuenta ya tiene 5 perfiles.<br>2. El sistema retorna error: "Límite de perfiles alcanzado".<<br>3. El flujo invocador (CU-USER-10) muestra el mensaje al usuario.<br><br>**FE-2: Nombre inválido**<<br>1. El nombre está vacío, excede 50 caracteres o contiene términos prohibidos.<br>2. El sistema retorna: "El nombre del perfil no es válido".<<br><br>**FE-3: Formato de avatar no soportado**<<br>1. El archivo no es JPG/PNG o excede 2 MB.<br>2. El sistema retorna: "Selecciona una imagen JPG o PNG de máximo 2 MB". |
| **Reglas de Negocio** | • El límite de 5 perfiles es rígido por cuenta.<br>• Los nombres de perfil deben ser únicos dentro de una misma cuenta.<br>• La clasificación por edades determina los filtros de contenido aplicados al perfil.<br>• La validación debe completarse en ≤ 200 ms para no degradar la experiencia. |
| **Requerimientos asociados** | • RF-PROF-01 (máximo 5)<br>• RF-PROF-03 (clasificación, avatar)<br>• RNF-PERF-01 (tiempo de respuesta) |



























![Diagrama1](Diagramas/Diagrama-Primera-descomposicion-Suscripciones.png)


![Diagrama1](Diagramas/Diagrama-Primera-descomposicion-Suscripciones.png)


---

## CU-SUBS-01: Visualizar Planes

| Campo | Detalle |
|---|---|
| **Código** | CU-SUBS-01 |
| **Módulo** | Planes y Suscripciones |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | Proveedor Fx (vía FX-Service con Redis Cache) |
| **Prioridad** | Alta |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario ha iniciado sesión (CU-USER-04) o navega como visitante con acceso público a la landing de planes.<br>2. El microservicio de suscripciones está operativo y registrado en el API Gateway.<br>3. El FX-Service tiene al menos una tasa de cambio cacheada o puede consultar al proveedor externo. |
| **Postcondiciones** | • **Éxito:** El usuario visualiza los planes disponibles con precios en su moneda local y características diferenciadas.<br>• **Fallo:** Se muestra mensaje de error; no se expone información de precios inconsistente. |
| **Flujo Principal** | 1. El usuario accede a la sección "Planes y Precios" desde el menú principal o el panel de cuenta.<br>2. El sistema identifica el país/moneda preferida del usuario (desde preferencias de cuenta o detección por IP).<br>3. El sistema ejecuta «include» **CU-SUBS-03: Mostrar Planes**.<br>4. El sistema ejecuta «include» **CU-SUBS-04: Seleccionar Suscripción** (presenta la interfaz de selección junto a la visualización).<br>5. El sistema consulta el FX-Service para obtener el tipo de cambio USD → moneda local (cache Redis, TTL 1 hora).<br>6. El sistema renderiza las tarjetas de planes: Básico, Estándar y Premium, mostrando nombre, precio local, resolución máxima, número de dispositivos simultáneos y funciones incluidas.<br>7. El usuario visualiza la comparativa completa. |
| **Flujos Alternativos** | **FA-1: Visualización sin sesión iniciada**<br>1. El visitante accede a "/planes" sin autenticación.<br>2. El sistema detecta ausencia de JWT/cookie.<br>3. El sistema usa moneda por defecto (USD) o detecta país por IP.<br>4. El flujo continúa desde el paso 3, pero el botón de "Contratar" redirige a registro (CU-USER-01).<br><br>**FA-2: Usuario con suscripción activa**<br>1. El sistema detecta que el usuario ya tiene un plan activo.<br>2. Destaca el plan actual con etiqueta "Plan actual" y muestra opciones de upgrade/downgrade. |
| **Flujos de Excepción** | **FE-1: FX-Service no disponible**<br>1. El FX-Service no responde y no hay cache válido en Redis.<br>2. El sistema muestra precios en USD con indicador: "Precios en dólares. Actualizaremos tu moneda local en breve".<br>3. Se registra el fallo para monitoreo.<br><br>**FE-2: No hay planes configurados**<br>1. La — de suscripciones retorna conjunto vacío.<br>2. El sistema muestra: "No hay planes disponibles en este momento".<br>3. Se dispara alerta al equipo de operaciones.<br><br>**FE-3: Timeout en consulta a Redis**<br>1. El cache de Redis no responde.<br>2. El sistema consulta directamente al FX-Service o muestra USD como fallback.<br>3. Se registra degradación de servicio. |
| **Reglas de Negocio** | • Los planes siempre se muestran en el orden: Básico → Estándar → Premium.<br>• El precio en moneda local se calcula como: `precio_usd * tasa_cambio` y se redondea a 2 decimales.<br>• Si el cache de la tasa de cambio tiene > 1 hora, se debe refrescar en background.<br>• El plan Básico debe indicar explícitamente si incluye publicidad. |
| **Requerimientos asociados** | • RF-SUBS-01 (visualización de planes)<br>• RF-SUBS-03 (consulta FX-Service)<br>• RNF-PERF-02 (respuesta ≤ 200 ms con cache) |

---

## CU-SUBS-02: Administrar Credenciales

| Campo | Detalle |
|---|---|
| **Código** | CU-SUBS-02 |
| **Módulo** | Planes y Suscripciones / Administración de Cuenta |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | Proveedor de Pasarela de Pago (externo) |
| **Prioridad** | Media |
| **Tipo** | Principal (<<extend>> de CU-SUBS-01) |
| **Precondiciones** | 1. El usuario está visualizando los planes (CU-SUBS-01) o se encuentra en el panel de administración de cuenta.<br>2. El usuario tiene una sesión activa válida.<br>3. El sistema requiere credenciales de pago actualizadas antes de permitir una contratación o renovación. |
| **Postcondiciones** | • **Éxito:** Las credenciales de pago (método de pago, facturación) quedan actualizadas y validadas por la pasarela.<br>• **Fallo:** Las credenciales previas se mantienen; se notifica el error sin almacenar datos sensibles inválidos. |
| **Flujo Principal** | 1. Desde la vista de planes, el usuario selecciona "Actualizar método de pago" (punto de extensión desde CU-SUBS-01).<br>2. El sistema presenta el formulario: nombre del titular, número de tarjeta/token, fecha de expiración, CVV, dirección de facturación.<br>3. El usuario ingresa o modifica los datos.<br>4. El sistema valida el formato de los datos (Luhn para tarjeta, fecha futura).<br>5. El sistema envía los datos de forma segura (TLS 1.3) a la pasarela de pago para tokenización.<br>6. La pasarela retorna un token de método de pago (nunca se almacena PAN completo en Quetzal TV).<br>7. El sistema almacena el token y los últimos 4 dígitos de la tarjeta para referencia.<br>8. El sistema confirma: "Método de pago actualizado correctamente". |
| **Flujos Alternativos** | **FA-1: Agregar método de pago alternativo**<br>1. El usuario selecciona "Agregar otro método de pago".<br>2. El sistema permite registrar un segundo método y marcar uno como predeterminado.<br>3. El flujo continúa desde el paso 4.<br><br>**FA-2: Eliminar método de pago**<br>1. El usuario selecciona eliminar un método existente.<br>2. El sistema valida que quede al menos un método activo si hay suscripción vigente.<br>3. Si es válido, elimina el token de la pasarela y de la —. |
| **Flujos de Excepción** | **FE-1: Tarjeta rechazada por pasarela**<br>1. La pasarela retorna código de rechazo (fondos insuficientes, tarjeta bloqueada, etc.).<br>2. El sistema muestra: "No pudimos validar tu método de pago. Verifica los datos o intenta con otro".<br>3. No se almacena ningún token; se descartan los datos sensibles.<br><br>**FE-2: Violación de PCI-DSS (datos sensibles detectados)**<br>1. El sistema detecta que se intentó almacenar CVV o PAN completo en logs/DB.<br>2. El sistema aborta la operación, borra trazas y registra alerta de seguridad crítica.<br><br>**FE-3: Sesión expirada durante el proceso**<br>1. El JWT expira mientras el usuario edita el formulario.<br>2. Al enviar, el API Gateway retorna 401.<br>3. El sistema guarda el progreso en localStorage (opcional) y redirige a re-autenticación. |
| **Reglas de Negocio** | • Quetzal TV nunca almacena el PAN completo ni el CVV; solo tokens de la pasarela y últimos 4 dígitos.<br>• Se debe soportar mínimo 2 métodos de pago por cuenta (primario y secundario).<br>• La dirección de facturación debe coincidir con el país de la cuenta para emitir recibos fiscales correctos.<br>• La tokenización debe cumplir PCI-DSS SAQ-A. |
| **Requerimientos asociados** | • RF-ADMIN-01 (panel de administración de cuenta)<br>• RF-ADMIN-02 (actualizar credenciales)<br>• RNF-AUTH-01 (seguridad de datos sensibles)<br>• RNF-PORT-01 (.env para credenciales de pasarela) |

---

## CU-SUBS-03: Mostrar Planes

| Campo | Detalle |
|---|---|
| **Código** | CU-SUBS-03 |
| **Módulo** | Planes y Suscripciones |
| **Actor(es) principal(es)** | Sistema (Microservicio de Suscripciones) |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Alta |
| **Tipo** | Secundario (<<include>> de CU-SUBS-01) |
| **Precondiciones** | 1. El usuario ha solicitado visualizar planes (CU-SUBS-01).<br>2. El microservicio de suscripciones tiene conectividad con la — de planes. |
| **Postcondiciones** | • **Éxito:** El sistema obtiene y estructura la información cruda de los planes para su renderizado.<br>• **Fallo:** Se retorna error al caso invocador; no se muestran datos parciales. |
| **Flujo Principal** | 1. El sistema recibe la petición de visualización desde CU-SUBS-01.<br>2. El sistema consulta la —: tabla `subscription_plans` filtrando por `status = 'active'`.<br>3. El sistema recupera para cada plan: `plan_id`, `name`, `price_usd`, `max_resolution`, `max_devices`, `features_json`, `advertising_included`.<br>4. El sistema ordena los resultados por campo `display_order` (Básico=1, Estándar=2, Premium=3).<br>5. El sistema enriquece los datos con metadatos de disponibilidad por país del usuario.<br>6. El sistema retorna la estructura de datos al caso invocador (CU-SUBS-01) para su renderizado en la UI. |
| **Flujos Alternativos** | **FA-1: Planes con promoción activa**<br>1. El sistema detecta que existe una promoción vigente para el país del usuario.<br>2. Recupera el `discount_percentage` y el `promo_end_date`.<br>3. Incluye estos campos en la estructura retornada para que la UI muestre precio tachado y precio promocional. |
| **Flujos de Excepción** | **FE-1: — no disponible**<br>1. La conexión a la — del microservicio de suscripciones falla.<br>2. El sistema retorna error 503 al caso invocador.<br>3. CU-SUBS-01 muestra mensaje fallback: "Información de planes no disponible temporalmente".<br><br>**FE-2: Esquema de datos inconsistente**<br>1. El `features_json` de un plan tiene formato inválido.<br>2. El sistema descarta ese plan del listado y registra warning.<br>3. Continúa con los planes válidos (degradación graceful). |
| **Reglas de Negocio** | • Solo se muestran planes con `status = 'active'` y `available_in_countries` que incluya el país del usuario.<br>• El orden de visualización es inmutable: Básico primero, Estándar segundo, Premium tercero.<br>• Los precios base siempre se almacenan en USD en la —; la conversión se hace en runtime.<br>• La consulta a DB debe usar un índice por `status` + `display_order` para rendimiento. |
| **Requerimientos asociados** | • RF-SUBS-01 (visualización de planes)<br>• RNF-PERF-02 (respuesta con cache/DB)<br>• RNF-DISP-01 (disponibilidad del servicio) |

---

## CU-SUBS-04: Seleccionar Suscripción

| Campo | Detalle |
|---|---|
| **Código** | CU-SUBS-04 |
| **Módulo** | Planes y Suscripciones |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | — |
| **Prioridad** | Alta |
| **Tipo** | Secundario (<<include>> de CU-SUBS-01) |
| **Precondiciones** | 1. El usuario está visualizando los planes (CU-SUBS-01) y los datos han sido mostrados exitosamente.<br>2. El usuario tiene una sesión activa (para contratar) o navega como visitante (solo puede pre-seleccionar). |
| **Postcondiciones** | • **Éxito:** El plan queda pre-seleccionado o marcado para contratación; el flujo avanza al checkout.<br>• **Fallo:** No se registra selección; el usuario permanece en la vista de planes. |
| **Flujo Principal** | 1. El usuario visualiza las tarjetas de planes renderizadas por CU-SUBS-01.<br>2. El usuario selecciona uno de los planes (clic en "Elegir plan" o similar).<br>3. El sistema valida que el plan seleccionado esté activo y disponible en el país del usuario.<br>4. El sistema almacena la selección en el estado de la sesión o carrito temporal.<br>5. Si el usuario está autenticado, el sistema redirige al flujo de contratación (checkout).<br>6. Si el usuario es visitante, el sistema redirige a registro/login (CU-USER-01) preservando la selección en `redirect_uri`. |
| **Flujos Alternativos** | **FA-1: Cambio de selección**<br>1. El usuario ya había seleccionado un plan previamente.<br>2. Selecciona otro plan diferente.<br>3. El sistema sobrescribe la selección anterior en el estado de sesión.<br>4. Muestra confirmación visual del nuevo plan seleccionado.<br><br>**FA-2: Selección con upgrade/downgrade**<br>1. El usuario ya tiene una suscripción activa.<br>2. Al seleccionar un plan diferente, el sistema calcula prorrateo automáticamente.<br>3. Muestra mensaje: "Cambiarás de [Plan Actual] a [Nuevo Plan]. Se aplicará un cargo/prorrateo de [Monto]". |
| **Flujos de Excepción** | **FE-1: Plan no disponible para el país**<br>1. El usuario selecciona un plan cuya disponibilidad geográfica fue modificada entre la carga de la página y el clic.<br>2. El sistema detecta la inconsistencia.<br>3. Muestra: "Este plan no está disponible en tu país. Selecciona otra opción".<br>4. Refresca automáticamente la lista de planes.<br><br>**FE-2: Plan desactivado recientemente**<br>1. El plan fue desactivado por el administrador de la plataforma momentos antes de la selección.<br>2. El sistema muestra: "Este plan ya no está disponible".<br>3. Elimina la opción de la UI y registra el intento. |
| **Reglas de Negocio** | • La selección de plan no genera obligación de pago hasta que se confirme explícitamente en el checkout.<br>• Solo puede haber una selección activa por sesión; una nueva selección sobrescribe la anterior.<br>• Si el usuario tiene suscripción activa, la selección de un plan inferior al actual requiere confirmación adicional de downgrade.<br>• La selección debe persistir mínimo 30 minutos en el estado de sesión del cliente. |
| **Requerimientos asociados** | • RF-SUBS-02 (selección de plan)<br>• RF-SUBS-03 (consulta FX-Service para precio local)<br>• RNF-AUTH-03 (seguridad de sesión para preservar selección) |

---

## CU-SUBS-05: Cancelar Suscripción

| Campo | Detalle |
|---|---|
| **Código** | CU-SUBS-05 |
| **Módulo** | Planes y Suscripciones |
| **Actor(es) principal(es)** | Usuario Final |
| **Actor(es) secundario(s)** | Proveedor de Correo |
| **Prioridad** | Alta |
| **Tipo** | Principal |
| **Precondiciones** | 1. El usuario ha iniciado sesión (CU-USER-04).<br>2. El usuario tiene una suscripción activa (`status = 'active'`).<br>3. El usuario accede al panel de "Mi Suscripción" dentro de la administración de cuenta. |
| **Postcondiciones** | • **Éxito:** La suscripción cambia a estado `cancelled` (soft delete); el usuario conserva acceso hasta el final del período pagado.<br>• **Fallo:** La suscripción permanece activa; se notifica el error. |
| **Flujo Principal** | 1. El usuario accede a "Mi Suscripción" y selecciona "Cancelar suscripción".<br>2. El sistema muestra pantalla de retención: resumen del plan actual, fecha de próximo cobro, y advertencia de pérdida de beneficios.<br>3. El sistema solicita motivo de cancelación (encuesta opcional: precio, contenido, competencia, otros).<br>4. El usuario confirma la cancelación.<br>5. El sistema valida que la suscripción no tenga cargos pendientes o disputas abiertas.<br>6. El sistema actualiza el estado de la suscripción a `cancelled` y establece `access_until` = fecha final del período pagado.<br>7. El sistema notifica a la pasarela de pago para detener cobros recurrentes futuros.<br>8. El sistema ejecuta «include» **CU-USER-03: Notificar Registro** adaptado para enviar correo de confirmación de cancelación (Proveedor de Correo).<br>9. El sistema muestra: "Tu suscripción fue cancelada. Seguirás teniendo acceso hasta el [fecha]". |
| **Flujos Alternativos** | **FA-1: Cancelación con reembolso parcial**<br>1. El sistema detecta que la cancelación ocurre dentro de la ventana de garantía (ej. 7 días).<br>2. Ofrece al usuario: "¿Deseas solicitar reembolso del período actual?".<br>3. Si acepta, se genera ticket de reembolso hacia la pasarela.<br>4. El flujo continúa desde el paso 6.<br><br>**FA-2: Oferta de retención (win-back)**<br>1. Antes de confirmar, el sistema detecta que el usuario es elegible para descuento de retención.<br>2. Muestra oferta: "Quédate con 30% de descuento por 3 meses".<br>3. Si el usuario acepta, se ejecuta CU-SUBS-02 (modificar suscripción) en lugar de cancelar. |
| **Flujos de Excepción** | **FE-1: Suscripción ya cancelada**<br>1. El sistema detecta que la suscripción ya tiene estado `cancelled`.<br>2. Muestra: "Tu suscripción ya fue cancelada previamente. Tu acceso finaliza el [fecha]".<br>3. Deshabilita el botón de cancelación.<br><br>**FE-2: Error en pasarela de pago al detener recurrencia**<br>1. La pasarela no responde o rechaza la solicitud de cancelación de recurrencia.<br>2. El sistema marca la suscripción como `cancellation_pending` y reintenta en background.<br>3. Notifica al usuario: "Estamos procesando tu cancelación. Te confirmaremos por correo".<br><br>**FE-3: Usuario con facturación pendiente**<br>1. El sistema detecta una factura no pagada del período actual.<br>2. Muestra: "Tienes un pago pendiente. Debes regularizarlo antes de cancelar".<br>3. Redirige a CU-SUBS-02 (Administrar Credenciales) para actualizar método de pago. |
| **Reglas de Negocio** | • La cancelación es soft delete: la suscripción se marca `cancelled` pero no se elimina el registro histórico.<br>• El usuario mantiene acceso hasta `access_until` (final del período pagado), sin generar nuevos cobros.<br>• La pasarela debe recibir la solicitud de cancelación de recurrencia dentro de las 24 horas posteriores a la acción del usuario.<br>• Se debe ofrecer al usuario la opción de "Reactivar suscripción" mientras `access_until` no haya vencido.<br>• El correo de confirmación de cancelación debe incluir fecha de corte y enlace a encuesta de satisfacción. |
| **Requerimientos asociados** | • RF-SUBS-04 (cancelar suscripción)<br>• RF-ADMIN-01 (panel de administración de cuenta)<br>• RNF-AUTH-01 (protección de datos de facturación)<br>• RNF-DISP-01 (disponibilidad del servicio de suscripciones) |
