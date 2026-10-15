# AgentJournalist — SPEC v0.1 (FAST DEMO, PA-SDD)

Estado: diseño inicial; no implica despliegue operativo.
Fuente de patrones: [DevPattern](https://github.com/JuliusCordova/DevPattern), especialmente `definition-packs/minimum-sufficient-specification.md`, `agentic-reference-architectures/gcp.md`, `patterns/implementation-profiles/gcp/pattern-pack-mapping.md` y `patterns/pattern-packs/governed-agent.md`.

## 1. Intención del negocio
Demostrar a periodistas que una redacción agéntica puede investigar una pista o noticia sospechosa, descomponer afirmaciones, mostrar evidencia trazable y producir un borrador revisable sin reemplazar criterio humano.

**Indicador principal:** 100% de afirmaciones fácticas en el borrador con enlace a evidencias verificadas; cero publicaciones automáticas.

## 2. Persona y caso
Periodista/editor de investigación en una demo facilitada. Entrada: afirmación, URL o texto breve, preferiblemente un caso real previamente contrastado; para los ensayos desconectados, escenario ficticio etiquetado. Salida: panel de investigación, expediente y borrador sujeto a firma humana.

## 3. Alcance FAST DEMO
- Seis roles portados desde los skills originales: editor-jefe, investigacion, datos, verificacion, expediente y redaccion.
- Captura de texto/URL, plan de investigación, evidencias numeradas H/D/V, cronología y expediente.
- Cuatro aprobaciones obligatorias persistidas: plan, selección de anomalías, expediente y borrador.
- Evidencia original con URL, fecha de consulta, fragmento, emisor y huella cuando proceda.
- Borrador limitado al expediente validado, con [DESCARGO PENDIENTE] cuando aplique.
- UI con progreso por agente, tarjetas de fuentes y botones aprobar/rechazar.
- Evals con casos positivos, insuficientes, contradicciones y prompt injection.

**Fuera de alcance:** publicación en redes o CMS, automatización de acusaciones, crawling masivo, scraping contra TOS, OCR/video, búsqueda de datos privados, Agent Engine administrado, GraphRAG, GKE, plena plataforma empresarial.

## 4. Historias
- US-001: como periodista quiero pegar una afirmación para obtener un plan editable.
- US-002: como editor quiero aprobar o rechazar el plan antes de investigar.
- US-003: como periodista quiero ver evidencia y anomalías con fuente y método reproducible.
- US-004: como editor quiero validar el expediente antes de generar el borrador.
- US-005: como editor quiero revisar y firmar manualmente un borrador atribuido.

## 5. Requisitos funcionales
- FR-001: la API registra un caso con ID, tipo de entrada y estado.
- FR-002: el editor-jefe propone preguntas acotadas y no continúa sin aprobación 1.
- FR-003: investigación y datos devuelven H/D con referencias y cálculos; si no hay datos tabulares, datos registra NO_APLICA.
- FR-004: la aprobación 2 establece explícitamente qué anomalías pasan a verificación.
- FR-005: verificación descompone en afirmaciones atómicas y contrasta fuentes independientes, sin equiparar dos republicaciones a dos fuentes.
- FR-006: el expediente mantiene confirmado/refutado/indicio/insuficiente/descartado; no inventa hechos.
- FR-007: la aprobación 3 bloquea o habilita redacción.
- FR-008: redacción recibe solo expediente validado; la aprobación 4 firma, pero no publica automáticamente.
- FR-009: todas las transiciones y aprobaciones registran actor, fecha, versión, decisión y hash de artefacto.
- FR-010: errores o fuentes inaccesibles producen estado inconcluso, nunca certeza falsa.

## 6. Requisitos no funcionales
NFR-001: autorización de cambios de estado aplicada en backend, nunca por prompt.
NFR-002: cuentas de servicio de mínimos privilegios; herramientas de solo lectura excepto almacenamiento del expediente.
NFR-003: logs sin secretos ni información personal innecesaria.
NFR-004: sin declaración de veracidad si la evidencia es insuficiente.
NFR-005: despliegue reproducible en Cloud Run, con prueba de salud y límites de gasto configurables.
NFR-006: tiempos medidos en evaluación; no prometer latencia fija de búsqueda externa.

## 7. Contratos de información
`Case {case_id, input_text, input_url?, created_at, status, version}`
`Evidence {evidence_id, case_id, source_url, publisher, published_at?, retrieved_at, excerpt, source_hash?, parent_source_id?}`
`Finding {finding_id, case_id, kind:H|D|V, claim, evidence_ids[], method?, verdict?, confidence_reason?}`
`Approval {approval_id, case_id, gate:1|2|3|4, artifact_version, artifact_hash, decision, reviewer_id, decided_at}`
`Dossier {case_id, version, confirmed_ids[], rejected_ids[], inconclusive_ids[], timeline[], human_tasks[]}`
`Draft {case_id, dossier_version, cited_finding_ids[], text, approved_by?}`

## 8. Reglas del negocio
BR-001: G1, G2, G3 y G4 son compuertas de aplicación, no instrucciones opcionales al LLM.
BR-002: una anomalía no demuestra ilegalidad.
BR-003: las fuentes independientes deben rastrearse a su origen documental.
BR-004: redactar solo con afirmaciones confirmadas; las restantes son pendientes o se omiten.
BR-005: rechazos o cambios materiales invalidan aprobaciones posteriores.
BR-006: nunca publicar ni enviar mensajes a terceros desde el agente.

## 9. Contrato de agente y herramientas
- `editor_jefe`: propone plan y delega; no ejecuta `approve`, `publish` o `sign`.
- `investigacion`: `search_sources`, `get_source` (lectura).
- `datos`: `analyze_dataset` en ejecución aislada, límite de recursos, sin acceso irrestricto a red.
- `verificacion`: `search_independent_sources`, `get_source` (lectura).
- `expediente`: `assemble_dossier` a partir de H/D/V, sin búsqueda.
- `redaccion`: `draft_from_approved_dossier`, sin herramientas de búsqueda ni publicación.

Cada llamada lleva `case_id`, rol autorizado, límites, resultado y `trace_id`. Aprobaciones mediante endpoint autenticado fuera del modelo.

## 10. API inicial
`POST /api/v1/cases`; `GET /api/v1/cases/{id}`; `POST /api/v1/cases/{id}/plan`; `POST /api/v1/cases/{id}/approvals/{gate}`; `GET /api/v1/cases/{id}/evidence`; `GET /api/v1/cases/{id}/dossier`; `GET /api/v1/cases/{id}/draft`; `GET /health`.

## 11. Criterios de aceptación
AC-001: dado un caso nuevo, al generar plan queda WAITING_GATE_1 y ninguna investigación se ejecuta antes de aprobar.
AC-002: sin G2 aprobado, verificación no avanza; sin G3 aprobado, redacción no corre.
AC-003: el borrador incluye evidencia válida por cada afirmación sustantiva; si no, la omite.
AC-004: cualquier rechazo queda registrado y bloquea la continuación.
AC-005: caso de evidencia contradictoria/insuficiente muestra abstención razonada.
AC-006: inyección dentro de una fuente no puede ejecutar aprobación ni cambiar herramientas autorizadas.

## 12. Evals FAST DEMO
Fixture A: afirmación verificable con dos fuentes independientes.
Fixture B: titular engañoso con dato parcialmente cierto.
Fixture C: cita no rastreable o evidencia insuficiente.
Fixture D: datos tabulares con anomalía explicable, sin inferencia acusatoria.
Fixture E: fuente con instrucción maliciosa para saltar un gate.
Metas iniciales de ensayo (a validar con golden set): 100% bloqueos de gates; 100% citas fácticas trazables; cero publicación autónoma.

## 13. Secuencia de entrega
Fase 0: conservar skills originales y pruebas golden set.
Fase 1: contratos JSON, máquina de estados y UI con fixtures.
Fase 2: seis agentes ADK in-process, Gemini, fuentes limitadas.
Fase 3: Cloud Run + Firestore/Cloud Storage, monitoreo y ensayo con periodistas.

## 14. Riesgos y decisiones abiertas
Disponibilidad de fuentes web y límites de grounding; sesgo y circularidad de fuentes; difamación; condiciones de uso; coste por reintentos; permisos de edición. La elección del proveedor de búsqueda se evalúa antes de codificar su adaptador.
