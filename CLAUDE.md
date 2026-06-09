# Contexto del proyecto — MentorLink (TFG)

## Qué es este proyecto

MentorLink es una plataforma web desarrollada como Trabajo de Fin de Grado en la UPV (Universitat Politècnica de València). Su objetivo es conectar mentores y mentees dentro de la comunidad universitaria: estudiantes que buscan orientación con estudiantes o egresados que quieren compartir su experiencia.

## Tipos de mentoría que cubre

- **Académica**: ayuda con asignaturas, TFGs, proyectos de carrera.
- **Profesional**: orientación sobre prácticas, búsqueda de empleo, salida laboral.

## Stack tecnológico

| Capa | Tecnología |
|------|------------|
| Frontend | React (Vite) |
| Backend | Node.js + Express |
| Base de datos | PostgreSQL vía Supabase |

## Estructura del repositorio

```
tfg/
├── frontend/   # React + Vite
├── backend/    # Node + Express (API REST)
├── CLAUDE.md
├── README.md
└── .gitignore
```

## Estado actual

Esqueleto inicial creado. Sin lógica de negocio todavía.

## Decisiones tomadas

- Idioma de comunicación con Claude: **español**.
- Repositorio en `/home/ayuda62/tfg`.
- Arquitectura: monorepo con carpetas `frontend/` y `backend/` separadas.
- ORM/cliente Supabase: por definir (candidatos: `@supabase/supabase-js` en frontend y/o backend).

## Convenciones

- Responder siempre en español.
- Preferir editar archivos existentes antes de crear nuevos.
- Sin comentarios de código salvo que el *por qué* no sea obvio.

## Contacto / Autor

berjo04loz@gmail.com
