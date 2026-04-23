# Contexto Colombia — checklist de adaptación local

La sección "Adaptación Colombia" del entregable no es un checklist genérico pegado como apéndice. Es un ensayo concreto con los elementos normativos y operativos que tocan al módulo específico. Este archivo es la **lista de estímulos** para no olvidar categorías; cada ítem debe traducirse a algo concreto del dominio.

## Tributario

- **Régimen**: responsable de IVA vs no responsable (antes "régimen común/simplificado"). ¿El módulo emite documentos que dependen del régimen del tercero?
- **IVA**: tarifas aplicables (0%, 5%, 19%). Excluidos y exentos. ¿Cómo se parametriza por producto/servicio/cliente?
- **Retenciones en la fuente**:
  - Renta (según concepto y cuantía mínima).
  - IVA (ReteIVA).
  - ICA (ReteICA, varía por municipio; Bogotá, Medellín, Cali tienen sus tarifas).
- **Autorretenciones**: régimen especial, CREE ya derogado pero autorretención de renta persiste para ciertos contribuyentes.
- **Impuesto al consumo** (INC): restaurantes, bolsas plásticas, vehículos, datos móviles.
- **Gran Contribuyente**: cambia el esquema de retenciones.

Para cada punto: indicar qué campos del dominio se afectan y dónde se parametriza.

## Facturación electrónica DIAN

- **Tipos de documento electrónico**:
  - Factura electrónica de venta.
  - Nota crédito y nota débito electrónicas.
  - Documento soporte en adquisiciones con no obligados.
  - Documento equivalente (POS, tiquetes de máquina registradora).
  - Documento soporte de pago de nómina electrónica.
- **Eventos**: acuse de recibo, aceptación/rechazo, recepción del bien o servicio.
- **RADIAN**: para factura electrónica como título valor (cesión, endoso, financiación).
- **CUFE / CUDE / CUDS / CUNE**: códigos únicos. Saber cuál aplica a cada documento.
- **Proveedor tecnológico**: Factory, Facture, Siigo, Alegra, Carvajal, The Factory HKA, Siesa, etc. El módulo puede integrar con uno o dejarlo abierto.
- **Plazos de emisión y reporte**: la factura se debe emitir y reportar en tiempo real o con márgenes estrechos.

## Nómina electrónica

- Obligatoriedad por tramos según número de empleados y fecha.
- Documento soporte de pago de nómina (DSPN) y notas de ajuste.
- Frecuencias y fechas de reporte.
- Integración con seguridad social (PILA, operadores: Aportes en Línea, SOI, Simple, Mi Planilla).

## RUT y terceros

- Campos mínimos: número de identificación (NIT/CC/CE/pasaporte), dígito de verificación, razón social / nombres, tipo de persona (natural/jurídica), régimen, responsabilidades (códigos 47 – responsable IVA, etc.), actividad económica CIIU, dirección fiscal.
- Validaciones: DV calculado, CIIU existente, municipio DANE.
- Terceros extranjeros: tratamiento especial sin RUT.

## Seguridad y Salud en el Trabajo (SST)

Cuando el módulo toca personas en operación (obra, maquinaria, transporte):

- **Resolución 0312 de 2019**: estándares mínimos según tamaño y nivel de riesgo (I a V).
- Afiliaciones a EPS, ARL, AFP, caja de compensación — estado vigente antes de permitir operar.
- Exámenes médicos ocupacionales (ingreso, periódicos, egreso).
- Dotación, EPP, inducción, reinducción.
- Reporte de ATEL (accidentes de trabajo y enfermedad laboral).
- COPASST, Comité de Convivencia.

Para módulos de maquinaria, construcción, agro, transporte: pensar qué bloqueos aplican si los documentos SST no están al día.

## Contratos y documentos legales

- **Contratos laborales**: fijo, indefinido, obra o labor, prestación de servicios (OPS — tener cuidado, es contrato civil, no laboral).
- **Contratos civiles/comerciales**: arrendamiento, suministro, obra.
- **Firma electrónica**: Ley 527 de 1999, firma digital (certificado) vs firma electrónica simple. Validez probatoria.
- **Documentos soporte**: acta de inicio, acta de entrega, paz y salvo, certificado de cumplimiento.

## Centros de costo y contabilidad

- Plan Único de Cuentas (PUC) para comerciantes (Decreto 2650) o sectorial (construcción, salud, educación).
- Centros de costo, subcentros, proyectos como dimensiones adicionales.
- NIIF Plenas vs NIIF Pymes vs microempresas — el régimen afecta valuación y revelaciones.
- Periodo contable: meses que se cierran; no se permiten movimientos en periodos cerrados.

## Sectoriales frecuentes

- **Construcción**: actas de obra, AIU (Administración + Imprevistos + Utilidad), retegarantía, formatos de entrega parcial.
- **Transporte de carga**: RNDC (Registro Nacional de Despachos de Carga), manifiestos, remesas, estudio de seguridad.
- **Transporte de pasajeros**: tarjetas de operación, permisos.
- **Salud**: RIPS (Registro Individual de Prestaciones en Salud), habilitación, historia clínica.
- **Agro**: ICA (registros fitosanitarios), INVIMA (cuando hay procesamiento).
- **Comercio exterior**: VUCE, DIAN aduanas, declaraciones de importación/exportación.
- **Servicios públicos**: CREG, SUI, regulación SSPD.

## Cómo escribir la sección en el entregable

Tres reglas:

1. **Nombrar el acto administrativo, el documento o el código.** No "cumplir con DIAN" — decir "emitir factura electrónica en formato UBL 2.1 con CUFE, notas crédito y eventos RADIAN cuando aplique".
2. **Conectar con la máquina de estados y las restricciones.** Ejemplo: "Una máquina no puede pasar a `operando` si el operario no tiene EPS vigente (validación contra archivo plano de ARL)".
3. **Señalar lo que queda fuera o depende del integrador.** Es válido decir "la integración con proveedor tecnológico de FE queda fuera del alcance del módulo, pero los hooks son X".
