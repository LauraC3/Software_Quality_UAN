# Informe – Taller de Pruebas de Integración

**Grupo:** Velásquez – Caminos  
**Rama:** `grupo_velasquez_caminos`  
**Integrantes:** [Nombre completo 1] – [Código] | [Nombre completo 2] – [Código]

---

## Parte 2 – Análisis crítico

### ¿Las pruebas verifican la colaboración entre módulos?

No. Las pruebas originales solo comprobaban `assert result is True` sin verificar que `TaskStorage` persistiera datos ni que `Notifier.send` fuera invocado.

### Interacciones no validadas

- Service → Storage (`load` / `save`)
- Service → Notifier (`send` con mensaje correcto)
- Manejo de excepciones y consistencia de estado
- Títulos vacíos y duplicados en disco

### Fallos que pasaban desapercibidos

- `add_task` que devuelve `True` sin usar storage ni notifier (Parte 3)
- Fallo aleatorio del notifier (10 % con `random`)
- Tarea guardada aunque falle la notificación

---

## Parte 3 – Sabotaje controlado

**Modificación:** `add_task` siempre retorna `True` sin llamar dependencias.

**Resultado con tests originales:** siguen pasando (2/2).

**Por qué:** solo assertaban el booleano de retorno, no efectos en archivo ni llamadas al notificador.

**Debilidad:** confunden “usar objetos reales” con “verificar contratos de integración”.

**Con tests mejorados:** fallan porque `storage.load()` queda vacío y `notifier.send_calls == 0`.

---

## Parte 4 – Enfoques aplicados

| Enfoque | Implementación |
|---------|----------------|
| **Top-Down** | `TestTopDown` con `StorageStub` y `NotifierStub` |
| **Bottom-Up** | `StorageDriver` en `test_storage_driver.py` (4 pruebas) |
| **Sandwich** | `TestSandwich`: `TaskStorage` real + `NotifierStub` |
| **Big-Bang** | Tests iniciales (débiles, todo conectado sin aserciones de integración) |

---

## Parte 5 – Decisiones de diseño

- **Título vacío:** rechazado en `TaskService` (`False`).
- **Duplicado:** `False`, sin `save` ni `send`.
- **Fallo en storage:** `False`, sin notificar.
- **Fallo en notifier:** se revierte con segundo `save` (no dejar tarea a medias).
- **ConnectionError** se captura antes que `OSError` (herencia en Python 3).

---

## Parte 6 – Cobertura de código vs integración

- **Código:** líneas ejecutadas; puede ser 100 % con mocks que no reflejan comportamiento real.
- **Integración:** contratos entre módulos (datos persistidos, orden, errores, estado compartido).

Un 100 % unitario no garantiza el sistema integrado porque cada módulo puede estar bien aislado pero mal ensamblado (parámetros, orden, excepciones no propagadas).

**Señales de tests insuficientes:** pasan con `add_task` saboteado; no hay aserciones sobre disco o llamadas; dependencias aleatorias sin control.

---

## Parte 7 – Reflexión final

Las pruebas unitarias validan piezas; las de integración detectan errores de **ensamblaje**. Bottom-up conviene para validar `Storage` primero; top-down para reglas de `Service` con stubs; en microservicios, stubs para APIs externas y drivers para probar un servicio o repositorio de forma aislada.

---

## Diagrama

```
TaskService ──load/save──► TaskStorage (JSON)
     │
     └──send()──────────► Notifier
```
