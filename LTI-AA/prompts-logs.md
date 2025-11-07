# Prompt 1

Con la tecnica de meta prompting ayudame a mejorar el siguiente prompt (adicionalmente hazme preguntas interactivamente si hay algo por aclarar)

## Rol

Eres un experto en Product Management y Business Analyst.

## Contexto

Tu misión es preparar la documentación necesaria para empezar a implementar LTI especificados en @LTI-AA.md

## Resultado deseado

Debes generar un markdown con los siguientes puntos:

1. Genera historias de usuario con la siguiente estructura:

   Formato estándar: "Como [tipo de usuario], quiero [realizar una acción] para [obtener un beneficio]".
   Descripción: Una descripción concisa y en lenguaje natural de la funcionalidad que el usuario desea.
   Criterios de Aceptación: Condiciones específicas que deben cumplirse para considerar la User Story como "terminada", éstos deberian de seguir un formato similar a “Dado que” [contexto inicial], "cuando” [acción realizada], “entonces” [resultado esperado].
   Notas adicionales: Notas que puedan ayudar al desarrollo de la historia
   Tareas: Lista de tareas y subtareas para que esta historia pueda ser completada

2. Debes armar el Backlog de producto con las User Stories, priorizándolas como consideres conveniente
3. Elige solo una User Story (la mas importante), y genera los Tickets de trabajo. Aterrízalos técnicamente
4. Estima el esfuerzo de los tickets de trabajo usando la metodología fibonacci.

## Restricciones

- Genera la documentación considerando que es un MVP, no hagas sobreingenieria
- Usa un tono técnico, además agrega comentarios si es absolutamente necesario

## Resultado

Escribe el resultado en el archivo UserStories-AA.md

---

# Prompt 2

# Prompt Mejorado - Product Management Documentation para LTI ATS

## Contexto del Sistema

Eres un Senior Product Manager y Business Analyst especializado en productos SaaS B2B y sistemas con componentes de IA.

## Información de Entrada

Analiza la documentación técnica y de producto del sistema LTI (Applicant Tracking System) disponible en @LTI-AA.md

## Tarea Principal

Genera documentación de Product Management lista para desarrollo, enfocada en un **MVP funcional** que implemente únicamente el **flujo principal de AI Interview Assistant** (Caso de Uso 1 del documento).

## Entregables Específicos

### 1. User Stories (Formato Estándar)

Para cada funcionalidad del flujo principal, genera:

**Estructura obligatoria:**

```
### US-[ID]: [Título descriptivo]

**Como** [rol de usuario]
**Quiero** [capacidad/acción]
**Para** [beneficio/valor de negocio]

**Descripción:**
[2-3 oraciones explicando el contexto y valor para el usuario]

**Criterios de Aceptación:**
- [ ] **Dado que** [contexto/precondición inicial]
      **Cuando** [acción o evento que ocurre]
      **Entonces** [resultado observable y verificable]
- [ ] [Criterios adicionales si aplica]

**Notas Técnicas:**
- [Consideraciones de arquitectura general: qué servicios están involucrados (ej: AI Service, Core Service)]
- [Integraciones externas necesarias (ej: OpenAI API)]
- [Decisiones técnicas relevantes a nivel arquitectónico]

**Tareas de Implementación:**
- [ ] [Tarea técnica 1: específica y accionable]
  - [ ] [Subtarea técnica si aumenta claridad]
- [ ] [Tarea técnica 2: ...]
- [ ] [Tarea técnica N: ...]

**Definición de Done:**
- [ ] Código implementado y revisado
- [ ] Tests unitarios y de integración ejecutados exitosamente
- [ ] Documentación técnica actualizada
```

### 2. Product Backlog Priorizado

Genera una tabla markdown con todas las User Stories priorizadas según **valor para el usuario**:

| ID     | User Story | Prioridad | Valor para Usuario                                  | Dependencias |
| ------ | ---------- | --------- | --------------------------------------------------- | ------------ |
| US-001 | [Título]   | P0        | [Explicar impacto directo en experiencia/beneficio] | -            |
| US-002 | [Título]   | P1        | [Explicar valor]                                    | US-001       |

**Criterios de Priorización:**

- **P0 (Critical)**: Entrega valor inmediato y es requisito mínimo del flujo
- **P1 (High)**: Mejora significativa de experiencia pero flujo funciona sin ella
- **P2 (Medium)**: Optimización o mejora incremental

### 3. Desglose Técnico de US Más Importante

Selecciona la User Story **P0 con mayor valor para el usuario** y genera tickets de trabajo técnicos:

**Estructura de Ticket:**

```
### TICKET-[ID]: [Título técnico]

**Tipo:** [Backend | Frontend | Infrastructure | AI/ML]
**User Story:** US-XXX
**Estimación Fibonacci:** [1, 2, 3, 5, 8, 13]

**Descripción Técnica:**
[Qué componente/servicio se debe construir o modificar según arquitectura C4]
[Input y output esperados]

**Alcance a Nivel Arquitectónico:**
- **Servicios involucrados:** [ej: AI Service, API Gateway, Core Service]
- **Integraciones externas:** [ej: OpenAI API - modelo GPT-4]
- **Capa de datos:** [Entidades principales del modelo de datos involucradas]
- **Frontend:** [Componentes/vistas principales si aplica]

**Criterios Técnicos de Aceptación:**
- [ ] [Requisito técnico específico y verificable]
- [ ] [Requisito adicional]

**Dependencias Técnicas:**
- [TICKET-XXX debe estar completado antes]
- [Infraestructura/configuración requerida]

**Notas de Implementación:**
[Consideraciones técnicas importantes, trade-offs, o decisiones de diseño]
```

### 4. Estimación Fibonacci

Para cada ticket técnico, aplica escala Fibonacci con justificación:

**Guía de Estimación:**

- **1-2 pts**: Tarea simple, < 4 horas, sin dependencias complejas
- **3 pts**: Tarea estándar, ~1 día, implementación directa
- **5 pts**: Complejidad media, 2-3 días, requiere coordinación entre capas
- **8 pts**: Complejo, ~1 semana, múltiples servicios involucrados
- **13 pts**: Muy complejo, >1 semana, considerar subdividir

**Tabla de Estimaciones:**
| Ticket ID | Título | Puntos | Justificación de Estimación |
|-----------|--------|--------|----------------------------|
| TICKET-001 | ... | 5 | [Razón basada en complejidad técnica] |

**Resumen:**

- Total Story Points: [Suma]
- Velocidad estimada: [Si aplica]

## Restricciones y Lineamientos

### Alcance Estricto del MVP

**Incluir ÚNICAMENTE:**

- Flujo principal de "Caso de Uso 1: Conducir entrevista con asistencia de IA"
  - Sistema carga perfil de candidato y job description
  - IA genera guía de entrevista con preguntas sugeridas
  - Reclutador realiza preguntas
  - IA transcribe respuestas (post-procesado, NO tiempo real)
  - IA sugiere preguntas de profundización
  - Reclutador finaliza entrevista
  - IA genera resumen estructurado automático
  - Sistema guarda todo en perfil del candidato

**Excluir explícitamente:**

- Gestión de candidatos avanzada (solo CRUD básico para el flujo)
- Sistema de feedback colaborativo
- Toma de decisión con votación
- Programación de entrevistas e integración con calendarios
- Reportes y analytics
- Transcripción en tiempo real (solo post-procesada)
- Integración con Outlook
- Recordatorios automáticos
- Comparación de candidatos

### Principios de Diseño MVP

- **Simplicidad primero**: No agregar funcionalidades "por si acaso"
- **Validación rápida**: Priorizar lo que permite probar hipótesis de valor
- **API-first**: Diseñar pensando en integraciones futuras
- **Arquitectura limpia**: Separación entre servicios según diagrama C4

### Tono y Formato

- Lenguaje técnico y preciso
- Referencias a arquitectura C4 cuando sea relevante (ej: "AI Service - Summary Controller")
- Referencias a modelo de datos cuando sea necesario (ej: "Interview entity, ai_summary field")
- Comentarios solo cuando agreguen claridad técnica no obvia
- Markdown bien estructurado con jerarquía clara

## Formato de Salida

Escribe el resultado completo en el archivo `UserStories-AA.md` siguiendo esta estructura:

```markdown
# LTI ATS - Product Backlog MVP: AI Interview Assistant

## 1. Visión del MVP

[Párrafo ejecutivo: qué resuelve este MVP y por qué este flujo específico]

## 2. User Stories Completas

[Todas las US con formato completo especificado arriba]

## 3. Product Backlog Priorizado

[Tabla completa con priorización por valor de usuario]

## 4. Desglose Técnico: [Título de US Seleccionada]

### Contexto

[Breve explicación de por qué es la más importante]

### Tickets Técnicos

[Todos los tickets con formato completo]

### Dependencias entre Tickets

[Diagrama o lista mostrando orden de implementación]

## 5. Estimación y Planning

[Tabla de estimaciones Fibonacci]
[Resumen de story points]
[Consideraciones de capacity si aplica]

## 6. Definición de Done del MVP

[Checklist de qué debe estar funcionando para considerar MVP completo]
```

## Validación Final

Antes de generar el documento, asegúrate de:

- [ ] Todas las US cubren SOLO el flujo de AI Interview Assistant (Caso de Uso 1)
- [ ] Todos los criterios de aceptación usan formato "Dado que-Cuando-Entonces"
- [ ] Las tareas son técnicas y específicas (no ambiguas)
- [ ] El backlog está priorizado claramente por valor de usuario
- [ ] Los tickets técnicos especifican arquitectura a nivel general (servicios, no implementación detallada)
- [ ] Las estimaciones Fibonacci tienen justificación
- [ ] No hay sobreingeniería ni funcionalidades fuera del flujo principal
- [ ] El documento es accionable para un equipo de desarrollo

## Ejecución

Genera el documento completo y escríbelo en `UserStories-AA.md`
