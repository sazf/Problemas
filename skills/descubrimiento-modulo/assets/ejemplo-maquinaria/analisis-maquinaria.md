# Análisis de descubrimiento — Módulo Maquinaria

| Campo | Valor |
|---|---|
| Módulo | Maquinaria y equipos pesados |
| Versión del documento | v0.1 — 2026-04-23 |
| Sistema destino | ERP propio (Colombia), implementación greenfield sobre stack existente |
| Alcance (incluye) | Inventario de maquinaria propia y de terceros; asignación a obras; alquileres entrantes y salientes; mantenimiento preventivo y correctivo; liquidación y facturación; horómetros, combustible y disponibilidad |
| Alcance (excluye) | GPS/telemetría en tiempo real (integración futura); gestión de proveedores de repuestos (módulo Compras); nómina del operario (módulo RRHH) |
| Autor(es) | Equipo de producto |

## 0. Resumen ejecutivo

- El módulo modela **un mismo activo máquina en varios modos**: propia en obra propia, propia alquilada a terceros, y de tercero alquilada a nosotros. Un solo catálogo con metadatos de propiedad, no tres módulos separados.
- La **máquina de estados** usa 10 estados con dos ramas de excepción (mantenimiento y fuera de servicio) y estados terminales explícitos. Ver `estados-maquinaria.mmd`.
- La **liquidación** (horas extra, combustible, traslado) es el cuello de botella de error humano hoy; la pantalla de liquidación está diseñada para forzar la lectura final antes de permitir la factura.
- La **adaptación Colombia** incluye factura electrónica DIAN con AIU cuando aplique, retegarantía en contratos de alquiler con obra civil, y bloqueo operativo si SST del operario no está al día.
- Queda abierto: política de tarifas dinámicas vs lista fija, manejo de subarriendo, integración con telemetría.

## 1. Benchmark de referentes

### 1.1 Referentes analizados

| Referente | Tipo | Documento consultado | Formato |
|---|---|---|---|
| SAP S/4HANA — Plant Maintenance (PM) | Global | SAP PM Business Process Overview | PDF offline (`fuentes/sap/sap-pm-overview.pdf`) |
| Odoo — Maintenance + Rental | Global | Odoo 17 Functional Docs (módulos maintenance, rental) | URL (consultada 2026-04-23) |
| Sinco ERP — Equipos y Maquinaria | LatAm / Construcción | Ficha funcional Sinco Equipos | URL (consultada 2026-04-23) |
| EquipmentShare T3 / Tenna | Vertical | Whitepaper público "Construction Equipment Management" | PDF offline (`fuentes/tenna/tenna-overview.pdf`) |

### 1.2 Síntesis comparativa

Los cuatro sistemas convergen en tres elementos: **activo como entidad de primera clase** con ciclo de vida propio, **orden de trabajo** como documento unificado para mantenimientos (preventivo, correctivo, inspección), y **horómetro** como métrica primaria que dispara tanto mantenimientos preventivos como facturación en alquileres. Todos separan el activo físico del "turno operativo" (asignación/alquiler) para poder consultar histórico.

Las divergencias importantes: SAP modela el mantenimiento como proyecto multinivel (plan → orden → operación → componente), lo que es sobredimensionado para una empresa mediana colombiana; Odoo es más liviano pero no resuelve la liquidación de alquileres con variables mixtas (horas, km, combustible, traslado) como unidad; Sinco entiende centros de costo y AIU, pero su UX está anclada a listas largas y no a tableros de disponibilidad; los verticales (Tenna, EquipmentShare) brillan en telemetría y disponibilidad visual, pero no tienen contabilidad colombiana.

La decisión de diseño que sale de esto: adoptar el **patrón Activo + Asignación + Orden de trabajo** de SAP/Odoo, con la **UX de tablero** de los verticales, y los **añadidos regulatorios colombianos** (AIU, retegarantía, SST) de Sinco. Evitar la sobre-estructuración de SAP.

### 1.3 Divergencias interesantes

- Ninguno resuelve bien el caso **"máquina alquilada a nosotros que a su vez asignamos a una obra"**: cambia quién asume combustible, mantenimiento y responsabilidad civil. Oportunidad de diferenciación.
- El tratamiento del operario varía: SAP/Sinco lo vinculan al activo al momento de la asignación; Odoo lo deja suelto. Inclinación a vincular (bloquea despachos con SST vencida).

### 1.4 Fuentes

1. `fuentes/sap/sap-pm-overview.pdf` — SAP PM Business Process Overview (descargado de help.sap.com).
2. `fuentes/tenna/tenna-overview.pdf` — Tenna whitepaper "Construction Equipment Management 101".
3. Odoo 17 Functional Documentation — https://www.odoo.com/documentation/17.0/applications/inventory_and_mrp/maintenance.html (2026-04-23).
4. Sinco ERP — Módulo Equipos y Maquinaria — https://www.sinco.co/ (ficha funcional pública, 2026-04-23).

## 2. Mapa de escenarios

### 2.1 Operación normal

- E1 — Registrar una máquina nueva adquirida (ficha + SOAT + revisión técnica + manuales).
- E2 — Asignar máquina propia a obra propia, con operario interno.
- E3 — Alquilar máquina propia a cliente externo, sin operario.
- E4 — Alquilar máquina propia a cliente externo, con operario nuestro.
- E5 — Tomar en alquiler máquina de un tercero para una obra propia.
- E6 — Programar mantenimiento preventivo por horómetro (cada 250 h).
- E7 — Programar mantenimiento preventivo por calendario (cada 6 meses).
- E8 — Ejecutar mantenimiento correctivo por falla reportada en obra.
- E9 — Registrar consumo de combustible diario.
- E10 — Liquidar alquiler mensual: horas base + horas extra + combustible + traslado.
- E11 — Trasladar máquina entre dos obras sin pasar por bodega.
- E12 — Cerrar obra y devolver todas las máquinas asignadas.

### 2.2 Excepciones y casos límite

- E13 — Máquina falla durante alquiler externo: quién asume costos y cómo se pausa la facturación.
- E14 — Operario queda con SST vencida durante un alquiler en curso.
- E15 — Contrato de alquiler vencido sin lectura final de horómetro.
- E16 — Liquidación con valor negativo (devolución de depósito/garantía).
- E17 — Robo o siniestro total: anulación y proceso de baja con aseguradora.
- E18 — Subarriendo no autorizado detectado en obra.
- E19 — Discrepancia entre horómetro físico y el registrado (rollover, falla del contador).
- E20 — Cierre de mes con alquileres en curso: cómo se refleja en contabilidad.

### 2.3 Integraciones con otros módulos

- E21 — Al facturar un alquiler, generar factura electrónica DIAN con retenciones aplicables (módulo Facturación).
- E22 — Al ejecutar mantenimiento, descontar repuestos del inventario (módulo Inventario).
- E23 — Al asignar operario, validar su contrato y SST (módulos RRHH + SST).
- E24 — Al cerrar alquiler, afectar centros de costo de la obra (módulo Contabilidad).
- E25 — Al tomar en alquiler máquina de tercero, generar cuenta por pagar al proveedor (módulo Compras).

## 3. Máquina de estados

### 3.1 Entidad: Máquina (activo físico)

- **Estados**: `borrador`, `disponible`, `asignada`, `en_traslado`, `operando`, `detenida`, `en_mantenimiento`, `alquilada`, `fuera_de_servicio`, `dada_de_baja`.
- **Diagrama**: [`estados-maquinaria.mmd`](./estados-maquinaria.mmd)

Tabla de transiciones (extracto de las más relevantes):

| De | A | Evento | Guarda |
|---|---|---|---|
| borrador | disponible | activar | ficha completa + SOAT y revisión técnica vigentes |
| disponible | asignada | asignar | no hay solape de fechas con otra asignación |
| asignada | en_traslado | despachar | contrato firmado si aplica; operario con SST vigente |
| en_traslado | operando | llegada_a_obra | — |
| operando | en_mantenimiento | reportar_falla | — |
| en_mantenimiento | disponible | cerrar_mantenimiento | pruebas OK + repuestos descontados |
| operando | alquilada | iniciar_alquiler | contrato de alquiler firmado + lectura inicial |
| alquilada | disponible | cerrar_alquiler | lectura final registrada + liquidación generada |
| disponible | fuera_de_servicio | retirar | — |
| fuera_de_servicio | dada_de_baja | dar_de_baja | rol gerencia + motivo documentado |

Estados terminales: `dada_de_baja`. Estados de excepción: `en_mantenimiento`, `fuera_de_servicio`. No hay rollback automático; toda reversión se modela como transición explícita con evento nombrado.

### 3.2 Entidad: Alquiler (documento transaccional)

- **Estados**: `borrador`, `pendiente_aprobacion`, `aprobado`, `en_curso`, `cerrado`, `anulado`.
- Patrón 1 de `estados-patrones.md` (documento con aprobación), con la particularidad de que `en_curso` implica una máquina vinculada en estado `alquilada`.

## 4. Restricciones de negocio

### 4.1 Reglas duras (bloquean)

1. **RD-1** — No se puede activar una máquina (pasar `borrador → disponible`) si SOAT o revisión técnica están vencidos. *Entidad: Máquina. Estado: borrador.*
2. **RD-2** — No se puede asignar una máquina si hay solape de fechas con otra asignación vigente. *Entidad: Máquina. Estado: disponible.*
3. **RD-3** — No se puede asignar una máquina en `en_mantenimiento` o `fuera_de_servicio`. *Entidad: Máquina.*
4. **RD-4** — No se puede despachar con operario si EPS, ARL o AFP del operario están vencidas. *Entidad: Asignación. Estado: asignada.*
5. **RD-5** — No se puede cerrar un alquiler sin lectura final de horómetro registrada. *Entidad: Alquiler. Estado: en_curso.*
6. **RD-6** — No se puede facturar un alquiler sin su liquidación aprobada. *Entidad: Alquiler.*
7. **RD-7** — No se puede dar de baja una máquina sin motivo documentado y visto bueno del rol Gerencia. *Entidad: Máquina. Estado: fuera_de_servicio.*
8. **RD-8** — No se puede registrar un horómetro inferior al último registrado (salvo rollover explícito autorizado). *Entidad: Máquina.*

### 4.2 Reglas blandas (advertencias / defaults)

1. **RB-1** — Advertir al usuario si faltan menos de 30 días para vencimiento de SOAT/revisión; permitir operar.
2. **RB-2** — Advertir si el mantenimiento preventivo está atrasado más de 10% del ciclo; permitir operar con confirmación.
3. **RB-3** — Sugerir incluir traslado en la liquidación si la obra está a más de 50 km de la bodega.

### 4.3 Reglas cruzadas

1. **RC-1** — El cierre contable del mes (módulo Contabilidad) congela lecturas y liquidaciones; el módulo Maquinaria debe impedir registros retroactivos en meses cerrados.
2. **RC-2** — La factura electrónica (módulo Facturación) depende de que el alquiler esté `cerrado` con liquidación aprobada; ambos módulos deben compartir el evento de cierre.
3. **RC-3** — El descuento de repuestos (módulo Inventario) ocurre al cerrar la orden de mantenimiento; si falta stock, la orden queda en estado `bloqueada_por_inventario`.

## 5. Roles y permisos

### 5.1 Roles identificados

- **Operario**: quien opera la máquina en obra. Registra horómetro y consumos diarios.
- **Supervisor de obra**: asigna operarios a máquinas, aprueba reportes diarios, escala fallas.
- **Despachador / logística**: decide asignaciones entre obras, gestiona traslados.
- **Taller / mantenimiento**: programa y ejecuta mantenimientos; aprueba cierres de orden.
- **Administrativo / facturación**: genera liquidaciones y facturas electrónicas.
- **Gerencia**: aprueba bajas, anulaciones y excepciones.
- **Auditor**: acceso de lectura a todo; no puede modificar.

### 5.2 Matriz acción × rol

| Acción | Operario | Supervisor | Despachador | Taller | Admin/Fact | Gerencia | Auditor |
|---|---|---|---|---|---|---|---|
| Crear máquina | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Activar (borrador → disponible) | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Asignar a obra | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ |
| Registrar horómetro diario | ✅ | ✅ | ⚠️ | ❌ | ❌ | ❌ | ❌ |
| Programar mantenimiento | ❌ | ⚠️ | ❌ | ✅ | ❌ | ✅ | ❌ |
| Cerrar mantenimiento | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ |
| Iniciar alquiler externo | ❌ | ❌ | ✅ | ❌ | ⚠️ | ✅ | ❌ |
| Liquidar alquiler | ❌ | ❌ | ⚠️ | ❌ | ✅ | ✅ | ❌ |
| Emitir factura electrónica | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Anular alquiler | ❌ | ❌ | ⚠️ | ❌ | ⚠️ | ✅ | ❌ |
| Dar de baja máquina | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Ver todo (solo lectura) | ⚠️ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

⚠️ indica "con aprobación de rol superior" o "solo en casos específicos".

### 5.3 Permisos por estado

| Estado máquina | Operario | Supervisor | Despachador | Taller | Admin | Gerencia |
|---|---|---|---|---|---|---|
| borrador | ❌ | ❌ | ✏️ | ✏️ | ❌ | ✏️ |
| disponible | 👁 | 👁 | ✏️ | ✏️ | 👁 | ✏️ |
| asignada | 👁 | ✏️ | ✏️ | ❌ | 👁 | ✏️ |
| operando / alquilada | ✏️ (horóm) | ✏️ | ✏️ | 👁 | 👁 | ✏️ |
| en_mantenimiento | ❌ | 👁 | ❌ | ✏️ | 👁 | ✏️ |
| dada_de_baja | 👁 | 👁 | 👁 | 👁 | 👁 | 👁 |

✏️ = puede editar campos del estado. 👁 = solo lectura.

## 6. Flujos y mockups

### 6.1 Flujos críticos

- [`flujo-alquiler.mmd`](./flujo-alquiler.mmd) — Ciclo completo de un alquiler saliente, desde solicitud hasta facturación y cierre.
- `flujo-mantenimiento.mmd` (pendiente) — Preventivo por horómetro con descuento de repuestos.
- `flujo-traslado.mmd` (pendiente) — Movimiento entre dos obras sin bodega intermedia.
- `flujo-baja.mmd` (pendiente) — Siniestro/robo con aseguradora.

### 6.2 Mockups de pantallas clave

#### 6.2.1 Tablero de disponibilidad

**Propósito.** Vista principal del despachador. Muestra en tiempo real la flota con estado y asignación actual, permite filtrar y actuar sobre varias máquinas.

**Layout.**

```
+------------------------------------------------------------------+
| Maquinaria | [+ Nueva] [Importar]                    Usuario ▾   |
+------------------------------------------------------------------+
| Filtros: Estado ▾ | Tipo ▾ | Obra ▾ | Operario ▾ | Buscar ___    |
+------------------------------------------------------------------+
| Disponibles (12) | Operando (34) | Alquiladas (7) | Mnto (5)     |
+------------------------------------------------------------------+
| # Int | Tipo       | Estado       | Obra/Cliente | Operario | Horóm. | Acciones |
|-------+------------+--------------+--------------+----------+--------+----------|
| EX-01 | Excavadora | 🟢 operando  | Obra Chía    | J. Pérez | 1.234  | [···]    |
| RE-03 | Retro      | 🟡 alquilada | Constr. Ltda.| —        | 2.890  | [···]    |
| VO-07 | Volqueta   | 🔴 mnto      | (taller)     | —        | 4.120  | [···]    |
| ...                                                                       |
+------------------------------------------------------------------+
| Mostrando 12 de 58 · [<] 1 2 3 [>]                                |
+------------------------------------------------------------------+
```

**Datos visibles.** # interno (clickeable → ficha), tipo (chip de color), estado (chip mapeado a §3), obra/cliente (clickeable), operario (clickeable), horómetro actual, menú contextual.

**Acciones disponibles.**
- "Asignar a obra" (despachador, supervisor). Sujeta a RD-2, RD-3, RD-4.
- "Enviar a mantenimiento" (taller, despachador). Transición a `en_mantenimiento`.
- "Exportar CSV" (cualquiera con acceso).
- "Nueva máquina" (despachador, taller, gerencia).

**Comportamiento por estado y vacíos.** Sin datos: empty state con CTA "Registrar primera máquina". Error de red: banner sin perder filtros. Sin permisos: acciones deshabilitadas con tooltip explicativo.

#### 6.2.2 Ficha de máquina

**Propósito.** Detalle completo de un activo: datos, documentos, historial de asignaciones, mantenimientos, consumos.

(Descripción abreviada en este ejemplo — seguir el mismo esqueleto.)

#### 6.2.3 Liquidación de alquiler

**Propósito.** Pantalla crítica previa a facturar. Consolida lectura inicial/final, horas extra, combustible, traslados y descuentos en un resumen auditable antes de emitir la FE.

(Descripción abreviada.)

#### 6.2.4 Orden de mantenimiento

**Propósito.** Documento unificado para preventivo y correctivo, con checklist, repuestos y pruebas.

(Descripción abreviada.)

## 7. Adaptación Colombia

- **Facturación electrónica DIAN.** Las liquidaciones de alquiler se convierten en factura electrónica (UBL 2.1) con CUFE; notas crédito cuando se hace devolución de depósito. Si el alquiler incluye obra civil asociada, usar **AIU** (Administración 10% / Imprevistos 5% / Utilidad 10% — valores ejemplo, parametrizables) sobre la base gravable. Eventos RADIAN si el cliente descuenta la factura.
- **Retenciones.** Aplicar ReteFuente según concepto "arrendamiento de bienes muebles" y cuantías mínimas vigentes; ReteIVA si el cliente es agente retenedor; ReteICA según municipio de la obra (no del emisor). Parametrizar por municipio.
- **Retegarantía.** En alquileres vinculados a obra civil pública o con cláusula contractual, retener 5% del valor bruto como garantía, liberable al cierre de obra con paz y salvo.
- **SOAT y revisión técnico-mecánica.** Bloqueo duro (RD-1) para activación y despacho si alguno está vencido. Advertencia (RB-1) a 30 días.
- **SST del operario (Resolución 0312/2019).** Validar EPS, ARL, AFP y examen médico ocupacional vigentes antes de permitir transición a `en_traslado` u `operando` con operario asignado. ARL mínima nivel de riesgo IV o V para maquinaria pesada.
- **Contratos.** Contrato de alquiler con firma electrónica (Ley 527/1999) aceptable. Para OPS de operarios independientes, cláusulas específicas de afiliación y seguridad social.
- **Centros de costo.** Cada asignación afecta un centro de costo de obra; el consumo de combustible y las horas máquina se imputan ahí para costeo directo. Integra con PUC de construcción cuando aplique.
- **Sectorial construcción.** Formatos de acta de entrega parcial, actas de obra mensuales, cumplimiento de AIU en facturación con contratistas del sector público.

## 8. Preguntas abiertas y supuestos

- **P1** — ¿Se permite subarriendo y bajo qué condiciones? *Sugerido: preguntar a Gerencia y Legal.*
- **P2** — ¿Tarifa fija mensual o dinámica por demanda? *Sugerido: modelo de pricing con comercial.*
- **P3** — ¿Integración con GPS/telemetría en v1 o v2? *Sugerido: priorización con producto.*
- **S1** — Se asume que el cierre contable mensual es hard; no se permiten ajustes retroactivos.
- **S2** — Se asume una sola moneda (COP); operaciones en USD quedan fuera de alcance.

## 9. Próximos pasos sugeridos

- Pasar este documento a Codex para que proponga modelo de datos y migraciones sobre el repo del ERP.
- Validar la matriz de roles con el responsable operativo en obra real antes de codificar.
- Prototipar el tablero de disponibilidad (mockup 6.2.1) en alta fidelidad para testeo con despachador.
- Confirmar con Contabilidad el tratamiento exacto de AIU y retegarantía por tipo de cliente.
- Iterar los flujos faltantes (mantenimiento, traslado, baja) antes de cerrar discovery.
