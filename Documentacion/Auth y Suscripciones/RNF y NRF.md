## 1. Requerimientos Funcionales (RF) — Módulo Auth, Perfiles y Suscripciones

| ID | Prioridad | Requerimiento |
|:---|:---|:---|
| **RF-AUTH-01** | Alta | El sistema debe permitir el **registro de nuevos usuarios** solicitando: correo electrónico, contraseña segura (mín. 8 caracteres, mayúsculas, minúsculas, números y símbolo), nombre completo y país de residencia. |
| **RF-AUTH-02** | Alta | El sistema debe permitir el **inicio de sesión seguro** validando credenciales activas y generando un **JWT** con claims: `sub` (userId), `email`, `roles`, `iat`, `exp` y `jti`. |
| **RF-AUTH-03** | Alta | El sistema debe propagar la identidad del usuario mediante **JWT firmado (RS256)** en la comunicación service-to-service a través del API Gateway. |
| **RF-AUTH-04** | Alta | El sistema debe mantener el estado de sesión en el cliente mediante **Session Cookies seguras** (`HttpOnly`, `Secure`, `SameSite=Strict`, `Max-Age/Expires` configurable). |
| **RF-AUTH-05** | Media | El sistema debe integrar **OAuth 2.0 / OpenID Connect** como método alternativo de autenticación (Google, GitHub o similar), permitiendo login federado y vinculación a cuenta local. |
| **RF-AUTH-06** | Alta | El sistema debe implementar **logout seguro** invalidando el JWT en una lista de denegación (denylist/blacklist) o rotando el secret de firma, y eliminando la session cookie del cliente. |
| **RF-PROF-01** | Alta | El sistema debe permitir la creación y administración de **hasta 5 perfiles** por cuenta de usuario. |
| **RF-PROF-02** | Alta | Cada perfil debe mantener **aislamiento total e independiente** de: historial de reproducción, preferencias, calificaciones y progreso de visualización. |
| **RF-PROF-03** | Media | El sistema debe permitir asignar un nombre, avatar/imagen y clasificación por edades a cada perfil. |
| **RF-PROF-04** | Media | El usuario debe poder **editar o eliminar** perfiles existentes, excepto el perfil principal (owner), que solo puede eliminarse junto con la cuenta. |
| **RF-SUBS-01** | Alta | El sistema debe ofrecer la visualización de **tres planes de suscripción**: Básico, Estándar y Premium, mostrando: nombre, precio base (USD), resolución máxima, número de dispositivos simultáneos y funciones incluidas. |
| **RF-SUBS-02** | Alta | El sistema debe permitir la **selección y activación de un plan** vinculándolo a la cuenta del usuario y registrando la fecha de inicio, fecha de renovación y estado activo. |
| **RF-SUBS-03** | Alta | El microservicio de suscripciones debe consultar el **FX-Service** para mostrar el precio del plan en la **moneda local** del usuario, utilizando la tasa de cambio cacheada en Redis. |
| **RF-ADMIN-01** | Media | El sistema debe proveer un **panel de administración de cuenta** donde el usuario pueda: modificar su plan actual (upgrade/downgrade), cancelar la suscripción (soft delete / status inactive) y visualizar el historial de pagos. |
| **RF-ADMIN-02** | Media | El usuario debe poder **actualizar sus credenciales de acceso** (cambio de contraseña y correo electrónico) desde el panel, requiriendo re-autenticación previa. |
| **RF-DB-01** | Alta | La base de datos debe implementar un **Stored Procedure** `sp_registro_unificado_compra` que ejecute de forma transaccional: validación de usuario, registro de suscripción, generación de recibo y actualización de estado de cuenta. Si algún paso falla, debe realizarse **rollback total**. |
| **RF-DB-02** | Alta | La base de datos debe implementar un **Trigger** `tr_auditoria_credenciales` que registre automáticamente en una tabla de auditoría cualquier operación `UPDATE` sobre las credenciales del usuario (contraseña, email), almacenando: userId, campo modificado, fecha/hora, IP de origen (si aplica) y tipo de operación. |

---

## 2. Requerimientos No Funcionales (RNF) — Módulo Auth, Perfiles y Suscripciones

| ID | Categoría | Prioridad | Requerimiento |
|:---|:---|:---|:---|
| **RNF-AUTH-01** | Seguridad | Crítica | Las contraseñas deben almacenarse utilizando **bcrypt/Argon2id** con un cost factor ≥ 12 y nunca en texto plano. |
| **RNF-AUTH-02** | Seguridad | Crítica | Los JWT deben firmarse con **RS256** (par de claves privada/pública) con una longitud de clave ≥ 2048 bits y expirar en **15 minutos** (access token). Los refresh tokens deben expirar en **7 días**. |
| **RNF-AUTH-03** | Seguridad | Alta | Las Session Cookies deben incluir flags obligatorios: `HttpOnly`, `Secure` (en producción), `SameSite=Strict` y atributo `Path=/`. |
| **RNF-AUTH-04** | Seguridad | Alta | El sistema debe limitar intentos de login fallidos a **5 intentos en 15 minutos** por IP/cuenta, aplicando bloqueo temporal progresivo. |
| **RNF-AUTH-05** | Seguridad | Media | La integración OAuth debe validar el `state` param y verificar el `id_token` contra el JWKS del proveedor para prevenir ataques de replay y suplantación. |
| **RNF-PERF-01** | Rendimiento | Alta | El endpoint de login debe responder en **≤ 300 ms** (p95) bajo carga normal, incluyendo la generación de JWT y validación de credenciales. |
| **RNF-PERF-02** | Rendimiento | Media | La consulta de planes de suscripción con conversión de moneda debe responder en **≤ 200 ms** aprovechando el cache de Redis (TTL ≥ 1 hora para tipos de cambio). |
| **RNF-DISP-01** | Disponibilidad | Alta | El microservicio de autenticación debe mantener una **disponibilidad ≥ 99.9%** en ventanas de 30 días, tolerando fallos mediante replicación de instancias. |
| **RNF-SCAL-01** | Escalabilidad | Media | La arquitectura debe permitir el **escalamiento horizontal** del servicio de autenticación y suscripciones sin estado (stateless), permitiendo ≥ 3 réplicas balanceadas. |
| **RNF-PORT-01** | Portabilidad | Alta | Cada microservicio debe contener su propio **Dockerfile** multi-stage, generando imágenes ≤ 200 MB y exponiendo variables de configuración exclusivamente mediante **archivos `.env`** (nunca hardcodeadas). |
| **RNF-MAINT-01** | Mantenibilidad | Media | El código debe seguir los principios **SOLID**, con cobertura de pruebas unitarias ≥ 70% y pruebas de integración para los endpoints críticos de auth. |
| **RNF-CODE-01** | Gobierno | Alta | Todo cambio en el código debe integrarse mediante **Pull Request** a la rama `develop` o `main`, requiriendo al menos **1 aprobación** del equipo antes del merge. Commits directos quedan prohibidos. |

