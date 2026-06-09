# Especificación de Requisitos del Software (SRS)
## MentorLink — Plataforma de Mentoría Universitaria UPV

**Versión:** 1.0  
**Fecha:** 2026-06-09  
**Autor:** BerniA1  
**Estándar:** IEEE 830-1998  

---

## Tabla de contenidos

1. [Introducción](#1-introducción)
2. [Descripción general](#2-descripción-general)
3. [Requisitos específicos](#3-requisitos-específicos) (RF1–RF8)
4. [Requisitos no funcionales y restricciones](#4-requisitos-no-funcionales)
5. [Apéndices](#5-apéndices)

---

## 1. Introducción

### 1.1 Propósito

Este documento describe los requisitos del software para **MentorLink**, una plataforma web que conecta mentores y mentees dentro de la comunidad universitaria de la UPV. Va dirigido al equipo de desarrollo y al tutor del TFG.

### 1.2 Ámbito del sistema

MentorLink permite a estudiantes de la UPV encontrar compañeros con más experiencia en una asignatura o área concreta, solicitar sesiones de mentoría y valorar la experiencia. El sistema no gestiona contenidos académicos ni sustituye recursos oficiales de la universidad.

### 1.3 Definiciones, acrónimos y abreviaturas

| Término | Definición |
|---------|------------|
| Mentor | Usuario que ofrece orientación en una o varias asignaturas o áreas |
| Mentee / Alumno | Usuario que solicita mentoría |
| Mentoría | Relación formal entre un mentor y un mentee para una sesión o ciclo de sesiones |
| Matching | Proceso automatizado de recomendación de mentores |
| SRS | Software Requirements Specification |
| UPV | Universitat Politècnica de València |

### 1.4 Referencias

- IEEE Std 830-1998, *Recommended Practice for Software Requirements Specifications*
- Documentación de Supabase: https://supabase.com/docs
- React Router v6: https://reactrouter.com

### 1.5 Visión general del documento

La sección 2 describe el contexto y las características generales del sistema. La sección 3 detalla los requisitos funcionales. La sección 4 recoge los requisitos no funcionales. La sección 5 incluye apéndices con el modelo de datos preliminar.

---

## 2. Descripción general

### 2.1 Perspectiva del producto

MentorLink es una aplicación web de nueva creación, independiente de los sistemas actuales de la UPV. Se compone de:

- **Frontend:** aplicación React (SPA) accesible desde navegador.
- **Backend:** API REST en Node.js + Express.
- **Base de datos:** PostgreSQL gestionada a través de Supabase.
- **Tiempo real:** canal WebSocket proporcionado por Supabase Realtime para el chat.

### 2.2 Funciones del producto

- Registro e identificación de usuarios con perfil académico.
- Motor de matching que recomienda mentores según compatibilidad.
- Gestión del ciclo completo de una mentoría (solicitud → sesiones → cierre).
- Chat interno en tiempo real por mentoría activa.
- Sistema de valoraciones y reputación pública.

### 2.3 Características de los usuarios

| Tipo de usuario | Descripción |
|-----------------|-------------|
| Alumno (mentee) | Estudiante que busca mentoría. Accede al motor de matching, solicita mentorías y valora al mentor. |
| Mentor | Estudiante o egresado que ofrece mentoría. Acepta o rechaza solicitudes y registra sesiones. |
| Ambos | Un mismo usuario puede actuar como mentor en unas asignaturas y como mentee en otras. |

### 2.4 Restricciones

- El sistema está destinado exclusivamente a la comunidad UPV.
- La moderación de contenidos queda fuera del alcance del TFG actual (véase RF8).
- No se integra con el sistema de matrícula ni con el directorio LDAP de la UPV en esta versión.

### 2.5 Suposiciones y dependencias

- Los usuarios disponen de acceso a internet y navegador moderno.
- Supabase mantiene disponibilidad del servicio de base de datos y autenticación.
- Se asume que los usuarios se registran con datos verídicos; no hay verificación institucional en esta versión.

---

## 3. Requisitos específicos

### RF1 — Gestión de usuarios

#### RF1.1 Registro

- El sistema permitirá registrarse mediante correo electrónico y contraseña.
- El sistema validará el formato del correo y exigirá una contraseña de mínimo 8 caracteres.
- El sistema enviará un correo de verificación antes de activar la cuenta.

#### RF1.2 Autenticación

- El sistema permitirá iniciar sesión con las credenciales registradas.
- El sistema mantendrá la sesión activa mediante token JWT gestionado por Supabase Auth.
- El sistema permitirá cerrar sesión desde cualquier dispositivo activo.

#### RF1.3 Perfil académico

- El usuario podrá indicar su titulación, curso actual y asignaturas cursadas o en curso.
- El usuario podrá añadir habilidades técnicas y blandas relevantes para la mentoría.
- El usuario podrá definir su rol: **mentee**, **mentor** o **ambos**.
- El perfil será visible públicamente (excepto el correo electrónico).

#### RF1.4 Edición de perfil

- El usuario podrá modificar cualquier campo del perfil en cualquier momento.
- Los cambios en asignaturas o habilidades actualizarán su visibilidad en el motor de matching de forma inmediata.

#### RF1.5 Recuperación de contraseña

- El usuario podrá solicitar el restablecimiento de su contraseña desde la pantalla de inicio de sesión.
- El sistema enviará un enlace de restablecimiento al correo electrónico registrado, gestionado por Supabase Auth.
- El enlace de restablecimiento tendrá una validez limitada en el tiempo; pasado ese plazo el usuario deberá solicitar uno nuevo.

---

### RF2 — Motor de matching

#### RF2.1 Búsqueda por asignatura

- El mentee podrá buscar mentoría introduciendo el nombre o código de una asignatura.
- El sistema mostrará una lista ordenada de mentores compatibles con esa asignatura.

#### RF2.2 Puntuación de compatibilidad

La compatibilidad entre mentee y mentor se calculará mediante una puntuación ponderada con los siguientes factores:

| Factor | Peso |
|--------|------|
| Coincidencia en la asignatura solicitada | 40 % |
| Valoración media acumulada del mentor | 30 % |
| Número de mentorías completadas | 20 % |
| Coincidencia de titulación | 10 % |

#### RF2.3 Recomendación principal

- El sistema destacará en primera posición al mentor con mayor puntuación.
- Si no hay mentores disponibles para la asignatura, el sistema lo indicará explícitamente.

#### RF2.4 Exploración libre

- El mentee podrá ver la lista completa de mentores compatibles y elegir libremente uno distinto al recomendado.
- El mentee podrá filtrar la lista por valoración mínima y número de mentorías completadas.

---

### RF3 — Ciclo de mentoría

#### RF3.1 Solicitud

- El mentee podrá enviar una solicitud de mentoría a un mentor seleccionado, indicando la asignatura y una descripción opcional de sus necesidades.
- Una solicitud quedará en estado **pendiente** hasta que el mentor responda.

#### RF3.2 Respuesta del mentor

- El mentor recibirá una notificación de nueva solicitud.
- El mentor podrá **aceptar** o **rechazar** la solicitud.
- Al rechazar, el mentor podrá indicar un motivo (opcional).
- Al aceptar, se creará una mentoría activa entre ambos usuarios.

#### RF3.3 Registro de sesiones

- Cualquiera de los dos participantes podrá registrar una sesión indicando fecha, duración y notas opcionales.
- El historial de sesiones será visible para ambos participantes de la mentoría.

#### RF3.4 Cierre de mentoría

- Cualquiera de los participantes podrá cerrar la mentoría.
- Al cerrar, el mentee podrá valorar al mentor (véase RF5).
- Una mentoría cerrada pasará a estado **completada** y no admitirá nuevas sesiones.

#### RF3.5 Historial

- El usuario podrá consultar el historial completo de sus mentorías (activas, completadas y rechazadas).

---

### RF4 — Chat interno en tiempo real

#### RF4.1 Canal por mentoría

- Cada mentoría activa dispondrá de un canal de chat exclusivo entre mentor y mentee.
- El chat no estará disponible en mentorías en estado pendiente, rechazada o completada.

#### RF4.2 Mensajería en tiempo real

- Los mensajes se entregarán en tiempo real mediante WebSocket (Supabase Realtime).
- Los mensajes no entregados en tiempo real se recuperarán al reconectar.

#### RF4.3 Historial de mensajes

- El historial completo del chat estará disponible mientras la mentoría exista en el sistema.
- Los mensajes mostrarán remitente, contenido y marca de tiempo.

#### RF4.4 Restricciones del chat

- No se soportarán archivos adjuntos ni imágenes en esta versión.
- El contenido de los mensajes no será moderado automáticamente en esta versión (véase RF8).

---

### RF5 — Valoraciones y reputación

#### RF5.1 Valoración al cerrar

- Al cerrar una mentoría completada, el sistema solicitará al mentee que valore al mentor con una puntuación de 1 a 5 estrellas y un comentario opcional.
- La valoración es opcional pero se incentivará mostrando un recordatorio.

#### RF5.2 Puntuación acumulada

- La puntuación de un mentor será la media aritmética de todas las valoraciones recibidas.
- La puntuación se mostrará en el perfil público del mentor y en los resultados del motor de matching.

#### RF5.3 Visibilidad

- Las valoraciones y comentarios serán visibles públicamente en el perfil del mentor valorado.
- El autor de una valoración no será anónimo.

---

### RF6 — Notificaciones

#### RF6.1 Eventos notificables

El sistema generará una notificación al usuario afectado en los siguientes eventos:

| Evento | Destinatario |
|--------|-------------|
| Nueva solicitud de mentoría recibida | Mentor |
| Solicitud aceptada | Mentee |
| Solicitud rechazada | Mentee |
| Nuevo mensaje en el chat | El participante que no envió el mensaje |
| Mentoría cerrada por la otra parte | Ambos participantes |

#### RF6.2 Canal de notificación

- Las notificaciones se mostrarán dentro de la propia aplicación (in-app) mediante un indicador en la barra de navegación.
- Adicionalmente, el sistema enviará un correo electrónico al destinatario para los eventos de solicitud recibida, solicitud aceptada y solicitud rechazada.
- El usuario podrá consultar el historial de notificaciones desde la aplicación.

---

### RF7 — Panel de inicio

#### RF7.1 Vista principal tras el login

- Tras autenticarse, el usuario accederá a un panel de inicio que mostrará:
  - Sus mentorías activas con el estado y la fecha de la última actividad.
  - Sus solicitudes pendientes de respuesta (como mentor o como mentee).
  - El número de mensajes no leídos en el chat.

#### RF7.2 Accesos rápidos

- El panel incluirá accesos directos a: buscar mentor (motor de matching), ver perfil propio y consultar historial de mentorías.
- Si el usuario no tiene mentorías ni solicitudes, el panel mostrará un mensaje de bienvenida con un enlace al motor de matching.

---

### RF8 — Moderación (trabajo futuro)

> **Fuera del alcance del TFG actual.**

Se identifica como trabajo futuro un módulo de moderación que incluiría:

- Panel de administración para gestionar usuarios y contenidos.
- Reporte de mensajes o valoraciones inapropiadas.
- Sistema de sanciones (advertencia, suspensión temporal, baja definitiva).

---

## 4. Requisitos no funcionales

### RNF1 — Rendimiento

- La búsqueda de mentores deberá devolver resultados en menos de 2 segundos para un volumen de hasta 500 usuarios registrados.

### RNF2 — Usabilidad

- La interfaz será responsiva y funcional en pantallas de escritorio y móvil.
- El flujo de registro y primera solicitud de mentoría no superará 5 pasos.

### RNF3 — Seguridad

- Las contraseñas se almacenarán hasheadas (gestionado por Supabase Auth).
- Las rutas de la API exigirán token JWT válido, salvo registro y login.
- Se aplicarán políticas Row Level Security (RLS) en Supabase para aislar datos entre usuarios.

### RNF4 — Disponibilidad

- Sin SLA formal; el sistema debe funcionar correctamente durante el periodo de demostración funcional del TFG.

### RNF5 — Mantenibilidad

- El código se organizará en módulos separados para autenticación, matching, mentorías, chat y valoraciones.
- La API REST se documentará mediante un archivo OpenAPI (YAML/JSON), generado o mantenido manualmente junto al código fuente.

---

### RNF6 — Restricciones del sistema

Las siguientes restricciones delimitan el alcance técnico y de negocio de la versión entregada como TFG:

| # | Restricción | Detalle |
|---|-------------|---------|
| C1 | Comunidad exclusiva UPV | La plataforma está destinada únicamente a estudiantes y egresados de la Universitat Politècnica de València. No se permitirá el acceso a usuarios externos. |
| C2 | Sin integración institucional | No se integra con el directorio LDAP ni con el sistema de matrícula de la UPV en esta versión. La verificación de pertenencia a la UPV queda pendiente de trabajo futuro. |
| C3 | Sin moderación automática | No existe módulo de moderación automática de mensajes ni valoraciones. Esta funcionalidad se identifica como trabajo futuro (véase RF8). |
| C4 | Sin archivos adjuntos en el chat | El canal de mensajería no soporta el envío de archivos adjuntos ni imágenes. Solo se admite texto plano. |
| C5 | Stack gratuito | Toda la infraestructura utiliza planes gratuitos: React + Vite (frontend), Node.js + Express (backend) y Supabase Free Tier (base de datos, autenticación y tiempo real). No se incurrirá en coste de hosting durante el TFG. |

---

## 5. Apéndices

### Apéndice A — Modelo de datos preliminar

```
users
  id, email, created_at, role (mentee|mentor|both)

profiles
  user_id, full_name, titulacion, curso, bio, avg_rating, total_mentorships

skills
  id, name

user_skills
  user_id, skill_id

subjects
  id, code, name, titulacion

user_subjects
  user_id, subject_id, as_mentor (bool)

mentorships
  id, mentor_id, mentee_id, subject_id, status (pending|active|completed|rejected),
  created_at, closed_at, reject_reason

sessions
  id, mentorship_id, date, duration_min, notes, created_by

messages
  id, mentorship_id, sender_id, content, created_at

ratings
  id, mentorship_id, rater_id, rated_id, score (1-5), comment, created_at
```

### Apéndice B — Diagrama de estados de una mentoría

```
[Solicitud enviada]
        │
        ▼
    PENDIENTE
    /         \
RECHAZADA    ACTIVA ──→ RECHAZADA
                │         (cancelada por mentor)
            COMPLETADA
```
