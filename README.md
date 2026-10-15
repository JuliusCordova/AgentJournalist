# AgentJournalist
Fast PoC de una redacción agéntica verificable, basada en seis skills originales de Claude y diseñada mediante PA-SDD.

## Estado
**SDD v0.1 preparado; implementación pendiente.** No existe todavía servicio GCP desplegado ni ejecución ADK validada.

## Documentos
- [SPEC.md](SPEC.md): especificación mínima suficiente, requisitos, contratos y aceptación.
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md): arquitectura GCP y decisiones de patrones.
- [docs/DEMO-RUNBOOK.md](docs/DEMO-RUNBOOK.md): guion de demostración y evaluación.

## Fuentes
Patrones de [DevPattern](https://github.com/JuliusCordova/DevPattern). Los skills originales se mantienen como artefactos de referencia, no se transcriben a GitHub hasta completar su importación y control de versiones.

## Regla central
**La IA investiga y propone; los periodistas deciden.** Aprobaciones fuera del LLM, evidencia por afirmación y ninguna publicación automática.
