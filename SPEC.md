# AgentJournalist — SPEC v0.5 (FAST DEMO, PA-SDD)

Estado: diseño; no implica despliegue operativo.
Fuente de patrones: [DevPattern](https://github.com/JuliusCordova/DevPattern), especialmente `definition-packs/minimum-sufficient-specification.md`, `agentic-reference-architectures/gcp.md`, `patterns/implementation-profiles/gcp/pattern-pack-mapping.md` y `patterns/pattern-packs/governed-agent.md`.

## 0. Cambios de versión
| Versión | Cambio |
|---|---|
| v0.1 | Redacción agéntica de investigación: seis roles, cuatro compuertas, contratos, evals. |
| v0.2 | GraphRAG por caso (§15, ADR-002). |
| v0.3 | Verificación probabilística explicable (§16, ADR-003). |
| v0.4 | **Perfil TALLER** para el taller «IA aplicada al periodismo» de Kyndryl (alcance técnico de Henry Wong, Alternativas 01 y 02): verificación de un artículo contra fuentes oficiales, tablero con índice de respaldo documental, artículo corregido con estilo formal/informal, exportación Word, PDF con páginas, sesiones aisladas y concurrencia (§17). Skill de Claude equivalente (§18). Matriz de trazabilidad en [docs/TRAZABILIDAD-TALLER.md](docs/TRAZABILIDAD-TALLER.md). GraphRAG **se mantiene** en ambos perfiles. Se corrige la contradicción de v0.3 que listaba GraphRAG como fuera de alcance. |

| **v0.5** | **Hardening del perfil TALLER**: evidencia directa vs declaración oficial, alcance del IRD, G2 automático distinto de aprobación, diseño asíncrono/seguro, GraphRAG reproducible, evaluación y criterios de salida (§19). |

## 1. Intención del negocio
Demostrar a periodistas que una IA agéntica puede verificar o investigar información, descomponer afirmaciones, mostrar evidencia trazable y producir un texto revisable sin reemplazar el criterio humano.

**Indicador principal:** 100% de afirmaciones fácticas del texto generado con enlace a evidencia realmente recuperada; cero publicaciones automáticas.

## 2. Perfiles, persona y caso
La misma aplicación opera con dos perfiles que comparten contratos, reglas y controles:

| Perfil | Persona | Entrada | Salida |
|---|---|---|---|
| `TALLER` (§17) | Periodista sin conocimientos técnicos, en un hands-on de 35 min | Artículo de prueba + fuentes oficiales autorizadas (PDF/URL) | Tablero de verificación, evidencias, artículo corregido formal/informal, .docx |
| `REDACCION` (§3–§16) | Periodista/editor de investigación en una demo facilitada | Afirmación, URL o texto breve | Panel de investigación, expediente y borrador sujeto a firma |

Para ensayos desconectados se usan escenarios ficticios etiquetados como tales. El artículo del taller es ficticio y no debe difundirse como noticia real.

## 3. Alcance FAST DEMO
- Seis roles portados desde los skills originales: editor-jefe, investigacion, datos, verificacion, expediente y redaccion. En `TALLER` actúan investigacion (recuperación sobre corpus autorizado), verificacion, expediente y redaccion; datos queda `NO_APLICA`.
- Captura de texto/URL/PDF, plan o lista de afirmaciones, evidencias numeradas H/D/V, cronología y expediente.
- Compuertas humanas persistidas en backend (G1–G4); su forma de interacción depende del perfil (§17.4).
- Evidencia original con URL o documento, página cuando exista, fecha de consulta, fragmento, emisor y huella.
- Texto generado limitado a evidencias validadas, con [DESCARGO PENDIENTE] (REDACCION) o marca «Propuesta de IA sujeta a revisión editorial» (TALLER).
- GraphRAG por caso con grafo interactivo y comparación contra RAG convencional (§15).
- Métricas de verificación sin probabilidades inventadas (§16).
- UI con progreso, tarjetas de fuente, tablero y acciones de aprobación.
- Evals con casos positivos, insuficientes, contradicciones y prompt injection; matriz de resultados esperados del taller.

**Fuera de alcance:** publicación en redes o CMS, automatización de acusaciones, crawling masivo, scraping contra TOS, OCR de PDF escaneados, video, búsqueda de datos privados, páginas con inicio de sesión, Agent Engine administrado, GKE, aplicación móvil nativa, plena plataforma empresarial.

## 4. Historias
- US-001: como periodista quiero pegar una afirmación para obtener un plan editable.
- US-002: como editor quiero aprobar o rechazar el plan antes de investigar.
- US-003: como periodista quiero ver evidencia y anomalías con fuente y método reproducible.
- US-004: como editor quiero validar el expediente antes de generar el borrador.
- US-005: como editor quiero revisar y firmar manualmente un borrador atribuido.
- US-T01: como periodista del taller quiero cargar el artículo de ejemplo y elegir fuentes oficiales para verificarlo con un botón.
- US-T02: como periodista quiero ver un tablero por colores y abrir la evidencia de cada afirmación.
- US-T03: como periodista quiero generar el artículo corregido en estilo formal o informal y copiarlo o descargarlo en Word.
- US-T04: como periodista quiero ver qué relaciones (alianzas, productos, países) conectan las afirmaciones con las fuentes.

## 5. Requisitos funcionales (comunes)
- FR-001: la API registra un caso con ID, perfil, sesión, tipo de entrada y estado.
- FR-002: el editor-jefe propone preguntas acotadas y no continúa sin aprobación 1.
- FR-003: investigación y datos devuelven H/D con referencias y cálculos; si no hay datos tabulares, datos registra NO_APLICA.
- FR-004: la aprobación 2 establece explícitamente qué anomalías pasan a verificación (o NO_APLICA si no hay datos).
- FR-005: verificación descompone en afirmaciones atómicas y contrasta fuentes, sin equiparar dos republicaciones a dos fuentes.
- FR-006: el expediente mantiene confirmado/refutado/parcial/indicio/insuficiente/descartado; no inventa hechos.
- FR-007: la aprobación 3 bloquea o habilita redacción.
- FR-008: redacción recibe solo expediente validado; la aprobación 4 confirma revisión humana, pero no publica.
- FR-009: todas las transiciones y aprobaciones registran actor, fecha, versión, decisión y hash de artefacto.
- FR-010: errores o fuentes inaccesibles producen estado inconcluso con mensaje comprensible, nunca certeza falsa.
- FR-011: toda cita mostrada se valida en backend contra el fragmento recuperado; si no existe en el corpus, se descarta.

## 6. Requisitos no funcionales
- NFR-001: autorización de cambios de estado aplicada en backend, nunca por prompt.
- NFR-002: cuentas de servicio de mínimos privilegios; herramientas de solo lectura excepto almacenamiento del caso.
- NFR-003: logs sin secretos ni información personal innecesaria.
- NFR-004: sin declaración de veracidad si la evidencia es insuficiente.
- NFR-005: despliegue reproducible en Cloud Run con URL HTTPS, prueba de salud y límites de gasto configurables.
- NFR-006: tiempos medidos en evaluación; no prometer latencia fija de búsqueda externa.
- NFR-007: aislamiento por sesión: un participante no puede leer casos, archivos ni resultados de otro.
- NFR-008: concurrencia objetivo configurable (por defecto 30 sesiones simultáneas) validada con prueba de carga.
- NFR-009: retención limitada: datos del taller borrados al vencer el TTL (por defecto 24 h).
- NFR-010: interfaz responsive optimizada para portátil; mensajes en español claro.

## 7. Contratos de información
`Session {session_id, profile:TALLER|REDACCION, display_name?, created_at, expires_at}`
`Case {case_id, session_id, profile, input_text, input_url?, created_at, status, version}`
`SourceDocument {doc_id, session_id?, title, url?, kind:PDF|HTML|TEXT, authority:OFFICIAL|INDEPENDENT|UNKNOWN, preloaded:bool, pages?, content_hash, ingested_at}`
`Chunk {chunk_id, doc_id, page?, start, end, text}`
`Evidence {evidence_id, case_id, doc_id?, chunk_id?, source_url?, document_title, page?, publisher, published_at?, retrieved_at, excerpt, source_hash?, parent_source_id?, authority, injection_flags[]}`
`AtomicClaim {claim_id, case_id, text, span_start, span_end, claim_type:FECHA|CIFRA|PRODUCTO|CARACTERISTICA|PRESENCIA|ALIANZA|DECLARACION|OTRO, verifiable:bool, excluded_reason?}`
`Finding {finding_id, case_id, kind:H|D|V, claim, evidence_ids[], method?, verdict?, confidence_reason?}`
`Approval {approval_id, case_id, gate:1|2|3|4, artifact_version, artifact_hash, decision, reviewer_id, decided_at}`
`Dossier {case_id, version, confirmed_ids[], rejected_ids[], partial_ids[], inconclusive_ids[], timeline[], human_tasks[]}`
`Draft {case_id, dossier_version, cited_finding_ids[], text, approved_by?}`
`CorrectedArticle {case_id, dossier_version, style:FORMAL|INFORMAL, original_text, corrected_text, changes[], headlines[], disclaimer, fact_set_hash}`
`Change {claim_id, action:KEEP|CORRECT|QUALIFY|REMOVE, original_span, replacement, evidence_ids[], recommendation}`

## 8. Reglas del negocio
- BR-001: G1, G2, G3 y G4 son compuertas de aplicación, no instrucciones opcionales al LLM.
- BR-002: una anomalía no demuestra ilegalidad.
- BR-003: las fuentes independientes deben rastrearse a su origen documental.
- BR-004: redactar solo con afirmaciones respaldadas o corregidas con evidencia; las restantes se omiten o se matizan.
- BR-005: rechazos o cambios materiales invalidan aprobaciones posteriores.
- BR-006: nunca publicar ni enviar mensajes a terceros desde el agente.
- BR-007: la ausencia de información en una fuente no implica que una afirmación sea falsa.
- BR-008: el modelo no usa conocimiento general ni fuentes no autorizadas como sustituto de evidencia.
- BR-009: opiniones, expresiones subjetivas y recursos literarios no se clasifican como verdaderos o falsos.
- BR-010: umbral de evidencia por perfil: en `TALLER`, una fuente `OFFICIAL` autorizada puede servir para una conclusión limitada cuando presenta evidencia directa, pertinente al periodo y objeto de la afirmación; una declaración institucional solo demuestra lo que esa institución declaró, salvo corroboración adicional. En `REDACCION` se exigen dos fuentes con origen independiente (configurable). Ver §19.

## 9. Contrato de agente y herramientas
- `editor_jefe`: propone plan y delega; no ejecuta `approve`, `publish` o `sign`.
- `investigacion`: `search_sources` (corpus de la sesión + precargado), `get_source` (lectura, dominios autorizados).
- `datos`: `analyze_dataset` en ejecución aislada, límite de recursos, sin red.
- `verificacion`: `search_independent_sources`, `get_source` (lectura).
- `expediente`: `assemble_dossier` a partir de H/D/V, sin búsqueda.
- `redaccion`: `draft_from_approved_dossier`, `correct_article(style)`, sin búsqueda ni publicación.

Cada llamada lleva `case_id`, `session_id`, rol autorizado, límites, resultado y `trace_id`. Aprobaciones mediante endpoint autenticado fuera del modelo.

## 10. API
Comunes: `GET /health`; `POST /api/v1/sessions`; `POST /api/v1/cases`; `GET /api/v1/cases/{id}`; `POST /api/v1/cases/{id}/approvals/{gate}`; `GET /api/v1/cases/{id}/evidence`; `GET /api/v1/cases/{id}/graph`; `POST /api/v1/cases/{id}/graph/query`.
REDACCION: `POST /api/v1/cases/{id}/plan`; `GET /api/v1/cases/{id}/dossier`; `GET /api/v1/cases/{id}/draft`.
TALLER: `GET /api/v1/sources/preloaded`; `POST /api/v1/sessions/{sid}/sources/pdf`; `POST /api/v1/sessions/{sid}/sources/url`; `POST /api/v1/cases/{id}/verify`; `GET /api/v1/cases/{id}/dashboard`; `POST /api/v1/cases/{id}/corrected-article {style}`; `GET /api/v1/cases/{id}/corrected-article.docx`; `GET /api/v1/cases/{id}/report.docx` (opcional); `GET /api/v1/examples/article`.

## 11. Criterios de aceptación (REDACCION)
- AC-001: dado un caso nuevo, al generar plan queda WAITING_G1 y ninguna investigación se ejecuta antes de aprobar.
- AC-002: sin G2 aprobado, verificación no avanza; sin G3 aprobado, redacción no corre.
- AC-003: el borrador incluye evidencia válida por cada afirmación sustantiva; si no, la omite.
- AC-004: cualquier rechazo queda registrado y bloquea la continuación.
- AC-005: caso de evidencia contradictoria/insuficiente muestra abstención razonada.
- AC-006: inyección dentro de una fuente no puede ejecutar aprobación ni cambiar herramientas autorizadas.

Los criterios del perfil TALLER están en §17.10.

## 12. Evals FAST DEMO
- Fixture A: afirmación verificable con dos fuentes independientes.
- Fixture B: titular engañoso con dato parcialmente cierto.
- Fixture C: cita no rastreable o evidencia insuficiente.
- Fixture D: datos tabulares con anomalía explicable, sin inferencia acusatoria.
- Fixture E: fuente con instrucción maliciosa para saltar un gate.
- Fixture K-01 (taller): artículo ficticio sobre Kyndryl con afirmaciones respaldadas, contradichas, parcialmente respaldadas, no verificables y al menos una opinión; matriz de resultados esperados firmada por el equipo funcional.

Metas iniciales de ensayo (a validar con golden set): 100% bloqueos de gates; 100% citas fácticas trazables; cero publicación autónoma; clasificación de K-01 igual a la matriz esperada.

## 13. Secuencia de entrega
- Fase 0: conservar skills originales, artículo K-01, corpus oficial autorizado y matriz esperada.
- Fase 1: contratos JSON, máquina de estados, ingesta PDF/HTML, UI de taller con fixtures.
- Fase 2: agentes ADK in-process con Gemini, RAG híbrido + GraphRAG, artículo corregido y exportación Word; Skill de Claude (§18).
- Fase 3: Cloud Run + Firestore/Cloud Storage, sesiones con TTL, prueba de carga, ensayo general del taller.

## 14. Riesgos y decisiones abiertas
Disponibilidad de fuentes web y límites de grounding; sesgo y circularidad de fuentes; difamación; condiciones de uso; coste por reintentos; latencia con 30 sesiones simultáneas; límites de cuentas gratuitas de Claude para la Alternativa 02; vigencia de funciones de la plataforma Claude al día del evento.

| ID | Decisión | Responsable propuesto |
|---|---|---|
| D-T1 | Aprobar la fórmula del índice de respaldo documental (§17.6) | Equipo funcional del taller |
| D-T2 | Lista cerrada de documentos y dominios oficiales autorizados | Kyndryl |
| D-T3 | Número de asistentes y prueba de carga | Organización del taller |
| D-T4 | Región GCP, retención y borrado de datos | Kyndryl |
| D-T5 | Límite de palabras del artículo (por defecto 4.000) | Equipo técnico |
| D-T6 | Proveedor de búsqueda para el perfil REDACCION | Equipo técnico |

## 15. GraphRAG — alcance demostrativo obligatorio (v0.2)
**Objetivo:** demostrar investigación relacional entre entidades, afirmaciones, fuentes y evidencias, no sustituir la verificación por un grafo.
- FR-GR-01: extraer entidades (persona, organización, contrato, producto, país, fecha, afirmación, noticia, documento) y relaciones tipadas con `source_evidence_id`.
- FR-GR-02: presentar un grafo interactivo que permita seleccionar un nodo/arista y abrir el documento o fragmento probatorio.
- FR-GR-03: responder al menos tres consultas de múltiples saltos. REDACCION: ¿qué documentos conectan dos entidades?, ¿qué afirmaciones comparten fuente?, ¿qué entidades aparecen en contratos relacionados? TALLER: ¿qué documento oficial conecta a Kyndryl con el socio mencionado?, ¿qué afirmaciones del artículo comparten fuente?, ¿en qué países se documenta el servicio citado?
- FR-GR-04: permitir comparar respuesta con recuperación convencional y con GraphRAG sobre el mismo corpus y pregunta.
- FR-GR-05: etiquetar relaciones como `documented`, `inferred` o `unverified`; relaciones inferidas no pueden convertirse automáticamente en hechos confirmados ni respaldar una afirmación.
- FR-GR-06: detectar y mostrar entidades ambiguas y duplicadas sin resolver identidad automáticamente cuando haya incertidumbre.
- FR-GR-07: preservar procedencia y fecha de todas las aristas; no inferir culpabilidad, irregularidad ni vínculo personal por coocurrencia.
- FR-GR-08 (TALLER): el grafo aparece como pestaña «Relaciones» del tablero; aporta evidencia candidata a la verificación, pero el estado de cada afirmación lo deciden las reglas de §17.5.
- AC-GR-01: toda relación factual mostrada tiene evidencia original alcanzable; sin ella se etiqueta no verificada.
- AC-GR-02: consultas locales y relacionales del golden set se ejecutan tanto en baseline como en GraphRAG, documentando calidad/costo/latencia.
- AC-GR-03: grafo visual y consultas operan sin saltarse G1–G4.
- Modelo lógico: `GraphNode {node_id, type, label, aliases[], case_id}`; `GraphEdge {edge_id, from_id, to_id, relation_type, evidence_ids[], confidence_label, status, case_id}`.
- Límites: grafo por caso, corpus acotado y sin extracción masiva. NetworkX/JSON para la Fast PoC; persistencia en Cloud Storage y metadatos en Firestore. Sin base de grafos dedicada ni clustering de comunidades hasta que lo justifique la prueba. En TALLER, el subgrafo del corpus precargado se construye por adelantado.
- Exit gate: medir utilidad en preguntas de relación frente a baseline RAG, con exactitud por arista, cobertura de procedencia, latencia y costo. El criterio de promoción se fija antes del benchmark.

## 16. Métricas de verificación probabilística (v0.3)
**Intención:** permitir que el periodista vea evidencia, estado de verificación e incertidumbre cuantificada sin presentar la confianza verbal de Gemini como una probabilidad científicamente validada.

### Decisiones de producto
- Unidad primaria: **afirmación atómica** (`claim_id`), no noticia completa.
- Estado interno por afirmación: `SUPPORTED`, `REFUTED`, `MISLEADING`, `OUT_OF_CONTEXT`, `INSUFFICIENT_EVIDENCE`; se conserva revisión humana. En TALLER se presentan con las cuatro etiquetas de §17.5.
- Probabilidad `p_false` solo si hay clasificador calibrado y validado sobre dataset etiquetado fuera de muestra y dominio comparable. Si no, `p_false=null` y `calibration_status=NOT_CALIBRATED`.
- Presentar por separado **fortaleza de evidencia**, **cobertura**, **índice de respaldo documental** (descriptivo, §17.6) y **probabilidad** (solo calibrada).
- No sumar porcentajes de credibilidad inventados ni promediar probabilidades de afirmaciones correlacionadas.
- `INSUFFICIENT_EVIDENCE` / ambigüedad / fuentes dependientes implican abstención de veredicto automático.
- El resultado nunca habilita publicación ni acusaciones.

### Requisitos
- FR-PV-01: extraer y registrar afirmaciones atómicas, alcance, emisor, fecha de contexto y enlaces H/D/V.
- FR-PV-02: producir estados verificables y explicar por qué se eligió cada uno, citando evidencia y contradicciones.
- FR-PV-03: exponer conteos por estado, cobertura, índice de respaldo, cantidad de fuentes y procedencia de cada cita.
- FR-PV-04: mostrar `p_false` solo si el calibrador fue evaluado; incluir versión de modelo, dataset de calibración, fecha y dominio.
- FR-PV-05: evaluar comparativamente RAG vs GraphRAG y ejecución por agentes sobre golden set etiquetado.
- FR-PV-06: persistir decisión editorial, evidencia, predicción, versión y correcciones para auditoría.
- FR-PV-07: mostrar la cobertura incompleta y abstenerse cuando no alcance el umbral de evidencia.
- FR-PV-08: admitir degradación segura si faltan fuentes, referencias, calibrador o modelo.

### Esquema
`ClaimAssessment {claim_id, case_id, text, verdict, display_status, evidence_ids[], contradicting_evidence_ids[], evidence_strength, independent_source_count, p_false:float|null, calibration_status, model_version?, evaluated_at, abstain_reason?, explanation, recommendation?, reviewer_status}`
`EvaluationRun {run_id, dataset_version, model_version, method, split_policy, n_cases, metric_values, calibration_method?, created_at}`
`NewsSummary {case_id, claim_count, verifiable_count, excluded_count, supported_count, refuted_count, partial_count, insufficient_count, coverage_rate, support_index:float|null, support_index_formula_version, overall_status, overall_p_false:null}`

### Métricas de evaluación
- Matriz de confusión por clase; precisión, recall y F1 macro.
- Si existe calibrador: Brier score, log loss y ECE con curva de confiabilidad; intervalos por bootstrap si el tamaño lo permite.
- Recuperación: recall@k de evidencias, corrección de citas y exactitud de aristas GraphRAG frente a RAG simple.
- Seguridad: hallucinated-citation rate, false-accusation rate y gate-bypass rate (objetivo 0 en pruebas).
- Utilidad: tasa de abstención, cobertura, latencia y costo por artículo.

### Protocolo
1. Golden set con revisión humana doble; preferir casos públicos de fact-checking y registrar licencias.
2. Separar entrenamiento, calibración y prueba sin fuga por noticia, fuente o republicación.
3. Baseline sin calibración: estados y métricas descriptivas, sin probabilidades artificiales.
4. Solo con muestras suficientes: entrenar, calibrar en holdout y medir en test separado.
5. Reportar por temática, idioma y nivel de evidencia; documentar drift y límites.

### Acceptance
- AC-PV-01: sin calibrador validado, la UI no muestra ninguna cifra de falsedad.
- AC-PV-02: el reporte de pruebas incluye matriz de confusión, precision/recall/F1, cobertura y abstención.
- AC-PV-03: un artículo con afirmaciones de estados distintos conserva resultados individuales.
- AC-PV-04: toda clasificación incluye citas trazables o abstención explícita.
- AC-PV-05: el periodista puede inspeccionar por qué una afirmación quedó inconclusa.

## 17. Perfil TALLER — Agente de Verificación de Información (v0.4)
Responde al alcance técnico «Alternativa 01» del taller «IA aplicada al periodismo» (Kyndryl, 120 minutos, bloque de 35 minutos). La experiencia sigue el flujo: **Artículo → Fuentes oficiales → Verificación → Tablero → Corrección**.

### 17.1 Experiencia y pantallas
- **Pantalla 1 · Ingreso:** título «Agente de Verificación de Información», explicación breve, área de texto editable, botón «Cargar artículo de ejemplo», lista de documentos oficiales precargados con casillas, carga de PDF, campo de enlaces autorizados y botón «Verificar información».
- **Pantalla 2 · Tablero:** conteos por estado, índice de respaldo documental y cobertura, lista de afirmaciones con color, explicación, evidencia desplegable (documento, página/URL, fragmento) y botón «Generar artículo corregido». Pestaña «Relaciones» con el grafo (§15).
- **Pantalla 3 · Artículo corregido:** texto original y corregido lado a lado con cambios resaltados, selector Formal/Informal, botones Copiar y Descargar Word, «Nueva verificación».
- Sin configurar modelos, escribir código ni entrar a la consola de GCP. Progreso visible mientras se verifica.

### 17.2 Ingreso del artículo y fuentes
- FR-T01: pegar y editar el artículo; cargar el ejemplo K-01; validar que no esté vacío; rechazar con mensaje claro si supera el límite (por defecto 4.000 palabras, D-T5).
- FR-T02: fuentes precargadas oficiales de Kyndryl, procesadas e indexadas antes del evento (texto, fragmentos, embeddings y subgrafo).
- FR-T03: carga de PDF autorizados por sesión (texto digital; límite configurable de tamaño y páginas; sin OCR).
- FR-T04: ingreso de una o varias URL. El backend valida esquema HTTPS, dominio contra la lista autorizada (D-T2), destino final tras redirecciones y tamaño; si no cumple, rechaza e indica el motivo. Solo HTML público y PDF accesibles, sin páginas con inicio de sesión.
- FR-T05: extracción de texto de PDF y HTML, división en fragmentos con solapamiento y metadatos: título, URL o identificador, página, posición y huella.
- FR-T06: los archivos y fragmentos cargados en una sesión quedan en un espacio de nombres propio; no son visibles para otras sesiones (NFR-007).

### 17.3 Identificación de afirmaciones
- FR-T07: el agente identifica afirmaciones verificables (fechas y acontecimientos, cifras y porcentajes, productos y servicios, características, presencia geográfica, alianzas, declaraciones empresariales) y conserva el texto original y su ubicación (`span_start`, `span_end`).
- FR-T08: opiniones, valoraciones y recursos literarios se listan como «no evaluadas por ser opinión» y no cuentan en el índice (BR-009).

### 17.4 Compuertas en el perfil TALLER
Las transiciones están en backend. En TALLER hay **3 decisiones humanas (G1, G3, G4) y 1 transición automática NO_APLICA (G2)**, para caber en 20 minutos de práctica; no se presenta G2 como aprobación humana:

| Gate | REDACCION | TALLER | Artefacto con hash |
|---|---|---|---|
| G1 | Aprobar plan | Pulsar «Verificar información» con artículo y fuentes seleccionadas | Artículo + lista de fuentes |
| G2 | Elegir anomalías | `AUTO_NOT_APPLICABLE` (sin datos tabulares); transición técnica registrada por backend, no aprobación humana | — |
| G3 | Aprobar expediente | Pulsar «Generar artículo corregido» tras ver el tablero | Resultados de verificación |
| G4 | Firmar borrador | Confirmar «Revisé la propuesta» antes de copiar o descargar | Artículo corregido |

La identidad del revisor en TALLER es la sesión del participante. Ningún agente puede ejecutar estas acciones. En el bloque de comparación puede mostrarse el flujo REDACCION completo como ejemplo de control empresarial.

### 17.5 Estados y reglas de verificación
| Estado visible | Color | Estado interno | Regla |
|---|---|---|---|
| Respaldada | Verde | SUPPORTED | Evidencia oficial directa, pertinente y contextualizada respalda la afirmación completa; una nota institucional no prueba por sí sola hechos externos controvertidos. |
| Contradicha | Rojo | REFUTED | Evidencia documental directa y específica contradice la afirmación en su mismo contexto temporal; ausencia de mención y comunicados no concluyentes no cuentan como refutación. |
| Parcialmente respaldada | Naranja | MISLEADING / OUT_OF_CONTEXT | Una parte está respaldada y otra es incorrecta, incompleta o fuera de contexto. |
| No verificable | Gris | INSUFFICIENT_EVIDENCE | Las fuentes disponibles no aportan evidencia suficiente, o hay fuentes oficiales en conflicto sin resolver. |

- FR-T09: el modelo propone la postura de cada fragmento (respalda / contradice / parcial / sin relación); el estado lo calculan reglas deterministas con esa postura y la autoridad de la fuente (BR-010).
- FR-T10: la ausencia de información nunca produce «Contradicha» (BR-007); el modelo no usa conocimiento general ni fuentes externas (BR-008).
- FR-T11: cada resultado muestra afirmación original, estado, explicación breve, documento, fragmento, página o URL, y recomendación de corrección cuando corresponda. El periodista puede seleccionar la afirmación y abrir la evidencia.
- FR-T12: cada fragmento mostrado debe existir en los fragmentos recuperados del corpus (FR-011); páginas, citas o enlaces que no existan se descartan y la afirmación pasa a «No verificable».

### 17.6 Tablero e índice de respaldo documental
- FR-T13: indicadores mínimos: total de afirmaciones, respaldadas, contradichas, parcialmente respaldadas, no verificables, opiniones excluidas, cobertura e índice.
- **Fórmula propuesta (D-T1, versión `IRD-1`):**
  - Cobertura = (R + C + P) / (R + C + P + NV)
  - Índice de respaldo **entre afirmaciones evaluadas** = (R + 0,5 × P) / (R + C + P)
  - R = respaldadas, C = contradichas, P = parcialmente respaldadas, NV = no verificables; las opiniones excluidas no cuentan.
  - Si R + C + P = 0, el índice no se muestra y aparece «Sin evidencia suficiente para calcular el índice».
- FR-T14: el índice se rotula como «IRD-1 · Respaldo entre afirmaciones evaluadas», medida descriptiva del respaldo entre las afirmaciones evaluadas; nunca como probabilidad de verdad ni exactitud del modelo. Debe mostrarse con cobertura y conteos absolutos. Se muestra siempre junto a la cobertura y con la versión de fórmula.

### 17.7 Artículo corregido y estilo
- FR-T15: «Generar artículo corregido» produce una versión que mantiene la intención informativa, conserva lo respaldado, corrige lo contradicho solo con la evidencia disponible, ajusta lo parcialmente respaldado, retira o matiza lo no verificable («no pudo confirmarse con las fuentes consultadas») y no agrega información sin sustento.
- FR-T16: cada cambio se registra como `Change` con acción, evidencia y recomendación, y se resalta en la vista lado a lado.
- FR-T17: selector Formal / Informal. El cambio de estilo no modifica los hechos: ambas versiones comparten el mismo `fact_set_hash` normalizado (afirmaciones, cifras, fechas, entidades y negaciones); además se valida conservación semántica por afirmación. El hash es una comprobación necesaria, no prueba suficiente de equivalencia semántica; si hay discrepancia, se bloquea para revisión o regeneración.
- FR-T18: el texto muestra la marca «Propuesta generada por IA, sujeta a revisión editorial antes de su publicación» y, de forma opcional, tres titulares alternativos.

### 17.8 Exportación
- FR-T19: copiar el texto corregido al portapapeles y descargar un .docx válido con título, texto corregido y marca de propuesta de IA.
- FR-T20: mantener el resumen de verificación disponible durante la sesión.
- FR-T21 (opcional): reporte de verificación .docx con afirmaciones, estados, fuentes y fragmentos.

### 17.9 Operación del taller
- FR-T22: sesiones anónimas con identificador por participante; TTL y borrado automático (NFR-009).
- FR-T23: verificación asíncrona con barra de progreso y reintento; evitar ejecuciones duplicadas por doble clic.
- FR-T24: límites por sesión (artículos, PDF, llamadas) y tope de gasto global configurable.
- FR-T25: errores con mensajes comprensibles y opción de reintentar; fuentes caídas no detienen el análisis del resto.
- Plataforma: el alcance admite Azure o GCP; se elige **GCP** (Cloud Run, Vertex AI/Gemini, Firestore, Cloud Storage) por coherencia con la arquitectura de referencia.

### 17.10 Criterios de aceptación TALLER
| ID | Prueba | Resultado esperado |
|---|---|---|
| CA01 | Acceder mediante URL HTTPS | La aplicación abre desde un navegador externo autorizado. |
| CA02 | Cargar artículo de ejemplo | El contenido aparece en el editor. |
| CA03 | Seleccionar fuentes oficiales | El sistema usa solo las fuentes autorizadas seleccionadas. |
| CA04 | Cargar PDF válido | El documento se procesa, con páginas, y entra al análisis. |
| CA05 | Ingresar URL no autorizada | El sistema la rechaza e informa el motivo. |
| CA06 | Verificar artículo | Se identifican y clasifican las afirmaciones principales. |
| CA07 | Consultar una evidencia | Se muestra un fragmento real con su documento y página o URL. |
| CA08 | Información sin evidencia | El agente devuelve «No verificable», sin inventar. |
| CA09 | Generar artículo corregido | Las correcciones se basan en evidencias y cada cambio está trazado. |
| CA10 | Cambiar estilo | Cambia el tono sin alterar los hechos (`fact_set_hash` igual). |
| CA11 | Descargar artículo | Se genera un .docx válido. |
| CA12 | Varias sesiones | Los participantes trabajan sin ver datos de otros. |
| CA13 | Carga concurrente | Comportamiento aceptable con la cantidad de asistentes prevista (D-T3). |
| CA14 | Gestionar errores | Mensajes comprensibles y reintento posible. |
| CA15 | Fuente o artículo con instrucción maliciosa | No cambia estados, herramientas ni resultados; la fuente aparece marcada. |
| CA16 | Cita inexistente propuesta por el modelo | Se descarta en backend; no se muestra. |
| CA17 | Artículo K-01 | La clasificación coincide con la matriz de resultados esperados. |
| CA18 | Pestaña Relaciones | Las aristas mostradas tienen evidencia; la comparación RAG vs GraphRAG se ejecuta sobre el mismo corpus. |

## 18. Alternativa 02 — Skill de Claude «Verificador de Información Periodística» (v0.4)
Contraparte del perfil TALLER para el bloque de 40 minutos. Usa el mismo artículo K-01, las mismas fuentes y los mismos estados, para que los periodistas comparen flexibilidad, control, trazabilidad y gobernanza.

- FR-S01: Skill en formato compatible con Claude, empaquetado como `skill-verificador-informacion.zip`, con `SKILL.md` en Markdown y sin scripts ni dependencias externas en la primera versión. Se deriva del skill original `newsroom-verificacion`.
- FR-S02: `SKILL.md` define nombre, descripción, cuándo usarlo, procedimiento, identificación de afirmaciones, reglas de fuentes, criterios de los cuatro estados (§17.5), presentación de evidencias, cálculo del índice con la misma fórmula `IRD-1` (§17.6), reglas del artículo corregido, estilos formal/informal, restricciones contra información inventada y formato de salida en tablas.
- FR-S03: reglas explícitas: no considerar verdadera una afirmación por parecer razonable; no considerarla falsa por no aparecer; no inventar fuentes, páginas ni citas; no usar conocimiento general como evidencia; informar cuando un enlace no pueda consultarse; evitar porcentajes si no hay afirmaciones verificables.
- FR-S04: instrucción base para el ejercicio de personalización: «Después de corregir el artículo, genera tres titulares alternativos» y «Presenta las afirmaciones en el orden en que aparecen en el artículo». El ejercicio modifica una regla de formato, nunca el criterio de verificación.
- FR-S05: exportación por copia o mediante la creación de Word disponible en Claude; si no está disponible, copiar el texto.
- FR-S06: materiales: ZIP, `SKILL.md` editable, artículo K-01, PDF oficiales, lista de enlaces, guías de instalación, ejecución y modificación, matriz de resultados esperados y material de comparación.
- Límite declarado: las reglas del Skill son instrucciones, no bloqueos técnicos; no aísla fuentes ni registra auditoría. Esa diferencia con §17 es parte del mensaje del taller.
- Aceptación: CA01–CA20 del documento de alcance de la Alternativa 02, verificados con cuentas gratuitas antes del evento y revalidados cerca de la fecha por posibles cambios de la plataforma.

## 19. Fortalecimiento técnico y criterios de salida (v0.5)

### 19.1 Evidencia, fuente oficial y decisiones
- FR-H01: cada afirmación se evalúa como una proposición verificable con sujeto, predicado, alcance, fecha y contexto. El sistema distingue (a) fuente primaria documental sobre el hecho, (b) testimonio/declaración oficial de parte interesada, (c) fuente independiente y (d) ausencia de evidencia.
- FR-H02: una fuente oficial que declara «X ocurrió» demuestra que la entidad dijo «X», pero no convierte por sí sola «X ocurrió» en hecho confirmado si requiere comprobación externa.
- FR-H03: la regla de respaldo/refutación exige coincidencia de entidad, predicado, periodo y condiciones; fuentes temporalmente desactualizadas o contradictorias conducen a `INSUFFICIENT_EVIDENCE` y revisión.
- FR-H04: el juicio de postura lo sugiere el modelo; un motor de reglas comprueba existencia de citas, correspondencia contextual y condición de la fuente. No presumir que reglas deterministas certifican la veracidad de una postura emitida por LLM.
- FR-H05: cualquier revisión humana de un veredicto guarda motivo, evidencia y versión anterior; la corrección no reescribe originales.

### 19.2 IRD y scoring
- Nombre visible: `IRD-1: respaldo entre afirmaciones evaluadas`. Definición descriptiva `(R+0.5P)/(R+C+P)`, solo cuando denominador es positivo. Mostrar al lado cobertura `(R+C+P)/(R+C+P+NV)`, numeradores y denominadores.
- Nota metodológica permanente: este índice depende del corpus seleccionado y **no representa probabilidad de verdad, precisión del clasificador ni porcentaje de noticias falsas**. Debe advertir si la cobertura es baja.
- `p_false` continúa nulo hasta calibración por conjunto independiente representativo; guardar binning, Brier, ECE, tamaño y corte temporal de prueba.
- No convertir `1-IRD` en `p_false`; las afirmaciones no verificables se mantienen diferenciadas de las refutadas.

### 19.3 Contratos HITL por perfil
- REDACCION: G1/G2/G3/G4 son cuatro controles humanos.
- TALLER: G1/G3/G4 son interacciones humanas registradas; G2 es `AUTO_NOT_APPLICABLE` y debe incluir `reason=NO_TABULAR_DATA`, `actor_type=SYSTEM`, huella de artefacto y marca de tiempo; **no** tiene `reviewer_id` ficticio.
- Los endpoints de aprobación exigen sesión autenticada/autorizada por token opaco, control de propietario del caso y versión optimista; sin autorización se devuelve 403 y sin versión correcta 409.
- `G4` evita exportar hasta que se confirme «Revisé la propuesta», sin fingir identidad jurídica del periodista cuando el taller usa sesión anónima.
- Un cambio en fuente, artículo o veredicto invalida el hash de aprobaciones posteriores.

### 19.4 Orquestación, seguridad y concurrencia
- Procesar verificación mediante job lógico `PENDING→RUNNING→COMPLETED|PARTIAL|FAILED|CANCELLED` con `job_id`, idempotency key, checkpoints e historial; API devuelve `202 Accepted` y endpoint para consultar progreso. Cloud Run no asume estado persistente en memoria.
- Si el trabajo supera la ejecución interactiva segura, emplear cola/ejecutor administrado cuando las pruebas justifiquen la complejidad; no afirmar que 30 sesiones equivalen a 30 llamadas paralelas al modelo sin cuota.
- Ingesta de URL: HTTPS, lista de hosts permitidos, resolución DNS y bloqueo de IP privadas/link-local/metadata, revalidación de redirecciones y DNS, límites de tamaño, tiempo y tipo MIME; descarga sin credenciales, defensa SSRF y contenido no confiable.
- Ingesta de PDF: límites de páginas y bytes, sanitización, hash y extracción sin ejecución de instrucciones embebidas. TTL con política efectiva de borrado; verificar su comportamiento en servicios usados.
- Evitar que prompts extraídos de fuentes modifiquen instrucciones, herramientas, orquestación o aprobaciones. Registrar intentos sin almacenar secretos.

### 19.5 GraphRAG verificable
- Esquema tipado de nodo y arista con dirección, evidencia exacta, vigencia temporal, estado `DOCUMENTED|INFERRED|UNVERIFIED` y procedencia `doc_id/page/chunk_id`.
- Separar ruta relacional sugerida por el grafo de afirmación comprobada. La proximidad/coocurrencia no implica causalidad, contrato comercial ni irregularidad.
- Comparar al menos tres consultas predefinidas multi-hop entre RAG clásico y GraphRAG, con mismo corpus, corte temporal, respuesta esperada y evaluación ciega. Medir exactitud de relaciones, recall de evidencia, costo y latencia.
- En TALLER precalcular el subgrafo de las fuentes oficiales y reservar la extracción dinámica a archivos del participante. Si no mejora calidad en el golden set, la pestaña Relaciones se mantiene como exploración y no se declara superioridad.

### 19.6 SDD Definition of Ready / Done
**Ready para implementación:** corpus autorizado y licencias aprobadas; K-01 y matriz esperada versionados; servicio/región y quotas configurables; contratos de sesión y evidencia definidos; responsables de revisión editorial nombrados.

**Done para demo:** CA01–CA18 más HA01–HA08 (abajo) automatizados o ejecutados con evidencia; prueba de carga de 30 sesiones según patrón representativo; ninguna cita fabricada en golden set, ningún bypass de autorización/gates, cero publicaciones automáticas, registro de fallos/latencia/costo, y plan de contingencia con corpus precargado.

### 19.7 Criterios adicionales
- HA01: declaración de empresa interesada no verifica automáticamente hecho externo; se marca como declaración o requiere corroboración.
- HA02: ausencia de una afirmación en un PDF no es refutación.
- HA03: IRD=100% con cobertura 20% se muestra con aviso de baja cobertura y sin probabilidad.
- HA04: G2 TALLER registra actor SYSTEM y no una firma humana falsa.
- HA05: cambiar fuentes tras aprobación invalida artefactos posteriores; solicitudes simultáneas con versión antigua reciben 409.
- HA06: URL que redirige a IP privada, localhost o metadata se rechaza antes de obtener contenido.
- HA07: una relación GraphRAG sin fragmento sustentatorio se muestra como no verificada y nunca fundamenta un veredicto.
- HA08: un fallo de proveedor deja job PARTIAL/FAILED recuperable; no se pierde aislamiento ni se generan aprobaciones implícitas.
