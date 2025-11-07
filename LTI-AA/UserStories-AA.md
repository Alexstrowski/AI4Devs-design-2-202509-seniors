# LTI ATS - Product Backlog MVP: AI Interview Assistant

## 1. Visión del MVP

El MVP de LTI se enfoca exclusivamente en validar la **propuesta de valor diferencial**: asistencia de IA durante entrevistas técnicas. Este flujo específico permite a startups con equipos pequeños realizar entrevistas más estructuradas, consistentes y objetivas sin aumentar la carga operativa de los reclutadores.

El objetivo es demostrar que la IA puede:
1. Reducir el tiempo de preparación de entrevistas en 80%
2. Eliminar la toma manual de notas
3. Generar evaluaciones estructuradas automáticamente
4. Sugerir preguntas de profundización relevantes

Este MVP se centra en el **Caso de Uso 1** del documento LTI-AA.md, implementando el flujo completo desde la carga del perfil hasta la generación del resumen post-entrevista.

---

## 2. User Stories Completas

### US-001: Carga de contexto pre-entrevista

**Como** reclutador
**Quiero** que el sistema cargue automáticamente el perfil del candidato y la descripción del puesto antes de iniciar la entrevista
**Para** tener toda la información relevante disponible sin buscarla manualmente

**Descripción:**
Cuando un reclutador inicia una sesión de entrevista, el sistema debe recuperar y mostrar el CV parseado del candidato junto con la job description del puesto. Esta información es la base para que la IA genere preguntas contextuales relevantes.

**Criterios de Aceptación:**
- [ ] **Dado que** existe un candidato con CV cargado y una posición activa asignada
      **Cuando** el reclutador selecciona "Iniciar Entrevista" desde el perfil del candidato
      **Entonces** el sistema muestra un panel con: nombre del candidato, resumen de experiencia, skills clave, y descripción del puesto
- [ ] **Dado que** el candidato no tiene CV parseado
      **Cuando** se intenta iniciar una entrevista
      **Entonces** el sistema muestra un error indicando que el CV debe ser procesado primero
- [ ] **Dado que** la carga de información tarda más de 2 segundos
      **Cuando** se está recuperando el contexto
      **Entonces** se muestra un indicador de progreso

**Notas Técnicas:**
- Core Service: maneja las entidades Candidate (parsed_resume JSON) y JobPosition (description)
- API Gateway: endpoint GET /interviews/prepare/{candidate_id}/{job_position_id}
- Database: consultas a tablas Candidate y JobPosition con JOIN a Application

**Tareas de Implementación:**
- [ ] Crear endpoint GET /api/interviews/prepare en Core Service
  - [ ] Validar existencia de candidate_id y job_position_id
  - [ ] Recuperar datos de BD con relaciones necesarias
- [ ] Implementar lógica de extracción de datos relevantes del parsed_resume JSON
- [ ] Crear componente frontend InterviewPreparationPanel
  - [ ] Mostrar información del candidato en formato legible
  - [ ] Mostrar descripción del puesto con skills requeridos destacados
- [ ] Implementar manejo de estados de carga y error
- [ ] Agregar tests de integración para validar estructura de response

**Definición de Done:**
- [ ] Código implementado y revisado
- [ ] Tests unitarios y de integración ejecutados exitosamente
- [ ] Documentación técnica actualizada

---

### US-002: Generación de guía de entrevista con IA

**Como** reclutador
**Quiero** que la IA genere automáticamente una guía de entrevista con preguntas estructuradas
**Para** tener un punto de partida profesional sin dedicar tiempo a preparar cada entrevista

**Descripción:**
Una vez cargado el contexto del candidato y puesto, el sistema solicita a la IA (OpenAI GPT-4) que genere 8-12 preguntas de entrevista categorizadas por competencia. Las preguntas deben ser específicas al perfil del candidato y alineadas con los requisitos del puesto.

**Criterios de Aceptación:**
- [ ] **Dado que** el contexto de la entrevista está cargado
      **Cuando** el sistema llama al AI Service para generar la guía
      **Entonces** recibe un JSON con 8-12 preguntas, cada una con: texto de pregunta, competencia evaluada, y nivel de dificultad
- [ ] **Dado que** la generación de la guía tarda más de 5 segundos
      **Cuando** se está generando
      **Entonces** se muestra un loader con mensaje "Generando guía personalizada..."
- [ ] **Dado que** las preguntas han sido generadas
      **Cuando** el reclutador visualiza la guía
      **Entonces** puede marcar preguntas como "usadas" o "saltadas" durante la entrevista
- [ ] **Dado que** el API de OpenAI falla
      **Cuando** se intenta generar la guía
      **Entonces** se usa un fallback con preguntas genéricas del puesto y se notifica al usuario

**Notas Técnicas:**
- AI Service: Interview Guide Controller (POST /ai/interview-guide)
- Integración externa: OpenAI API (modelo GPT-4)
- Prompt Engineering Module: construye el prompt con template + variables del candidato y job description
- Cache Manager: cachea la guía generada por 24h usando Redis (key: ai:guide:interview-{id})

**Tareas de Implementación:**
- [ ] Crear endpoint POST /ai/interview-guide en AI Service
  - [ ] Recibir interview_id y opcionalmente focus_areas
- [ ] Implementar Prompt Engineering Module
  - [ ] Crear template para generación de guías de entrevista
  - [ ] Incluir few-shot examples para mejorar calidad
- [ ] Integrar OpenAI Client con retry logic y rate limiting
- [ ] Implementar Cache Manager para almacenar guías generadas (TTL 24h)
- [ ] Crear componente frontend InterviewGuideDisplay
  - [ ] Mostrar preguntas categorizadas por competencia
  - [ ] Permitir marcar preguntas como usadas/saltadas
- [ ] Implementar fallback con preguntas genéricas del puesto
- [ ] Agregar logging de uso para tracking de costos de OpenAI

**Definición de Done:**
- [ ] Código implementado y revisado
- [ ] Tests unitarios y de integración ejecutados exitosamente
- [ ] Documentación técnica actualizada

---

### US-003: Transcripción de respuestas del candidato

**Como** reclutador
**Quiero** que el sistema transcriba automáticamente las respuestas del candidato durante la entrevista
**Para** concentrarme en la conversación sin tomar notas manualmente

**Descripción:**
Durante la entrevista, el reclutador puede grabar el audio de las respuestas del candidato. Al finalizar cada pregunta, el audio se envía al servicio de transcripción (OpenAI Whisper) que convierte el audio a texto. Este texto se almacena asociado a la pregunta correspondiente.

**Criterios de Aceptación:**
- [ ] **Dado que** el reclutador está en una entrevista activa
      **Cuando** hace clic en "Grabar Respuesta" y detiene la grabación después de que el candidato responde
      **Entonces** el audio se transcribe automáticamente y el texto aparece en el panel de notas
- [ ] **Dado que** el audio tiene una duración válida (entre 5 segundos y 10 minutos)
      **Cuando** se envía a transcripción
      **Entonces** el sistema procesa el audio y retorna el texto en menos de 30 segundos
- [ ] **Dado que** la transcripción está completa
      **Cuando** el reclutador revisa el texto
      **Entonces** puede editarlo manualmente si es necesario antes de guardarlo
- [ ] **Dado que** el formato de audio no es compatible
      **Cuando** se intenta transcribir
      **Entonces** se muestra un error indicando formatos aceptados (MP3, WAV, WebM)

**Notas Técnicas:**
- AI Service: Transcription Controller (POST /ai/transcribe)
- Integración externa: OpenAI Whisper API
- File Storage: AWS S3 para almacenar archivos de audio temporalmente
- Database: Interview entity tiene campo ai_transcript (Text) para guardar transcripciones

**Tareas de Implementación:**
- [ ] Crear endpoint POST /ai/transcribe en AI Service
  - [ ] Validar formato de audio (MP3, WAV, WebM)
  - [ ] Limitar duración máxima de audio (10 minutos)
- [ ] Integrar con OpenAI Whisper API
  - [ ] Enviar audio y recibir texto transcrito
  - [ ] Procesar response con timestamps si es posible
- [ ] Implementar subida temporal de audio a AWS S3
- [ ] Crear componente frontend AudioRecorder
  - [ ] Permitir grabación desde micrófono del navegador
  - [ ] Mostrar duración de grabación en tiempo real
  - [ ] Botones de iniciar/detener grabación
- [ ] Crear componente TranscriptionDisplay
  - [ ] Mostrar texto transcrito asociado a cada pregunta
  - [ ] Permitir edición manual del texto
- [ ] Implementar manejo de errores de formato y duración

**Definición de Done:**
- [ ] Código implementado y revisado
- [ ] Tests unitarios y de integración ejecutados exitosamente
- [ ] Documentación técnica actualizada

---

### US-004: Sugerencias de preguntas de profundización con IA

**Como** reclutador
**Quiero** recibir sugerencias de preguntas de profundización basadas en las respuestas del candidato
**Para** explorar áreas interesantes sin tener que pensar las preguntas en el momento

**Descripción:**
Después de que el candidato responde una pregunta y esta es transcrita, la IA analiza la respuesta y sugiere 2-3 follow-up questions contextuales. Estas sugerencias aparecen en tiempo real para que el reclutador pueda decidir si las usa o continúa con su propio criterio.

**Criterios de Aceptación:**
- [ ] **Dado que** una respuesta del candidato ha sido transcrita
      **Cuando** el sistema analiza el contenido de la respuesta
      **Entonces** genera 2-3 preguntas de follow-up relevantes en menos de 3 segundos
- [ ] **Dado que** las preguntas sugeridas están disponibles
      **Cuando** el reclutador las visualiza
      **Entonces** puede hacer clic en cualquiera para marcarla como "usada" o ignorarlas y continuar
- [ ] **Dado que** la respuesta del candidato menciona tecnologías o experiencias específicas
      **Cuando** se generan las sugerencias
      **Entonces** las preguntas deben profundizar en esos puntos específicos
- [ ] **Dado que** el API de OpenAI tiene latencia alta (>5 segundos)
      **Cuando** se están generando sugerencias
      **Entonces** se muestra un indicador de carga sin bloquear el flujo de la entrevista

**Notas Técnicas:**
- AI Service: Realtime Assistant Controller (POST /ai/realtime-suggest)
- Integración externa: OpenAI API (GPT-4 con streaming si es posible)
- Prompt Engine: construye prompt con contexto de las últimas 5 preguntas/respuestas
- Rate limiting: máximo 10 requests/minuto por interview para proteger costos

**Tareas de Implementación:**
- [ ] Crear endpoint POST /ai/realtime-suggest en AI Service
  - [ ] Recibir interview_id y candidate_last_response
  - [ ] Recuperar contexto de conversación (últimas 5 Q&A)
- [ ] Implementar lógica de análisis contextual en Prompt Engine
  - [ ] Identificar señales de profundización en la respuesta
  - [ ] Construir prompt con historial de conversación
- [ ] Integrar con OpenAI API usando streaming (Server-Sent Events)
- [ ] Implementar rate limiting (10 req/min por interview)
- [ ] Crear componente frontend FollowUpSuggestions
  - [ ] Mostrar sugerencias en panel lateral durante entrevista
  - [ ] Permitir marcar como "usada" al hacer clic
  - [ ] Mostrar loader mientras se generan
- [ ] Agregar tracking de analytics (sugerencias aceptadas vs rechazadas)

**Definición de Done:**
- [ ] Código implementado y revisado
- [ ] Tests unitarios y de integración ejecutados exitosamente
- [ ] Documentación técnica actualizada

---

### US-005: Finalización y resumen automático de entrevista

**Como** reclutador
**Quiero** que al finalizar la entrevista el sistema genere automáticamente un resumen estructurado
**Para** tener una evaluación documentada sin dedicar tiempo a escribir reportes

**Descripción:**
Cuando el reclutador finaliza la entrevista, el sistema procesa toda la transcripción y genera un resumen estructurado que incluye: impresión general, competencias evaluadas con rating, fortalezas, preocupaciones, recomendación (Yes/Maybe/No), y citas textuales clave. Este resumen se guarda en el perfil del candidato.

**Criterios de Aceptación:**
- [ ] **Dado que** la entrevista tiene al menos 3 preguntas respondidas y transcritas
      **Cuando** el reclutador hace clic en "Finalizar Entrevista"
      **Entonces** el sistema genera un resumen estructurado en formato JSON en menos de 60 segundos
- [ ] **Dado que** el resumen ha sido generado
      **Cuando** se muestra al reclutador
      **Entonces** incluye: impresión general, lista de competencias con rating 1-5, fortalezas, preocupaciones, recomendación, y al menos 2 citas textuales
- [ ] **Dado que** el resumen está disponible
      **Cuando** el reclutador lo revisa
      **Entonces** puede editarlo antes de guardarlo definitivamente en el perfil del candidato
- [ ] **Dado que** la generación del resumen falla
      **Cuando** ocurre un error en el AI Service
      **Entonces** se guarda la transcripción completa y se permite al reclutador crear un resumen manual

**Notas Técnicas:**
- AI Service: Summary Controller (POST /ai/summarize)
- Integración externa: OpenAI API (GPT-4)
- Competency Extractor: identifica skills técnicos y soft skills mencionados
- Database: Interview entity, campo ai_summary (JSON column)
- Core Service: actualiza el estado de la entrevista a "Completed"

**Tareas de Implementación:**
- [ ] Crear endpoint POST /ai/summarize en AI Service
  - [ ] Recibir interview_id
  - [ ] Obtener transcripción completa de la entrevista
- [ ] Implementar Competency Extractor
  - [ ] Usar regex patterns + keyword matching para identificar skills
  - [ ] Scoring basado en frecuencia y contexto
- [ ] Construir prompt de análisis en Prompt Engine
  - [ ] Incluir job description como referencia
  - [ ] Definir estructura JSON del resumen
- [ ] Integrar con OpenAI API para generación del resumen
- [ ] Guardar resultado en Interview.ai_summary (JSON column)
- [ ] Crear componente frontend InterviewSummaryDisplay
  - [ ] Mostrar resumen estructurado con secciones claras
  - [ ] Permitir edición antes de guardar definitivamente
  - [ ] Visualizar competencias con rating visual (estrellas o barras)
- [ ] Implementar endpoint PATCH /api/interviews/{id}/summary para guardar ediciones
- [ ] Agregar fallback para crear resumen manual si IA falla

**Definición de Done:**
- [ ] Código implementado y revisado
- [ ] Tests unitarios y de integración ejecutados exitosamente
- [ ] Documentación técnica actualizada

---

### US-006: Almacenamiento y visualización del historial de entrevista

**Como** reclutador
**Quiero** que toda la información de la entrevista (guía, transcripciones, resumen) se guarde automáticamente en el perfil del candidato
**Para** poder consultarla posteriormente y compartirla con mi equipo

**Descripción:**
Todo el flujo de la entrevista (guía generada, preguntas realizadas, respuestas transcritas, sugerencias usadas, y resumen final) debe persistirse en la base de datos asociado a la entidad Interview. Los reclutadores y hiring managers deben poder acceder a este historial completo desde el perfil del candidato.

**Criterios de Aceptación:**
- [ ] **Dado que** una entrevista ha sido finalizada
      **Cuando** se guarda el resumen
      **Entonces** todos los datos se persisten en BD: ai_transcript, ai_summary, y estado cambia a "Completed"
- [ ] **Dado que** un usuario con permisos adecuados accede al perfil del candidato
      **Cuando** visualiza la sección "Entrevistas"
      **Entonces** puede ver lista de todas las entrevistas realizadas con fecha, entrevistador, y estado
- [ ] **Dado que** el usuario hace clic en una entrevista pasada
      **Cuando** se abre el detalle
      **Entonces** puede ver: guía de preguntas usada, transcripción completa, y resumen estructurado
- [ ] **Dado que** el usuario es un Hiring Manager (no el entrevistador original)
      **Cuando** accede al historial
      **Entonces** tiene permisos de solo lectura sin poder editar

**Notas Técnicas:**
- Database: Interview entity con campos ai_transcript (Text), ai_summary (JSON), status (Enum)
- Core Service: endpoint GET /api/candidates/{id}/interviews para listar entrevistas
- Core Service: endpoint GET /api/interviews/{id}/detail para ver detalle completo
- API Gateway: validación de permisos según User.role

**Tareas de Implementación:**
- [ ] Crear endpoints en Core Service
  - [ ] GET /api/candidates/{id}/interviews (lista de entrevistas)
  - [ ] GET /api/interviews/{id}/detail (detalle completo de entrevista)
- [ ] Implementar lógica de permisos en API Gateway
  - [ ] Validar que el usuario tenga acceso al candidato
  - [ ] Diferenciar permisos de lectura vs edición según rol
- [ ] Crear componente frontend InterviewHistoryList
  - [ ] Listar entrevistas con fecha, entrevistador, estado
  - [ ] Permitir clic para ver detalle
- [ ] Crear componente InterviewDetailView
  - [ ] Mostrar guía de preguntas usada
  - [ ] Mostrar transcripción organizada por pregunta-respuesta
  - [ ] Mostrar resumen estructurado
  - [ ] Deshabilitar edición si usuario no es el entrevistador original
- [ ] Agregar tests de permisos y acceso a datos

**Definición de Done:**
- [ ] Código implementado y revisado
- [ ] Tests unitarios y de integración ejecutados exitosamente
- [ ] Documentación técnica actualizada

---

### US-007: Gestión básica de candidatos y posiciones

**Como** reclutador
**Quiero** poder crear candidatos y posiciones de trabajo de forma simple
**Para** tener la información base necesaria antes de iniciar entrevistas

**Descripción:**
El sistema debe permitir CRUD básico de candidatos (nombre, email, CV upload) y posiciones de trabajo (título, descripción, skills requeridos). Esta funcionalidad es el foundation necesario para poder ejecutar el flujo de entrevistas, pero no incluye features avanzados como pipeline visual o filtrado complejo.

**Criterios de Aceptación:**
- [ ] **Dado que** un reclutador accede a la sección "Candidatos"
      **Cuando** hace clic en "Agregar Candidato"
      **Entonces** puede ingresar: nombre, apellido, email, teléfono, LinkedIn URL, y subir CV (PDF o DOCX)
- [ ] **Dado que** un CV es subido
      **Cuando** el sistema lo procesa
      **Entonces** extrae automáticamente skills, experiencia, y educación en el campo parsed_resume (JSON)
- [ ] **Dado que** un reclutador accede a la sección "Posiciones"
      **Cuando** hace clic en "Crear Posición"
      **Entonces** puede ingresar: título, descripción, departamento, tipo de empleo, seniority, y skills requeridos
- [ ] **Dado que** un candidato y una posición existen
      **Cuando** el reclutador asocia el candidato a la posición
      **Entonces** se crea una Application que los vincula

**Notas Técnicas:**
- Core Service: maneja entidades Candidate, JobPosition, Application
- File Storage: AWS S3 para almacenar CVs
- CV Parsing: librería externa para extraer información del PDF (ej: Textract o similar)
- Database: tablas Candidate, JobPosition, Application con relaciones FK

**Tareas de Implementación:**
- [ ] Crear endpoints CRUD en Core Service
  - [ ] POST /api/candidates (crear candidato)
  - [ ] GET /api/candidates (listar candidatos)
  - [ ] POST /api/job-positions (crear posición)
  - [ ] GET /api/job-positions (listar posiciones)
  - [ ] POST /api/applications (asociar candidato a posición)
- [ ] Implementar subida de CV a AWS S3
- [ ] Integrar librería de CV parsing
  - [ ] Extraer texto del PDF
  - [ ] Parsear skills, experiencia, educación
  - [ ] Guardar en campo parsed_resume como JSON
- [ ] Crear componentes frontend
  - [ ] CandidateForm (formulario de creación/edición)
  - [ ] CandidateList (lista de candidatos)
  - [ ] JobPositionForm (formulario de creación/edición)
  - [ ] JobPositionList (lista de posiciones)
- [ ] Implementar validaciones básicas (email único, campos requeridos)
- [ ] Agregar tests de CRUD completo

**Definición de Done:**
- [ ] Código implementado y revisado
- [ ] Tests unitarios y de integración ejecutados exitosamente
- [ ] Documentación técnica actualizada

---

## 3. Product Backlog Priorizado

| ID | User Story | Prioridad | Valor para Usuario | Dependencias |
|----|-----------|-----------|-------------------|--------------|
| US-007 | Gestión básica de candidatos y posiciones | P0 | Foundation indispensable: sin candidatos y posiciones no hay entrevistas que realizar | - |
| US-001 | Carga de contexto pre-entrevista | P0 | Permite al reclutador tener información relevante antes de empezar, reduciendo tiempo de preparación manual | US-007 |
| US-002 | Generación de guía de entrevista con IA | P0 | Valor core del producto: elimina 80% del tiempo de preparación y estandariza entrevistas | US-001 |
| US-003 | Transcripción de respuestas del candidato | P0 | Elimina toma manual de notas, permitiendo al reclutador enfocarse en la conversación | US-002 |
| US-005 | Finalización y resumen automático de entrevista | P0 | Genera documentación estructurada automáticamente, validando la propuesta de valor principal | US-003 |
| US-004 | Sugerencias de preguntas de profundización con IA | P1 | Mejora calidad de entrevistas pero el flujo funciona sin esta feature. Diferenciador competitivo importante. | US-003 |
| US-006 | Almacenamiento y visualización del historial de entrevista | P1 | Permite consultar información posteriormente pero no es crítico para el flujo principal de entrevista | US-005 |

---

## 4. Desglose Técnico: US-002 - Generación de guía de entrevista con IA

### Contexto

Esta es la User Story **P0 con mayor valor para el usuario** porque representa el diferenciador core de LTI: la capacidad de generar automáticamente guías de entrevista personalizadas usando IA. Esta funcionalidad valida directamente la propuesta de valor de reducir el tiempo de preparación en 80% y estandarizar entrevistas.

Sin esta feature, LTI sería simplemente otro ATS genérico. La generación de guías con IA es lo que permite a startups con equipos pequeños realizar entrevistas profesionales sin dedicar horas a preparación.

### Tickets Técnicos

---

### TICKET-001: Implementar endpoint de generación de guía en AI Service

**Tipo:** Backend (AI/ML)
**User Story:** US-002
**Estimación Fibonacci:** 5

**Descripción Técnica:**
Crear endpoint POST /ai/interview-guide en el AI Service (Python/FastAPI) que reciba un interview_id, recupere el contexto necesario (job description + candidato), y orqueste la llamada a OpenAI GPT-4 para generar preguntas estructuradas.

Input esperado:
```json
{
  "interview_id": "uuid",
  "focus_areas": ["technical", "leadership"]
}
```

Output esperado:
```json
{
  "questions": [
    {
      "id": "q1",
      "text": "Describe una situación donde...",
      "competency": "Problem Solving",
      "difficulty": "medium"
    }
  ]
}
```

**Alcance a Nivel Arquitectónico:**
- **Servicios involucrados:** AI Service (FastAPI), Core Service (para obtener datos), API Gateway (routing)
- **Integraciones externas:** OpenAI API (modelo GPT-4)
- **Capa de datos:** Interview, Candidate (parsed_resume), JobPosition (description)
- **Frontend:** N/A

**Criterios Técnicos de Aceptación:**
- [ ] Endpoint responde con status 200 y JSON estructurado cuando datos son válidos
- [ ] Endpoint responde con status 404 si interview_id no existe
- [ ] Endpoint responde con status 500 y mensaje descriptivo si OpenAI API falla
- [ ] Response time < 10 segundos en el 95% de los casos

**Dependencias Técnicas:**
- Database schema con tablas Interview, Candidate, JobPosition debe existir
- Cuenta de OpenAI API con API key configurada
- Core Service debe tener endpoint para obtener datos de interview

**Notas de Implementación:**
Usar biblioteca oficial de OpenAI Python SDK. Implementar timeout de 15 segundos para llamadas a OpenAI. Considerar que el parsing del JSON response puede requerir validación robusta ya que GPT-4 ocasionalmente genera JSON mal formateado.

---

### TICKET-002: Desarrollar Prompt Engineering Module

**Tipo:** Backend (AI/ML)
**User Story:** US-002
**Estimación Fibonacci:** 3

**Descripción Técnica:**
Crear módulo Python en el AI Service que construya prompts estructurados usando templates y variables del contexto. Debe incluir few-shot examples para mejorar consistencia del output de GPT-4.

Input: datos del candidato (parsed_resume) y job description
Output: string con prompt completo listo para enviar a OpenAI

**Alcance a Nivel Arquitectónico:**
- **Servicios involucrados:** AI Service (módulo interno)
- **Integraciones externas:** N/A
- **Capa de datos:** Lee datos pero no escribe en BD
- **Frontend:** N/A

**Criterios Técnicos de Aceptación:**
- [ ] El prompt incluye: job title, required skills, candidate background summary
- [ ] El prompt especifica formato JSON de salida esperado
- [ ] El prompt incluye 2-3 ejemplos de preguntas bien formateadas (few-shot)
- [ ] El módulo es testeable unitariamente sin hacer llamadas reales a OpenAI

**Dependencias Técnicas:**
- TICKET-001 debe estar en progreso (usa este módulo)

**Notas de Implementación:**
Usar Python f-strings o Jinja2 para templates. Centralizar todos los prompts en un archivo de configuración para facilitar iteración. Considerar agregar parámetro de temperatura para controlar creatividad del modelo.

---

### TICKET-003: Implementar OpenAI Client con retry logic

**Tipo:** Backend (Infrastructure)
**User Story:** US-002
**Estimación Fibonacci:** 3

**Descripción Técnica:**
Crear wrapper robusto del OpenAI Python SDK que maneje retry con exponential backoff, rate limiting, y logging de uso. Debe ser reutilizable para todas las integraciones con OpenAI en el AI Service.

**Alcance a Nivel Arquitectónico:**
- **Servicios involucrados:** AI Service (módulo interno compartido)
- **Integraciones externas:** OpenAI API
- **Capa de datos:** Logs de uso en base de datos (opcional para MVP)
- **Frontend:** N/A

**Criterios Técnicos de Aceptación:**
- [ ] Implementa retry automático con exponential backoff (max 3 intentos)
- [ ] Maneja errores de rate limit (HTTP 429) esperando el tiempo indicado por OpenAI
- [ ] Logea cada llamada con: timestamp, modelo usado, tokens consumidos, latencia
- [ ] Lanza excepciones custom descriptivas cuando falla después de todos los retries

**Dependencias Técnicas:**
- TICKET-001 requiere este módulo

**Notas de Implementación:**
Usar biblioteca `tenacity` para retry logic. Configurar timeout de 15 segundos por request. En MVP el logging puede ser a stdout, pero diseñar para permitir logging a BD en futuro.

---

### TICKET-004: Implementar Cache Manager con Redis

**Tipo:** Backend (Infrastructure)
**User Story:** US-002
**Estimación Fibonacci:** 5

**Descripción Técnica:**
Implementar sistema de cache usando Redis para almacenar guías de entrevista generadas. Key structure: `ai:guide:interview-{id}`. TTL de 24 horas. Esto reduce costos de OpenAI evitando regenerar guías idénticas.

**Alcance a Nivel Arquitectónico:**
- **Servicios involucrados:** AI Service, Redis (cache layer)
- **Integraciones externas:** Redis (puede ser ElastiCache en AWS)
- **Capa de datos:** Cache layer (no BD relacional)
- **Frontend:** N/A

**Criterios Técnicos de Aceptación:**
- [ ] Antes de llamar a OpenAI, verifica si existe cache hit para ese interview_id
- [ ] Si cache hit, retorna resultado en <100ms
- [ ] Después de generar guía exitosamente, la almacena en cache con TTL 24h
- [ ] Maneja gracefully el caso donde Redis no está disponible (bypass cache, no falla)

**Dependencias Técnicas:**
- Redis debe estar configurado y accesible desde AI Service
- TICKET-001 debe integrar este módulo

**Notas de Implementación:**
Usar biblioteca `redis-py`. Serializar JSON antes de guardar en Redis. Implementar health check para validar conexión a Redis. Considerar que en MVP puede ser Redis local, pero diseñar para ElastiCache en producción.

---

### TICKET-005: Crear componente frontend InterviewGuideDisplay

**Tipo:** Frontend
**User Story:** US-002
**Estimación Fibonacci:** 5

**Descripción Técnica:**
Implementar componente React que muestre la guía de entrevista generada. Debe permitir al reclutador ver preguntas categorizadas por competencia y marcarlas como "usadas" o "saltadas" durante la entrevista.

**Alcance a Nivel Arquitectónico:**
- **Servicios involucrados:** Frontend (React), API Gateway
- **Integraciones externas:** N/A
- **Capa de datos:** Consume endpoint GET /ai/interview-guide/{interview_id}
- **Frontend:** Componente InterviewGuideDisplay con estado local

**Criterios Técnicos de Aceptación:**
- [ ] Muestra loader mientras se genera la guía (puede tomar 5-10 segundos)
- [ ] Agrupa preguntas por competencia (ej: "Problem Solving", "Leadership")
- [ ] Cada pregunta tiene checkbox o botón para marcar como "usada"/"saltada"
- [ ] Estado de preguntas marcadas persiste en localStorage (no crítico en servidor para MVP)
- [ ] Muestra error message si la generación falla, con opción de retry

**Dependencias Técnicas:**
- TICKET-001 debe estar completo (API disponible)
- Componente InterviewPreparationPanel de US-001 debe existir (integración visual)

**Notas de Implementación:**
Usar React Query para manejo de estado de loading/error. Considerar diseño mobile-friendly ya que reclutadores pueden usar tablets. Las marcas de "usada"/"saltada" pueden guardarse en localStorage para MVP, en v1.1 se pueden persistir en servidor.

---

### TICKET-006: Implementar fallback con preguntas genéricas

**Tipo:** Backend
**User Story:** US-002
**Estimación Fibonacci:** 2

**Descripción Técnica:**
Crear conjunto de preguntas genéricas por tipo de puesto (ej: "Software Engineer", "Product Manager") que se usan como fallback cuando OpenAI API falla. Deben estar hardcodeadas en el Core Service.

**Alcance a Nivel Arquitectónico:**
- **Servicios involucrados:** Core Service o AI Service
- **Integraciones externas:** N/A
- **Capa de datos:** Puede ser JSON file o tabla en BD
- **Frontend:** N/A

**Criterios Técnicos de Aceptación:**
- [ ] Contiene al menos 10 preguntas genéricas para "Software Engineer"
- [ ] Preguntas siguen la misma estructura JSON que las generadas por IA
- [ ] El endpoint de guía retorna estas preguntas con un flag `is_fallback: true`
- [ ] Frontend muestra un warning al reclutador cuando se usan preguntas de fallback

**Dependencias Técnicas:**
- TICKET-001 debe integrar este fallback

**Notas de Implementación:**
Para MVP, suficiente con un JSON file con preguntas. Priorizar "Software Engineer" ya que es el rol más común en startups tech. En futuro se pueden agregar más roles.

---

### TICKET-007: Agregar logging y monitoring de costos de OpenAI

**Tipo:** Backend (Infrastructure)
**User Story:** US-002
**Estimación Fibonacci:** 2

**Descripción Técnica:**
Implementar logging estructurado de cada llamada a OpenAI API con: timestamp, interview_id, modelo usado, tokens consumidos, latencia, y costo estimado. Esto permite monitorear costos en tiempo real.

**Alcance a Nivel Arquitectónico:**
- **Servicios involucrados:** AI Service
- **Integraciones externas:** OpenAI API (para obtener token usage)
- **Capa de datos:** Logs (stdout para MVP, puede ir a CloudWatch o similar)
- **Frontend:** N/A

**Criterios Técnicos de Aceptación:**
- [ ] Cada llamada a OpenAI logea: timestamp, interview_id, tokens_used, latency_ms, estimated_cost
- [ ] Los logs usan formato estructurado (JSON) para fácil parsing
- [ ] Los logs incluyen nivel de log (INFO para success, ERROR para failures)
- [ ] Costo estimado se calcula basado en pricing público de OpenAI

**Dependencias Técnicas:**
- TICKET-003 (OpenAI Client) debe implementar este logging

**Notas de Implementación:**
Usar biblioteca `structlog` para logging estructurado. Los costos de OpenAI son: $0.03/1K tokens input, $0.06/1K tokens output para GPT-4. En MVP logs van a stdout, pero diseñar para integrar con CloudWatch Logs en futuro.

---

### Dependencias entre Tickets

**Orden de implementación recomendado:**

1. **Fase 1 - Infrastructure** (pueden ser en paralelo)
   - TICKET-003: OpenAI Client con retry logic
   - TICKET-004: Cache Manager con Redis

2. **Fase 2 - Backend Core**
   - TICKET-002: Prompt Engineering Module (depende de conocer estructura de datos)
   - TICKET-001: Endpoint de generación de guía (depende de TICKET-002, TICKET-003, TICKET-004)
   - TICKET-006: Fallback con preguntas genéricas (puede ser en paralelo con TICKET-001)

3. **Fase 3 - Observability**
   - TICKET-007: Logging y monitoring (depende de TICKET-003)

4. **Fase 4 - Frontend**
   - TICKET-005: Componente InterviewGuideDisplay (depende de TICKET-001 completo)

**Diagrama de dependencias:**
```
TICKET-003   ,  > TICKET-001   > TICKET-005
                      ²
               > TICKET-007

TICKET-004   
TICKET-002   
TICKET-006   
```

---

## 5. Estimación y Planning

### Tabla de Estimaciones

| Ticket ID | Título | Puntos | Justificación de Estimación |
|-----------|--------|--------|----------------------------|
| TICKET-001 | Endpoint de generación de guía en AI Service | 5 | Complejidad media: requiere coordinación entre AI Service y Core Service, integración con OpenAI, y manejo de errores. Tiempo estimado: 2-3 días. |
| TICKET-002 | Prompt Engineering Module | 3 | Tarea estándar: crear templates y validar outputs. Requiere iteración para afinar prompts. Tiempo estimado: 1 día. |
| TICKET-003 | OpenAI Client con retry logic | 3 | Tarea estándar: wrapper con retry y error handling. Biblioteca `tenacity` simplifica implementación. Tiempo estimado: 1 día. |
| TICKET-004 | Cache Manager con Redis | 5 | Complejidad media: integración con Redis, manejo de serialización, health checks, y testing. Tiempo estimado: 2-3 días. |
| TICKET-005 | Componente frontend InterviewGuideDisplay | 5 | Complejidad media: componente React con múltiples estados (loading, success, error), integración con API, y UX de marcar preguntas. Tiempo estimado: 2-3 días. |
| TICKET-006 | Fallback con preguntas genéricas | 2 | Tarea simple: crear JSON con preguntas y modificar lógica del endpoint. Tiempo estimado: 4 horas. |
| TICKET-007 | Logging y monitoring de costos | 2 | Tarea simple: agregar logging estructurado a OpenAI Client. Tiempo estimado: 4 horas. |

### Resumen

- **Total Story Points:** 25 puntos
- **Esfuerzo estimado:** ~2.5 semanas para un equipo de 2-3 developers (1 backend, 1 frontend, 1 infraestructura/fullstack)
- **Velocidad esperada:** Asumiendo velocidad de 10-12 puntos por semana por equipo

### Consideraciones de Capacity

- **Riesgos técnicos:**
  - Latencia de OpenAI API puede variar (95th percentile ~10 segundos)
  - Calidad de prompts requiere iteración y testing con casos reales
  - Rate limits de OpenAI pueden impactar en testing intensivo

- **Dependencias externas críticas:**
  - Cuenta de OpenAI con límites de rate y budget adecuados
  - Redis configurado (puede ser local para desarrollo)
  - AWS S3 para almacenar CVs (US-007 debe estar completo)

- **Paralelización:**
  - Backend y Frontend pueden trabajar en paralelo después de definir contrato de API
  - Infraestructura (TICKET-003, TICKET-004) puede iniciar inmediatamente

---

## 6. Definición de Done del MVP

El MVP de LTI - AI Interview Assistant se considera **completo** cuando:

### Funcionalidad Core
- [ ] Un reclutador puede crear un candidato con CV y una posición de trabajo (US-007)
- [ ] El sistema carga automáticamente el contexto (candidato + job description) al iniciar entrevista (US-001)
- [ ] La IA genera una guía de entrevista personalizada con 8-12 preguntas en menos de 10 segundos (US-002)
- [ ] El reclutador puede grabar respuestas del candidato y el sistema las transcribe automáticamente (US-003)
- [ ] La IA sugiere preguntas de profundización basadas en las respuestas del candidato (US-004)
- [ ] Al finalizar, el sistema genera un resumen estructurado automático con competencias, fortalezas, y recomendación (US-005)
- [ ] Todo el historial de la entrevista es accesible desde el perfil del candidato (US-006)

### Calidad Técnica
- [ ] Todos los endpoints críticos tienen tests de integración con coverage >80%
- [ ] El sistema maneja gracefully errores de OpenAI API (fallback a preguntas genéricas)
- [ ] El frontend muestra estados de loading y errores de forma clara
- [ ] Los costos de OpenAI por entrevista son monitoreados y no exceden $2 USD por sesión

### Experiencia de Usuario
- [ ] Un reclutador nuevo puede completar su primera entrevista asistida en menos de 30 minutos sin documentación
- [ ] El flujo completo desde inicio de entrevista hasta resumen final funciona sin requerer más de 2 minutos de espera total (excluyendo duración de la entrevista)
- [ ] El resumen generado es suficientemente preciso que el reclutador solo necesita editar <20% del contenido

### Infraestructura
- [ ] El sistema está deployed en un ambiente accesible vía HTTPS
- [ ] OpenAI API key está configurada con rate limits adecuados
- [ ] Redis está corriendo y el cache funciona correctamente
- [ ] AWS S3 está configurado para almacenar CVs

### Documentación
- [ ] Existe documentación de API para todos los endpoints públicos
- [ ] Existe README con instrucciones de setup local para developers
- [ ] Existe guía de usuario básica explicando el flujo de entrevista

### Validación de Negocio
- [ ] Al menos 3 entrevistas reales han sido conducidas usando el sistema end-to-end
- [ ] Los reclutadores confirman que el tiempo de preparación se redujo comparado con su proceso manual anterior
- [ ] Los resúmenes generados son considerados útiles y precisos por los reclutadores
