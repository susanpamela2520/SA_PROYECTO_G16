# Quetxal TV — Proyecto Fase 1

**Curso:** Software Avanzado  
**Universidad:** San Carlos de Guatemala — Facultad de Ingeniería  
**Período:** Vacaciones de Junio 2026  
**Repositorio:** `SA_PROYECTO_GX`

---

## Tabla de integrantes

| Carné | Nombre | Módulo asignado |
|-------|--------|-----------------|
| 202041284 | German Enrique Caál Jalal | Auth & Suscripciones |
| _______ | _______________________ | Catálogo & Calificaciones |
| 201612218 | Susan Pamela Herrera Monzon | FX-Service, Historial & Notificaciones |
| _______ | _______________________ | API Gateway, Infraestructura & Arquitectura |

---

## Índice

1. [Introducción](#introducción)
2. [Descripción del sistema](#descripción-del-sistema)
3. [Requerimientos del sistema](#requerimientos-del-sistema)
   - [Requerimientos Funcionales (RF)](#requerimientos-funcionales-rf)
   - [Requerimientos No Funcionales (RNF)](#requerimientos-no-funcionales-rnf)
4. [Modelo de Casos de Uso](#modelo-de-casos-de-uso)
   - [Diagrama de alto nivel](#diagrama-de-alto-nivel)
   - [Descomposición por módulo](#descomposición-por-módulo)
   - [Casos de uso expandidos](#casos-de-uso-expandidos)
5. [Vista de Arquitectura — Modelo 4+1](#vista-de-arquitectura--modelo-41)
   - [Vista de Escenarios](#vista-de-escenarios)
   - [Vista Lógica](#vista-lógica)
   - [Vista de Procesos](#vista-de-procesos)
   - [Vista de Componentes (Desarrollo)](#vista-de-componentes-desarrollo)
   - [Vista de Despliegue (Física)](#vista-de-despliegue-física)
6. [Diagramas Estructurales y de Comportamiento](#diagramas-estructurales-y-de-comportamiento)
   - [Diagrama de arquitectura general](#diagrama-de-arquitectura-general)
   - [Diagrama de componentes](#diagrama-de-componentes)
   - [Diagrama de despliegue](#diagrama-de-despliegue)
   - [Diagrama de actividades](#diagrama-de-actividades)
   - [Diagrama de secuencia — Login y validación JWT](#diagrama-de-secuencia--login-y-validación-jwt)
   - [Diagrama de secuencia — Consumo de video](#diagrama-de-secuencia--consumo-de-video)
7. [Diseño de Base de Datos](#diseño-de-base-de-datos)
   - [Diagrama Entidad-Relación](#diagrama-entidad-relación)
   - [Objetos programables de base de datos](#objetos-programables-de-base-de-datos)
8. [Stack tecnológico](#stack-tecnológico)
9. [Estructura del repositorio](#estructura-del-repositorio)
10. [Instrucciones de despliegue](#instrucciones-de-despliegue)
    - [Entorno local](#entorno-local)
    - [Entorno cloud (GCP)](#entorno-cloud-gcp)
11. [Conclusiones](#conclusiones)

---

## Introducción


---

## Descripción del sistema

Quetxal TV es una plataforma de streaming de video bajo demanda construida bajo una arquitectura de microservicios políglota. El sistema utiliza **TypeScript**, **Go** y **Python** distribuidos estratégicamente por dominio, comunicación interna mediante **gRPC**, un **API Gateway** como único punto de entrada, y caché con **Redis**. La infraestructura completa está contenida en **Docker** y orquestada con **Docker Compose**, con despliegue en **Google Cloud Platform**.

### Módulos del sistema

| # | Módulo | Lenguaje | Descripción |
|---|--------|----------|-------------|
| 1 | Auth & Multiperfil | TypeScript | Registro, login, JWT, OAuth, perfiles |
| 2 | Suscripciones | TypeScript | Planes y gestión de cuenta |
| 3 | Catálogo | Go | Búsqueda y detalle de contenido |
| 4 | Calificaciones | Go | Sistema de rating dinámico |
| 5 | FX-Service | Python | Tipos de cambio con Redis Cache |
| 6 | Historial | Python | Progreso de reproducción por perfil |
| 7 | Notificaciones | Python | Correos automáticos |
| 8 | API Gateway | TypeScript | Enrutamiento y seguridad centralizada |

---

## Requerimientos del sistema

### Requerimientos Funcionales (RF)

| ID | Prioridad | Módulo | Descripción |
|----|-----------|--------|-------------|
| RF-01 | Alta | Auth | El sistema debe permitir el registro de nuevos usuarios con email y contraseña. |
| RF-02 | Alta | Auth | El sistema debe permitir el inicio de sesión con generación de JWT. |
| RF-03 | Alta | Auth | El sistema debe soportar autenticación mediante OAuth. |
| RF-04 | Alta | Auth | El sistema debe permitir crear hasta 5 perfiles por cuenta, cada uno con historial aislado. |
| RF-05 | Alta | Auth | El sistema debe permitir seleccionar, editar y eliminar perfiles. |
| RF-06 | Alta | Suscripciones | El sistema debe mostrar los planes disponibles (Básico, Estándar, Premium) con sus precios. |
| RF-07 | Alta | Suscripciones | El sistema debe permitir suscribirse, modificar o cancelar un plan. |
| RF-08 | Media | Suscripciones | El sistema debe permitir actualizar credenciales de acceso desde el panel de cuenta. |
| RF-09 | Alta | Catálogo | El sistema debe permitir búsqueda por título, género y categoría. |
| RF-10 | Alta | Catálogo | El sistema debe mostrar vista detallada con ficha técnica y reparto de cada contenido. |
| RF-11 | Alta | Calificaciones | El sistema debe permitir calificar contenido con estrellas o pulgar arriba/abajo. |
| RF-12 | Alta | Calificaciones | El catálogo debe mostrar el porcentaje global de recomendación calculado dinámicamente. |
| RF-13 | Alta | FX | El sistema debe mostrar el precio de los planes en la moneda local del usuario. |
| RF-14 | Alta | FX | El sistema debe cachear los tipos de cambio en Redis con políticas TTL. |
| RF-15 | Alta | Historial | El sistema debe registrar el progreso de reproducción por perfil. |
| RF-16 | Alta | Historial | Para series, el sistema debe almacenar temporada, capítulo y minuto exacto de pausa. |
| RF-17 | Media | Notificaciones | El sistema debe enviar correo de confirmación al registrar un usuario. |
| RF-18 | Media | Notificaciones | El sistema debe enviar recibo de compra al suscribirse a un plan. |
| RF-19 | Baja | Notificaciones | El sistema debe enviar alertas de nuevo contenido disponible. |

### Requerimientos No Funcionales (RNF)

| ID | Atributo | Descripción |
|----|----------|-------------|
| RNF-01 | Seguridad | Toda comunicación externa debe pasar por el API Gateway con validación JWT. |
| RNF-02 | Seguridad | Las contraseñas deben almacenarse con hashing (bcrypt). |
| RNF-03 | Seguridad | La información sensible (URLs, passwords, IPs) debe manejarse exclusivamente con archivos `.env`. |
| RNF-04 | Escalabilidad | Cada microservicio debe ser independiente y desplegable de forma autónoma. |
| RNF-05 | Rendimiento | El FX-Service debe responder en menos de 200ms gracias a la capa de caché Redis. |
| RNF-06 | Disponibilidad | El sistema debe contar con políticas de reinicio automático en el entorno cloud. |
| RNF-07 | Mantenibilidad | El código debe seguir los principios SOLID en todos los microservicios. |
| RNF-08 | Portabilidad | Todo servicio debe poder levantarse con un único comando (`docker compose up`). |
| RNF-09 | Trazabilidad | Los cambios de credenciales deben quedar registrados mediante triggers de base de datos. |
| RNF-10 | Colaboración | Todo cambio de código debe integrarse mediante Pull Request con al menos 1 aprobación. |

---

## Modelo de Casos de Uso

### Diagrama de alto nivel


![Diagrama de casos de uso — alto nivel](./arquitectura/diagramas/uc-alto-nivel.png)

### Descomposición por módulo


#### Módulo 1 — Auth & Multiperfil
![UC Auth](./arquitectura/diagramas/uc-auth.png)

#### Módulo 2 — Suscripciones
![UC Suscripciones](./arquitectura/diagramas/uc-suscripciones.png)

#### Módulo 3 — Catálogo & Calificaciones
![UC Catálogo](./arquitectura/diagramas/uc-catalogo.png)

#### Módulo 4 — FX, Historial & Notificaciones
![UC FX-Historial](./arquitectura/diagramas/uc-fx-historial.png)

### Casos de uso expandidos


---

#### CU-01 — Registro de usuario

| Campo | Detalle |
|-------|---------|
| **Actores** | Usuario no registrado |
| **Precondición** | El usuario no tiene cuenta activa |
| **Postcondición** | Cuenta creada, correo de confirmación enviado |

**Flujo principal:**
1. El usuario ingresa email, contraseña y nombre.
2. El sistema valida que el email no esté registrado.
3. El sistema hashea la contraseña y crea la cuenta.
4. El sistema genera un perfil por defecto.
5. El sistema llama al servicio de notificaciones para enviar correo de bienvenida.
6. El sistema retorna JWT y refresh token.

**Flujo alternativo — email ya registrado:**
- En el paso 2, si el email existe, el sistema retorna error `409 Conflict`.

**Flujo de excepción — fallo del servicio de notificaciones:**
- En el paso 5, si el servicio de notificaciones falla, la cuenta se crea igualmente y el correo se reintenta de forma asíncrona.

---

poner todos los casos de uso de aqui para abajo
---

## Vista de Arquitectura — Modelo 4+1

### Vista de Escenarios

Descripcion

### Vista Lógica

. Describir las principales entidades de dominio por servicio._

### Vista de Procesos

 _Diagrama de actividades o de secuencia mostrando 

### Vista de Componentes (Desarrollo)

 _Diagrama de componentes 
### Vista de Despliegue (Física)

 _Diagrama de despliegue m

---

## Diagramas Estructurales y de Comportamiento

### Diagrama de arquitectura general


![Arquitectura general](./arquitectura/diagramas/arquitectura-general.png)

### Diagrama de componentes

![Componentes](./arquitectura/diagramas/componentes.png)

### Diagrama de despliegue

![Despliegue](./arquitectura/diagramas/despliegue.png)

### Diagrama de actividades



![Actividades — Login](./arquitectura/diagramas/actividades-login.png)

### Diagrama de secuencia — Login y validación JWT



![Secuencia Login](./arquitectura/diagramas/secuencia-login.png)

### Diagrama de secuencia — Consumo de video



![Secuencia Video](./arquitectura/diagramas/secuencia-video.png)

---

## Diseño de Base de Datos

### Diagrama Entidad-Relación



![Diagrama ER](./arquitectura/diagramas/er-diagram.png)

### Objetos programables de base de datos

| Tipo | Nombre | Servicio | Descripción |
|------|--------|----------|-------------|
| Stored Procedure | `sp_register_purchase` | subscription-service | Registra la suscripción, el pago y actualiza el estado del usuario en una sola transacción. |
| Stored Procedure | `sp_cancel_subscription` | subscription-service | Cancela la suscripción y registra la fecha de expiración. |
| View | `vw_catalog_billboard` | catalog-service | Vista que consolida contenido con su porcentaje de recomendación para la cartelera principal. |
| View | `vw_actor_profile` | catalog-service | Vista que une actores con el contenido en el que participan. |
| Function | `fn_recommendation_pct` | rating-service | Calcula el porcentaje global de recomendación de un contenido a partir de sus calificaciones. |
| Trigger | `trg_credential_audit` | auth-service | Registra en tabla de auditoría cada cambio de email o contraseña con timestamp y user_id. |

---

## Conclusiones



**Integrante 1 — Auth & Suscripciones:**

**Integrante 2 — Catálogo & Calificaciones:**

**Integrante 3 — FX, Historial & Notificaciones:**


**Integrante 4 — Gateway & Infraestructura:**
