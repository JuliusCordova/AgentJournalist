# Matriz de trazabilidad — perfil TALLER (v0.5)

> Esta matriz se deriva de SPEC.md v0.5 y los criterios CA01–CA18 del artículo K-01. No sustituye la matriz funcional aprobada por los organizadores.

| Jornada | Requisitos | Criterios | Evidencia esperada |
|---|---|---|---|
| Entrar a aplicación | NFR-005, NFR-010 | CA01 | URL HTTPS y smoke test |
| Cargar artículo ficticio K-01 | FR-T01, FR-T07, FR-T08 | CA02, CA06, CA17 | texto y golden set versionado |
| Seleccionar documentos oficiales | FR-T02, FR-T06 | CA03 | lista de documentos efectivos por sesión |
| Cargar PDF digital | FR-T03, FR-T05 | CA04 | fragmento y número de página correctos |
| Fuente URL segura | FR-T04, FR-011, §19.4 | CA05, HA06 | rechazo de URL no autorizada/SSRF |
| Recuperar evidencia | FR-T09 a FR-T12 | CA07, CA08, CA16, HA01, HA02 | citas verificadas y abstenciones |
| Tablero | FR-T13, FR-T14 | CA06, HA03 | estados, conteos, IRD-1 y cobertura |
| Corrección formal/informal | FR-T15 a FR-T18 | CA09, CA10 | diff, evidencia y conservación semántica |
| Exportación Word | FR-T19, FR-T20 | CA11 | DOCX válido y descargo |
| Aislamiento por sesión | FR-T06, FR-T22 | CA12 | prueba cruzada entre dos sesiones |
| Concurrencia | FR-T23, FR-T24, NFR-008 | CA13, HA08 | reporte de carga, cuotas y latencia |
| Recuperación de fallos | FR-T25 | CA14, HA08 | job PARTIAL/FAILED y reintentos |
| Defensa de prompt injection | BR-008, NFR-001 | CA15 | prueba adversarial sin acciones indebidas |
| GraphRAG | FR-GR-01 a FR-GR-08 | CA18, HA07 | grafo, aristas citadas y benchmark |
| HITL | BR-001, §17.4, §19.3 | HA04, HA05 | tres aprobaciones humanas y G2 automático |
| Claude (alternativa 02) | FR-S01 a FR-S06 | Aceptación en §18 | ZIP, guía, mismas fuentes/K-01 y comparación |

## Matriz dorada por afirmación (plantilla)
`claim_id | original_span | estado_esperado | evidencia_ids | doc_id | page | justificacion | revisor_1 | revisor_2`

El ejercicio K-01 requiere completar y aprobar esta matriz antes de usar métricas de exactitud. Un estado de noticia derivado de un solo titular no sustituye la etiqueta de cada afirmación.

## Demos verificables
1. Noticia con dato correcto y enlace al PDF oficial.
2. Noticia con dato contradicho en período y contexto idénticos.
3. Noticia con afirmación no mencionada por el corpus: abstención.
4. Artículo mezcla hecho, opinión y parcialidad: índice y cobertura separados.
5. Pregunta relacional: evidencia directa por cada arista y comparación RAG/GraphRAG.

## Pendientes de validación funcional
D-T1 fórmula IRD-1; D-T2 corpus/dominios oficiales; D-T3 aforo exacto; D-T4 región/retención; D-T5 límite de palabras; D-T6 proveedor de búsqueda REDACCION.
