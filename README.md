# AgentJournalist — SDD v0.5 | GCP Fast PoC
Fast PoC de una redacción agéntica verificable, basada en seis skills originales de Claude y diseñada mediante PA-SDD.

## Estado
**SDD v0.5 listo para iniciar implementación, no desplegado.** Incluye dos perfiles: `TALLER` (verificación guiada del artículo K-01) y `REDACCION` (investigación editorial completa). Seis agentes y dos motores: GraphRAG y evaluación de evidencia. La fuente de verdad es [SPEC.md](SPEC.md).


## Documentos
- [SPEC.md](SPEC.md): especificación mínima suficiente, requisitos, contratos y aceptación.
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md): arquitectura GCP y decisiones de patrones.
- [docs/DEMO-RUNBOOK.md](docs/DEMO-RUNBOOK.md): guion de demostración y evaluación.

## Fuentes
Patrones de [DevPattern](https://github.com/JuliusCordova/DevPattern). Los skills originales se mantienen como artefactos de referencia, no se transcriben a GitHub hasta completar su importación y control de versiones.

## Regla central
**La IA investiga y propone; los periodistas deciden.** Aprobaciones fuera del LLM, evidencia por afirmación y ninguna publicación automática.


## Capacidad adicional: GraphRAG
La Fast PoC incluye un grafo interactivo de entidades, relaciones y evidencias, con consultas multi-hop y benchmark visible contra recuperación tradicional. Consultar [ADR GraphRAG](docs/ARCHITECTURE.md#graphrag-fast-demo-decisión-adr-002) y [SPEC.md](SPEC.md#15-graphrag--alcance-demostrativo-obligatorio-v02).


## Evaluación de noticias e incertidumbre
El SDD incorpora veredictos por afirmación, trazabilidad, métricas de clasificación, cobertura, abstención y probabilidad de falsedad **solo si ha sido calibrada y evaluada estadísticamente**. No se muestran porcentajes inferidos libremente por el LLM. Ver [SPEC](SPEC.md#16-métricas-de-verificación-probabilística-v03).

## Inicio para desarrolladores
1. Leer [SPEC v0.5](SPEC.md), especialmente §17 (TALLER), §18 (Claude) y §19 (hardening).
2. Revisar [contratos de agentes](docs/AGENT-CONTRACTS.md), [decisiones de arquitectura](docs/ARCHITECTURE-DECISIONS-v0.5.md) y [matriz de trazabilidad](docs/TRAZABILIDAD-TALLER.md).
3. Ejecutar primero un vertical slice con corpus K-01, extracción de claims, citas verificables y abstenciones.
4. Implementar GraphRAG sobre el mismo corpus, contrastar con RAG baseline.
5. Añadir aprobaciones persistidas, exportación DOCX, seguridad de URL y carga concurrente.

**Estado:** documentación versionada; agentes ADK, UI React, API FastAPI e infraestructura aún pendientes.

## Material de taller
- [Guion v0.5](docs/DEMO-RUNBOOK-v0.5.md)
- [Trazabilidad y criterios](docs/TRAZABILIDAD-TALLER.md)
- [Patrones de referencia DevPattern](https://github.com/JuliusCordova/DevPattern)
