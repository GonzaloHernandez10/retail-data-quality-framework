# :memo: Proyecto: Framework de Integridad y Análisis de Ventas Retail
Rol: Data Analyst | Stack: Python (Pandas), Business Intelligence, Data Engineering, Data Quality, Data Governance

# :book: 1. Contexto y Problema de Negocio

La empresa "Imaginación" enfrentaba una crisis de confianza en sus reportes financieros. Las métricas de ventas presentaban discrepancias entre los ingresos reportados y los reales, debido a la falta de un        proceso de Data Quality Governance.
Impacto: Decisiones comerciales basadas en datos inflados, riesgos en la planeación de inventarios y pérdida de credibilidad operativa.

# :dart: 2. Objetivos Estratégicos

1. Aseguramiento de la veracidad de los datos: Implementar un motor de reglas de negocio para filtrar registros no confiables y obtener un Golden DataSet.
2. Identificación de los datos no confiables: Recopilación de los datos no confiables en un Dirty DataSet.
3. Trazabilidad de errores: Diseñar una taxonomía de errores (Error Tagging) para identificar las fallas que presentan los datos.
4. Conciliación financiera: Generar un Bridge Report que explique la diferencia entre el dato bruto y el dato validado.
5. Reporte de causa raíz: Generar un RCA para identificar los posibles patrones de fallo.

# :bug: 3. Taxonomía de Errores

| CÓDIGO DE ERROR | DESCRIPCIÓN | TIPO | ACCIÓN
|:---:|:---|:---:|:---|
| CRITICAL_ERROR | Fallo en campo crítico o valores negativos en cantidad o precio. | Crítico | Eliminar del análisis |
| INVALID_RANGE | Descuento fuera de rango (>35%). | No crítico | Corregir (Imputar 0 o max) |
| DUPLICATED | Transacción idéntica duplicada | Duplicado | Conservar solo uno |
| INVALID_LOGIC | Inconsistencia | No crítico | Corregir (Inputar a 0% o 'Unknown')

# :gear: 4. Arquitectura de Calidad de Datos (Reglas de Negocio)

A. Reglas Críticas
  1. Si falla una, el registro se categoriza como NON-RELIABLE y se excluye del análisis financiero final.
  2. Integridad de transacción: quantity > 0 y unit_price > 0.
  3. Completitud: Campos críticos (order_id, category, quantity, unit_price, order_date) no pueden ser nulos.

B. Reglas de Negocio 
  1. Si falla, el registro se categoriza como CORRIGIBLE mediante reglas de imputación.
  2. Umbral de descuento: Discount ∈ [0,0.35]. Los valores fuera de rango se imputan a la mediana o al límite superior según política.
  3. Imputación lógica: Si discount es nulo, se imputa 0%. Si region es nula, se imputa como "Unknown".

C. Reglas de Unicidad
  1. Deduplicación: Identificación de registros idénticos por atributos clave, conservando solo la primera instancia (Timestamp más antiguo).

# :building_construction: 5. Metodología de Implementación

1. Exploratory Data Quality (EDQ): Diagnóstico inicial de nulos críticos, duplicados y valores inválidos.
2. Data Labeling & Categorization: Creación de columnas de control:
   - is_valid: Flag booleano de confiabilidad.
   - error_code: Etiqueta específica del error presentado.
   - category_value: Etiqueta específica de la categoría del registro.
3. Data Cleaning & Imputation: Aplicación de correcciones lógicas sin alterar la fuente original (Staging).
4. Recálculo de Métricas: Re-proceso de gross_sales y net_sales basado en datos limpios.

# :mag_right: 6. Análisis de Causa Raíz (RCA) - Hallazgos

Tras auditar los registros excluidos, se identificaron los siguientes patrones de falla:
1. Falla de integridad estructural: El 96% de los errores fueron NULL_CRITICAL. Esto indica una falta de campos obligatorios en el formulario de captura o fallos en la transferencia de datos (ETL) desde la base      de datos origen.
2. Inconsistencia de precios/cantidades: Los errores de tipo INVALID_VALUE sugieren que el sistema permite la entrada de números negativos. Esto es un fallo de validación en el Front-end que debe corregirse para     evitar ruido financiero.
3. Problema de sincronización: Se detectaron 104 registros duplicados. Esto ocurre típicamente cuando los procesos de carga de datos se ejecutan dos veces sin una validación de "llave primaria" o cuando hay          reintentos automáticos tras una falla de red.
4. Foco crítico: La categoría "Sports" concentra el mayor volumen de fallas, lo que requiere una auditoría específica en los procesos de facturación de dicha línea de negocio.

# :chart_with_upwards_trend: 7. Impacto Financiero (Bridge Report)
Tras aplicar un análisis sumatorio al corpus de datos inicial y final se obtuvo que: 
1. Ventas brutas iniciales: $16,922,593.85.
2. Ventas brutas validadas: $16,622,083.46.
3. Riesgo mitigado: $300,510.39 (1.78% de desviación corregida sobre el reporte original).
    
# :bulb: 8. Resultados y Entregables
1. DataSet "Golden Standard": Base de datos curada lista para consumo en Power BI.
2. DataSet “Dirty Data”: Base de datos lista, con los registros rechazados listos para su auditoría por parte de TI.
3. Bridge Report: Comparativa antes y después, recuperando la confianza en el KPI de ventas netas.
4. Reporte de causa raíz: Indicadores del posible origen de las fallas y errores encontrados en el DataSet original.

<img width="885" height="579" alt="Captura de pantalla 2026-01-30 a la(s) 11 44 56 p m" src="https://github.com/user-attachments/assets/8bf5d79b-c54c-49d2-b660-30180b670884" />

<img width="884" height="574" alt="imagen" src="https://github.com/user-attachments/assets/ff1467ff-26ed-4c60-8c09-001f7788ed4a" />

<img width="438" height="355" alt="Captura de pantalla 2026-01-30 a la(s) 11 45 29 p m" src="https://github.com/user-attachments/assets/d4e684e8-b327-4e01-8337-5e8c2cb4e3de" />


