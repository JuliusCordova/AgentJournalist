# Contratos de agentes — AgentJournalist

Arquitectura: **seis agentes ADK en proceso** + **dos motores especializados (GraphRAG y evaluación de evidencias)**. Los motores son herramientas/servicios de dominio, **no agentes autónomos adicionales**. El backend controla permisos, estados y cuatro gates lógicos.

| Componente | Entrada | Salida | Permitido | Prohibido |
|---|---|---|---|---|
| `newsroom-editor-jefe` | Case + session + profile | Plan + delegaciones + referencias | planificar, orquestar agentes | aprobar, firmar, publicar |
| `newsroom-investigacion` | preguntas + fuentes permitidas | H1…Hn + EvidenceIDs | buscar/leer fuentes autorizadas | declarar culpabilidad o certeza sin evidencia |
| `newsroom-datos` | artefactos tabulares autorizados | D1…Dn + método | cálculo reproducible aislado | acceso a red sin autorización, denuncias |
| `newsroom-verificacion` | afirmaciones + evidencias | V1…Vn, posturas y abstención | recuperar/contrastar y usar scoring | emitir `p_false` sin calibrador |
| `newsroom-expediente` | H/D/V y grafo citado | Dossier versionado | ordenar, deduplicar y documentar | crear hechos o buscar fuentes nuevas |
| `newsroom-redaccion` | Dossier aprobado | Draft / CorrectedArticle | corregir con evidencia y estilo | búsqueda, publicación o envío a terceros |

## Herramientas compartidas
- `graph_extract`, `graph_query`, `graph_evidence_path`: extracción/retrieval GraphRAG, con procedencia por arista, namespaces por sesión/caso.
- `assess_claim`, `compute_support_index`, `evaluation_report`: reglas y métricas; las probabilidades calibradas dependen de un modelo estadístico validado y versionado.
- `get_source`, `search_sources`: lectura con aislamiento/autorización.
- `transition_case`, `record_approval`: **solo backend** después de verificar identidad, gate y versión del artefacto; jamás se exponen como herramientas invocables por LLM.

## Contrato de ejecución mínimo
```json
{
  "case_id": "uuid",
  "session_id": "opaque-id",
  "profile": "TALLER",
  "actor_role": "newsroom-verificacion",
  "artifact_version": 3,
  "trace_id": "opaque-trace",
  "input_refs": ["evidence:H1"],
  "output_schema_version": "1.0",
  "timeout_s": 90
}
```

## Comp puertas por perfil
- `REDACCION`: G1 plan, G2 anomalías, G3 expediente, G4 borrador: **4 personas/decisiones humanas**.
- `TALLER`: G1 verificar, G2 sistema `AUTO_NOT_APPLICABLE`, G3 corregir, G4 revisión antes de exportación: **3 acciones humanas, una transición técnica**.
- Cualquier rechazo, invalidación de hash o cambio de fuente obliga a recomputar artefactos dependientes.

## Requisitos por tool
Validar JSON/Pydantic, número máximo de llamadas, presupuesto, identidad, pertenencia de fuente al corpus, documentos no confiables, límites de tokens y redacción de logs. No considerar strings de prompts como autorización.
