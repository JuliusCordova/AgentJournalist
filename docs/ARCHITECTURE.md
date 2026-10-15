# Arquitectura GCP — FAST DEMO

## Selección de patrones DevPattern
- `definition-packs/minimum-sufficient-specification.md`: SPEC.md antes de codificar.
- `agentic-reference-architectures/gcp.md`: Cloud Run + ADK + Gemini, evitar infraestructura innecesaria.
- `patterns/implementation-profiles/gcp/pattern-pack-mapping.md`: multi-agent manager in-process, HITL con estado persistente.
- `patterns/pattern-packs/governed-agent.md`: capacidades y autorizaciones deterministas.
- `examples/legal-contract-agent-fast-demo/GCP-FAST-DEMO.md`: React + FastAPI + ADK + Cloud Run, ajustado para el dominio periodístico.

## Por qué sí multiagente en FAST DEMO
Ya existen seis skills independientes validados de forma exploratoria en Claude. Reutilizar el reparto de trabajo genera una demostración diferenciada; **no** implica seis servicios ni seis runtimes separados.

## Arquitectura
```mermaid
flowchart TB
    P[Periodista / editor] --> UI[React]
    UI --> API[FastAPI en Cloud Run]
    API --> FSM[State machine / gates 1-4]
    FSM --> ADK[ADK root: editor jefe]
    ADK --> INV[Investigación]
    ADK --> DAT[Datos]
    INV --> VER[Verificación]
    DAT --> VER
    VER --> EXP[Expediente]
    EXP --> RED[Redacción]
    INV --> SEARCH[Herramienta de búsqueda]
    DAT --> ANALYSIS[Python aislado]
    ADK --> GEM[Gemini / Vertex AI]
    API --> FS[(Firestore: estados y aprobaciones)]
    API --> GCS[(Cloud Storage: evidencias y expedientes)]
    API --> LOG[Cloud Logging]
```

**Importante:** las flechas entre agentes representan precedencia lógica, pero las transiciones están gobernadas por FSM/API. ADK no debe completar todo el proceso en una invocación que ignore las aprobaciones.

## Etapas
`NEW -> PLAN_READY -> WAITING_G1 -> RESEARCH -> WAITING_G2 -> VERIFICATION -> DOSSIER_READY -> WAITING_G3 -> DRAFT_READY -> WAITING_G4 -> EDITOR_APPROVED`.

`REJECTED`, `INCONCLUSIVE` y `ERROR` son estados explícitos. Solo el backend con identidad autorizada cambia de gate.

## Herramientas y límites
- Consulta de fuentes: dominios autorizados, URLs y fecha de recuperación; preservar cadena de fuente original.
- Análisis tabular: Python aislado, solo archivos del caso y límites de CPU/memoria/tiempo.
- Agente de redacción: sin búsqueda, solo dossier aprobado.
- Autorización: cuentas de servicio IAM y reglas de aplicación; no confiar en el prompt.
- Persistencia: Firestore para flujo; Cloud Storage para originales, versiones y hashes.
- Seguridad: todo contenido web es no confiable, proteger contra prompt injection.

## Diferencia de Agent Engine
Se posterga Agent Engine administrado hasta medir necesidad real de sesiones, escalado o runtime. El primer despliegue corre ADK dentro de Cloud Run.

## Validación antes de promoción
Comprobar costo/tokens por investigación, latencia, citas, bloqueos de aprobación, resistencia a inyección y factibilidad de consulta de fuentes.
