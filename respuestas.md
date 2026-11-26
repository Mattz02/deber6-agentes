# Respuestas

## Subagente local

Usé Codex CLI `0.169.0`. Archivo: `.codex/agents/revisor-seguridad.toml`; formato TOML. Inicié la revisión con `codex exec --trust --json -s read-only`, desactivando apps, plugins y búsqueda web; el comando completo está en la evidencia. Pedí: «Lanza explícitamente UN subagente local revisor-seguridad sobre usuarios.py».

Codex lo reconoció como `agent_type="revisor-seguridad"` y ejecutó `collaboration.spawn_agent` con `task_name="revisor_seguridad"` y `fork_turns="none"`. El hilo separado y su respuesta aparecen en la evidencia. Es delegación del harness local: no hubo descubrimiento ni mensajes del protocolo A2A.

Solo autoricé lectura y búsqueda. El TOML no impone un sandbox ni restringe técnicamente la lectura a un archivo: el hijo heredó `read-only` de la sesión padre. Verifiqué sus referencias y comprobé con `git status` y `git diff HEAD -- usuarios.py` que el archivo no cambió. [Permisos de subagentes](https://learn.chatgpt.com/docs/agent-configuration/subagents).

## A2UI

Salida convencional, basada en el hallazgo del subagente:

| Archivo | Línea | Severidad | Problema |
|---|---|---|---|
| `usuarios.py` | 21 | Alta | `nombre` se concatena en SQL y permite inyección. |

Catálogo permitido: `Card` agrupa; `Text` muestra texto; `Button` emite únicamente `pedir_revision`.

*Ilustración didáctica; no es un mensaje A2UI estándar válido.* [Archivo JSON](ui_ejemplo.json).

```json
{
  "component": "Card",
  "children": [
    { "component": "Text", "text": "usuarios.py · línea 21 · severidad: alta" },
    { "component": "Text", "text": "nombre se concatena en SQL." },
    {
      "component": "Button",
      "label": "Solicitar revisión adicional",
      "event": "pedir_revision",
      "data": { "archivo": "usuarios.py", "linea": 21, "hallazgo": "sql-concatenado" }
    }
  ]
}
```

Un LLM puede producir la descripción o una herramienta devolver una plantilla fija. A2UI es el formato/protocolo de descripción, no un LLM. Generative UI es el enfoque general; una tabla Markdown o un artefacto interactivo no demuestran conformidad con A2UI.

Flujo: agente o plantilla produce la descripción → cliente valida esquema, catálogo, datos y acciones → renderer dibuja componentes implementados → usuario pulsa → evento de retorno. MCP no es obligatorio para renderizar. [Flujo A2UI](https://a2ui.org/concepts/overview/).

El cliente rechazaría JSON inválido y componentes desconocidos; bloquearía acciones peligrosas y URLs no autorizadas. El catálogo es una barrera, no toda la seguridad. [Catálogos](https://a2ui.org/concepts/catalogs/).

`pedir_revision` devolvería archivo, línea e identificador del hallazgo. El backend comprobaría la identidad, la autorización sobre el archivo y la validez del evento antes de encargar otra revisión. El botón por sí solo no autoriza esa operación.

## Preguntas finales

1. Dos especialistas separan seguridad y pruebas, como muestra el flujo. Mejoran el enfoque, pero agregan coordinación, tiempo y consumo.
2. Verifiqué la llamada, el hilo hijo y sus referencias en el código. Si fallara, pediría una corrección; si persistiera, avisaría a la persona.
3. Lectura y búsqueda, con sandbox heredado de solo lectura. El archivo TOML por sí solo no garantiza esos permisos ni limita qué archivos se pueden leer.
4. Los locales bastan dentro de una herramienta; A2A compensa al delegar a otro equipo con infraestructura e interfaz independientes.
5. Ante `input-required`, pediría el dato y continuaría la tarea. Ante `failed`, informaría el error y evaluaría un nuevo intento.
6. El cliente valida esquema, catálogo, datos, acciones y enlaces. El backend verifica identidad, autorización y alcance antes de atender el evento.
7. Una plantilla fija conviene cuando campos y acciones se repiten, como en esta tarjeta; facilita validar y mantener una presentación consistente.

## Evidencias

![Flujo](evidencias/parte1_flujo.png)
![Subagente completo](evidencias/parte2_subagente.png)
![Invocación y respuesta](evidencias/parte2_llamada.png)
![Agent Card y secuencia](evidencias/parte3_a2a.png)
![Tarjeta propuesta](evidencias/parte4_interfaz.png)
