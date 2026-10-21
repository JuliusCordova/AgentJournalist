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


## 15. GraphRAG — alcance demostrativo obligatorio (v0.2)
**Objetivo:** demostrar investigación relacional entre entidades, afirmaciones, fuentes y evidencias, no sustituir la verificación por un grafo.
- FR-GR-01: extraer entidades (persona, organización, contrato, fecha, afirmación, noticia, documento) y relaciones tipadas con `source_evidence_id`.
- FR-GR-02: presentar un grafo interactivo que permita seleccionar un nodo/arista y abrir el documento o fragmento probatorio.
- FR-GR-03: responder al menos tres consultas de múltiples saltos (¿qué documentos conectan dos entidades?, ¿qué afirmaciones comparten fuente?, ¿qué entidades aparecen en contratos relacionados?).
- FR-GR-04: permitir comparar respuesta con recuperación convencional y con GraphRAG sobre el mismo corpus y pregunta.
- FR-GR-05: etiquetar relaciones como `documented`, `inferred` o `unverified`; relaciones inferidas no pueden convertirse automáticamente en hechos confirmados.
- FR-GR-06: detectar y mostrar entidades ambiguas y duplicadas sin resolver identidad automáticamente cuando haya incertidumbre.
- FR-GR-07: preservar procedencia y fecha de todas las aristas; no inferir culpabilidad, irregularidad ni vínculo personal por coocurrencia.
- AC-GR-01: toda relación factual mostrada tiene evidencia original alcanzable; sin ella se etiqueta no verificada.
- AC-GR-02: consultas locales y relacionales del golden set se ejecutan tanto en baseline como en GraphRAG, documentando calidad/costo/latencia.
- AC-GR-03: grafo visual y consultas operan sin saltarse G1–G4.
- Modelo lógico ampliado: `GraphNode {node_id, type, label, aliases[], case_id}`; `GraphEdge {edge_id, from_id, to_id, relation_type, evidence_ids[], confidence_label, status, case_id}`.
- Límites: grafo por caso, corpus acotado y sin extracción masiva. Una representación NetworkX/JSON puede usarse para la Fast PoC; persistencia en Cloud Storage y metadatos en Firestore. No exigir base de grafos dedicada ni clustering de comunidades hasta que lo justifique la prueba.
- Exit gate: medir utilidad en preguntas de relación frente a baseline RAG, con métricas de exactitud por arista, cobertura de procedencia, latencia y costo. El criterio de promoción se fija antes del benchmark.


## 16. Métricas de verificación probabilística (v0.3)
**Intención:** permitir que el periodista vea evidencia, estado de verificación e incertidumbre cuantificada sin presentar la confianza verbal de Gemini como una probabilidad científicamente validada.

### Decisiones de producto
- Unidad primaria: **afirmación atómica** (`claim_id`), no noticia completa. Una noticia puede combinar afirmaciones verdaderas, falsas y no verificables.
- Estado editorial por afirmación: `SUPPORTED`, `REFUTED`, `MISLEADING`, `OUT_OF_CONTEXT`, `INSUFFICIENT_EVIDENCE`; se conserva revisión humana.
- Probabilidad `p_false` solo si hay clasificador calibrado y validado sobre dataset etiquetado fuera de muestra y dominio comparable. Si no, `p_false=null` y `calibration_status=NOT_CALIBRATED`.
- Presentar por separado **fortaleza/calidad de evidencia** (rúbrica ordinal con fuentes y procedencia), **cobertura** (qué parte de la noticia se pudo verificar) y **probabilidad** (estimación estadística con versión de modelo).
- No sumar porcentajes de 'credibilidad de fuentes' inventados ni promediar probabilidades de afirmaciones correlacionadas como si fuesen independientes.
- `INSUFFICIENT_EVIDENCE` / ambigüedad / fuentes dependientes implican abstención de veredicto automático, aun con una señal de riesgo alta.
- El resultado nunca habilita publicación ni acusaciones; se mantienen cuatro HITL gates.

### Requisitos
- FR-PV-01: extraer y registrar afirmaciones atómicas, alcance, emisor, fecha de contexto y enlaces H/D/V.
- FR-PV-02: producir estados verificables y explicar por qué se eligió cada uno, citando evidencia independiente y contradicciones.
- FR-PV-03: exponer conteos por estado, porcentaje de afirmaciones verificadas, cantidad de fuentes independientes y procedencia de cada cita.
- FR-PV-04: mostrar `p_false` como valor [0,1] solo si el calibrador fue evaluado; incluir versión de modelo, dataset de calibración, fecha y dominio. De lo contrario no mostrar porcentajes de falsedad.
- FR-PV-05: evaluar comparativamente pipeline RAG vs GraphRAG y ejecución por agentes sobre golden set etiquetado, estratificado por tema, fuente y tiempo.
- FR-PV-06: persistir decisión editorial, evidencia, predicción, versión y correcciones para auditoría.
- FR-PV-07: mostrar la cobertura incompleta y abstenerse cuando no alcance el umbral de evidencia definido en evaluación.
- FR-PV-08: admitir degradación segura si faltan fuentes, referencias, calibrador o modelo.

### Esquema
`ClaimAssessment {claim_id, case_id, text, verdict, evidence_ids[], contradicting_evidence_ids[], evidence_strength, independent_source_count, p_false:float|null, calibration_status, model_version?, evaluated_at, abstain_reason?, reviewer_status}`
`EvaluationRun {run_id, dataset_version, model_version, method, split_policy, n_cases, metric_values, calibration_method?, created_at}`
`NewsSummary {case_id, claim_count, assessed_count, supported_count, refuted_count, misleading_count, out_of_context_count, insufficient_count, coverage_rate, overall_status, overall_p_false:null}`

El resumen de noticia NO tiene `overall_p_false` numérico por defecto: cualquier agregación estadística futura requiere modelo propio entrenado para nivel de noticia, datos de dependencia y validación.

### Métricas de evaluación de clasificación
- Matriz de confusión por clase; precisión, recall y F1 macro (evitar que clase mayoritaria oculte errores).
- Para etiqueta binaria false/no-false únicamente en subset adjudicado: precision/recall/F1 de clase falsa, PR-AUC y ROC-AUC cuando proceda.
- Probabilísticas (si existe calibrador): Brier score, log loss y Expected Calibration Error (ECE), con curva de confiabilidad y tamaños por bin. Reportar intervalos de confianza por bootstrap si el tamaño lo permite.
- Calidad de recuperación: recall@k de evidencias, corrección de citas y exactitud de relaciones/aristas GraphRAG, con contraste contra RAG simple.
- Seguridad: hallucinated-citation rate, false-accusation rate y gate-bypass rate (objetivo 0 en pruebas).
- Utilidad: tasa de abstención, riesgo selectivo al abstenerse, cobertura, latencia y costo por noticia.

### Protocolo de evaluación
1. Crear golden set de afirmaciones y evidencias con revisión humana doble y protocolo para desacuerdos; preferir casos públicos de fact-checking y registrar licencias/contexto.
2. Separar entrenamiento, calibración y prueba sin fuga por noticia, fuente o republicación; corte temporal cuando aplique.
3. Implementar baseline sin calibración: estados y métricas descriptivas, **sin probabilidades artificiales**.
4. Si hay muestras suficientes: entrenar un clasificador de falsedad, calibrar en holdout (Platt/sigmoid o isotónica según volumen) y medir en test separado.
5. Reportar desempeño por temática, idioma y nivel de evidencia. Documentar drift y límite de aplicabilidad.

### Acceptance
- AC-PV-01: sin calibrador validado, UI muestra «Probabilidad no calibrada/no disponible» y ninguna cifra de falsedad.
- AC-PV-02: reporte de pruebas incluye matriz de confusión, precision/recall/F1, cobertura y abstención; probabilidades solo si hay evaluación Brier/ECE.
- AC-PV-03: noticia con dos afirmaciones de veredictos distintos conserva resultados individuales y no inventa porcentaje único.
- AC-PV-04: toda clasificación incluye citas trazables o abstención explícita.
- AC-PV-05: el periodista puede inspeccionar por qué un caso quedó inconcluso y puede corregirlo sin borrar historial.
