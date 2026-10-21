# Guion de demostración — Fast PoC v0.5

## Perfil TALLER (20 minutos de práctica dentro del bloque)
1. Abrir HTTPS y seleccionar artículo ficticio K-01; elegir corpus preprocesado de fuentes oficiales. Confirmar G1.
2. Ver progreso de job; desplegar tablero de afirmaciones con colores, evidencia por fragmento/página, IRD-1 y cobertura.
3. Abrir «Relaciones»: grafo de productos, países, documentos, empresa y socios. Seguir una arista al documento original. Repetir consulta con RAG y GraphRAG, y comparar evidencia.
4. Contrastar una afirmación «No verificable» donde falta evidencia; explicar que ausencia de mención no equivale a falsedad.
5. Confirmar G3 y crear versión corregida, luego cambiar formal/informal, revisar preservación de hechos.
6. Confirmar G4 («Revisé la propuesta») y exportar DOCX con marca editorial.

## Perfil REDACCION (demostración adicional)
Pista -> G1 plan -> H/D y G2 anomalías -> V + GraphRAG -> G3 expediente -> redacción -> G4 firma. Ningún agente aprueba las etapas.

## Escenarios obligatorios
- A: respaldo documental directo con página.
- B: contradicción exacta por objeto/tiempo.
- C: evidencia ausente -> abstención.
- D: fuente oficial interesada que emite declaración pero no prueba acto externo.
- E: fuente maliciosa con prompt injection -> ningún bypass.
- F: IP interna mediante redirección URL -> rechazo SSRF.
- G: 30 sesiones en prueba de carga, aislamiento y límites de cuota.
- H: puntuación IRD alta con cobertura baja, ambas visibles.

## Evidencias de ejecución
Reporte de CA01–CA18 y HA01–HA08, matriz de afirmaciones aprobada, traza de fuente/fragmento/autor/fecha, registro de G1-G4, costo/latencia, muestra de DOCX, comparación de GraphRAG vs RAG y capturas de errores.
