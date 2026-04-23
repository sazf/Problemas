# Guía de benchmark con WebSearch

El benchmark no es "listar features de SAP". Es **extraer patrones convergentes** que permitan tomar decisiones de diseño con evidencia. Tres referentes bien analizados valen más que ocho mal revisados.

## Cómo seleccionar referentes

Apuntar a 3–5, mezclando:

- **Globales consolidados** — SAP S/4HANA, Oracle Fusion/NetSuite, Microsoft Dynamics 365, Odoo. Útiles para ver el "estado del arte" y convenciones que ya nadie discute.
- **Regionales LatAm / Colombia** — Sinco, Siigo, Alegra, World Office, Helisa, TNS, Ofima, Enlace Operativo. Útiles para detectar adaptaciones locales (DIAN, centros de costo, retenciones, nómina electrónica) que los globales no traen de fábrica.
- **Verticales específicos cuando aplique** — p. ej. para maquinaria/equipos: Tenna, EquipmentShare, HCSS; para construcción: Procore, Buildertrend; para agro: Cropwise, Siembro; para transporte: Samsara, SimpliRoute.

Regla práctica: al menos **2 globales + 1 regional** por módulo. Si el dominio tiene mucha regulación local (nómina, facturación, impuestos), subir el regional a 2.

## Queries de WebSearch efectivas

- `"<referente>" "<módulo>" documentation pdf`
- `"<referente>" "<módulo>" user manual`
- `"<referente>" "<módulo>" business process`
- `site:help.<referente>.com <módulo>`
- `"<referente>" <entidad típica> state machine`
- Para LatAm: agregar `"<referente>" <módulo> colombia` o `dian`.

Cuando el sitio oficial no da, buscar **whitepapers de partners** (Deloitte, Accenture implementadores) y **foros oficiales** (SAP Community, Odoo Forum).

## Protocolo de recolección de fuentes

Por cada referente, seguir este orden:

1. **Buscar documento oficial del módulo.** Si existe PDF descargable desde el sitio del proveedor o de un partner reconocido:
   - Descargar a `analisis-<modulo>/fuentes/<referente>/<nombre-descriptivo>.pdf`.
   - Nombrar el archivo de forma autoexplicativa: `sap-s4hana-eam-overview-2024.pdf`, no `doc1.pdf`.
   - Registrar en la tabla de fuentes del `.md` maestro con la ruta relativa.

2. **Si solo hay manual en línea** (HTML, knowledge base pública, video):
   - Registrar URL + título + fecha de consulta (`YYYY-MM-DD`) en la tabla de fuentes.
   - No intentar descargar el HTML: se desactualiza y suele violar términos.

3. **Nunca** intentar acceder a:
   - Contenido que requiera login.
   - Paywalls (Gartner, Forrester pagos, etc.).
   - Repositorios privados o documentos marcados como confidenciales.
   - Material que explícitamente prohíba la redistribución si la intención es compartir el análisis.

4. **Si no hay fuente primaria disponible**, anotarlo explícitamente: *"No se encontró documentación oficial pública del módulo X en <referente>; síntesis basada en artículos de terceros y foros oficiales."* Es mejor admitir la limitación que fabricar citas.

## Qué extraer de cada referente

No copiar el índice del manual. Buscar estos **patrones** en particular:

- **Entidades principales** — qué sustantivos son de primera clase en el modelo (orden de trabajo, activo, contrato, tercero, movimiento).
- **Estados recurrentes** — qué máquina de estados usan y cómo nombran los estados. Muchas veces hay un canon (`draft → confirmed → done → cancelled`) que vale la pena conservar.
- **Integraciones típicas** — con qué otros módulos se conecta sí o sí (contabilidad, inventario, compras, RRHH).
- **Decisiones de UX dominantes** — ¿usan un tablero de cards? ¿una lista plana? ¿un calendario? Si 4 de 5 coinciden, probablemente hay razón.
- **Lo que ninguno resuelve bien** — a veces el valor diferencial del producto propio está justo ahí. Anotarlo.
- **Obligaciones regulatorias locales** — cómo lo resuelven los regionales y cómo lo ignoran los globales.

## Cómo escribir la síntesis

La sección 1.2 del `.md` maestro (síntesis comparativa) es un **ensayo corto**, no una tabla. Dos a cuatro párrafos que respondan:

1. ¿Qué tienen en común estos sistemas en la forma de modelar este módulo?
2. ¿Dónde divergen y por qué?
3. ¿Qué de todo esto tiene sentido copiar, y qué no encaja con el contexto del usuario?

La tabla de referentes (1.1) es inventario; el valor analítico está en la síntesis.

## Errores a evitar

- **Benchmark-feature-list.** Salir con una tabla de 50 features ticados. No aporta; lo importante es el patrón.
- **Sesgo al primer referente.** Si se empieza por SAP, todo el análisis termina modelado como SAP. Rotar el orden al escribir.
- **Confundir marketing con diseño.** Las páginas de producto venden beneficios; la documentación técnica es la que revela modelo y estados.
- **Ignorar los regionales.** Para Colombia, los regionales suelen ser los únicos que modelan bien retenciones, centros de costo y facturación electrónica.
