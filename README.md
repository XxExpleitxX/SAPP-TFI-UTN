# SAPP — Sistema de Automatización de Procesamiento de Pedidos

Material de reproducibilidad del **trabajo final de carrera** «SAPP — Sistema de Automatización de Procesamiento de Pedidos».

**Autores:** Juan Darío Sinatra · Matías Martínez · Emanuel Ortíz
**Carrera:** Ingeniería en Informática
**Institución:** Universidad Tecnológica Nacional — Facultad Regional Mendoza
**Año:** 2026

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22117135.svg)](https://doi.org/10.5281/zenodo.22117135)

---

## Qué es este repositorio

Contiene los artefactos necesarios para reproducir la validación reportada en el Capítulo 5 del trabajo: el flujo de automatización exportado, el corpus de documentos de prueba, el protocolo de validación con sus resultados, la línea de base del proceso manual y las evidencias de ejecución.

El corpus está compuesto por veinte documentos comerciales **generados de forma sintética** mediante un script en Python, conforme a una especificación que fijó diez ítems por documento, rubros diversos y datos de cliente ficticios. No se trata de pedidos de organizaciones reales.

## Contenido

| Carpeta | Contenido |
|---|---|
| `Workflow/` | Flujo de n8n exportado en formato JSON |
| `Corpus/` | Los 20 documentos comerciales del corpus de evaluación |
| `Validacion/` | Protocolo: corpus con valores esperados, registro de ejecuciones, línea de base manual, evaluación independiente y métricas |
| `Linea_de_base/` | Planillas de transcripción manual y grabación de pantalla de uno de los casos |
| `Evidencias/` | Capturas del entorno de ejecución referenciadas en el Anexo C |
| `Diagramas/` | Diagrama de componentes, modelo de datos y diagrama de flujo |

## El sistema

Flujo que recibe pedidos comerciales por correo electrónico, interpreta su contenido mediante un modelo de lenguaje y persiste los datos extraídos en planillas de cálculo.

**Cuatro capas:** ingesta y clasificación · procesamiento inteligente en dos pipelines paralelos (A para PDF adjunto, B para texto en el cuerpo) · persistencia · notificación.

**Entorno:** n8n 2.10.3 autoalojado en Docker · Mistral Cloud (`mistral-medium`) · Gmail, Google Drive y Google Sheets vía OAuth2.

## Cómo reproducir la validación

1. Importar el flujo de `Workflow/` en una instancia de n8n 2.10.3.
2. Configurar las credenciales localmente. **El archivo exportado no contiene secretos**: n8n conserva solo referencias, que deben recrearse en la instancia propia.
3. Ajustar los identificadores de carpeta de Drive en los nodos `Upload file`, `Move file`, `Move file1`, `Search files and folders` y `Search files and folders1`.
4. Consultar la hoja **Corpus y valores esperados** para saber qué documento corresponde a cada caso.
5. Enviar los documentos de `Corpus/` uno por vez, según la hoja **Instrucciones**.
6. Registrar cada corrida en la hoja **Registro de ejecuciones**.

> Los valores esperados deben definirse **antes** de procesar cada documento. Definirlos después de observar la salida invalida la medición.

## Resultados

### Validación del sistema

32 casos computados, ejecutados en cuatro jornadas (12, 14, 17 y 27 de agosto de 2026) con la misma versión del flujo: 14 por el Pipeline A y 18 por el Pipeline B.

| Métrica | Pipeline A | Pipeline B | Total |
|---|---|---|---|
| Casos | 14 | 18 | 32 |
| Sin defecto detectado | 13 | 11 | 24 (75,0 %) |
| Precisión extremo a extremo | 66,7 % | 94,0 % | 77,7 % |
| Precisión sobre campos persistidos | 100,0 % | 94,0 % | 97,2 % |

Tiempo de procesamiento: mediana 20,9 s, rango 14,5 – 36,0 s.

Se reportan dos métricas de precisión porque miden cosas distintas: la primera toma como denominador los campos que el esquema declara extraer; la segunda, solo los que el flujo escribe en la planilla. La diferencia corresponde a un defecto de la capa de persistencia.

**El sistema no derivó por sí mismo ningún caso a revisión humana.** El 75,0 % mide los casos procesados sin defecto detectado por una verificación externa, no la capacidad del sistema para discriminar los casos que requieren intervención. Esa carencia es uno de los hallazgos del trabajo.

### Línea de base manual

Diez documentos transcritos manualmente por un integrante del equipo que no participó de la ronda automatizada ni conocía los documentos. Cada uno tiene su contraparte procesada por el sistema, de modo que la comparación es **pareada sobre documentos idénticos**.

| Métrica | Manual | Sistema |
|---|---|---|
| Tiempo medio | 124,3 s | 20,4 s |
| Mediana | 121,5 s | 20,6 s |
| Coeficiente de variación | 12,9 % | 5,5 % |

Reducción media: **83,4 %** (rango por documento: 81,1 % – 86,2 %). Factor de aceleración: 6,1.

### Evaluación independiente

Ocho casos (25 % de los computados) verificados a ciegas por un segundo evaluador, sin acceso a los valores esperados ni al criterio de cómputo.

Concordancia: 7 de 8 casos · acuerdo a nivel de campo 93,8 % · **kappa de Cohen 0,64** (concordancia considerable).

El único desacuerdo corresponde al tratamiento de la capitalización, punto identificado de antemano como sensible en el análisis de sensibilidad.

### Contraste de hipótesis

**H1 — reducción del tiempo de al menos 50 %: se sostiene.** Los diez pares superan el umbral, sin excepciones.

**H2 — reducción de la tasa de error de al menos 2 puntos: se sostiene con margen estrecho.** Bajo el criterio declarado antes de medir, el proceso manual registró 12 discrepancias sobre 100 artículos y el sistema 8.

La conclusión es sensible al tratamiento de las tildes: excluyéndolas, el manual registraría 6 y el sistema 8, y H2 no se sostendría. Se adopta el criterio declarado a priori por razones de procedimiento; ambos cómputos se reportan. **Las seis discrepancias que separan un cómputo del otro pertenecen todas al proceso manual.**

## Defectos identificados

Cinco defectos, todos en la capa de orquestación y persistencia. **Ninguno fue corregido en la versión publicada**: el flujo se entrega en el estado sobre el cual se realizaron las mediciones, para que los resultados sean verificables.

| # | Defecto | Componente |
|---|---|---|
| 1 | Emite confirmación ante correos que no son pedidos | `Code in JavaScript1` |
| 2 | El campo de unidad se extrae y no se persiste | `Append row in sheet1` |
| 3 | Extracción no reproducible entre ejecuciones | nodos de modelo, sin temperatura fijada |
| 4 | La escritura no identifica de forma unívoca la planilla de destino | `Search files and folders1` |
| 5 | Sin correspondencia verificable entre lo procesado y lo comunicado | transversal |

El quinto agrupa siete casos con el mismo comportamiento: el sistema produce un resultado incorrecto, incompleto o duplicado y prosigue sin señal de anomalía, sea confirmando de más o —en el reenvío— omitiendo la confirmación debida.

En sentido favorable, el modelo **no fabrica los campos ausentes** ni inventa cantidades cuando el documento no las especifica.

## Limitaciones

- Ambos evaluadores integran el equipo de investigación; el 75 % de los casos cuenta con una única verificación.
- La línea de base se cronometró con un único operador, integrante del equipo y no un administrativo con experiencia en carga de pedidos.
- El corpus es sintético y de estructura homogénea: los documentos fueron generados por un modelo de lenguaje y procesados por un modelo de lenguaje, lo que podría favorecer el desempeño del sistema.
- No se aplicaron pruebas de inferencia estadística: el corpus no es una muestra aleatoria de una población de pedidos comerciales.
- Bajo la configuración de credenciales evaluada (proyecto en modo de prueba), el token expira a los siete días y exige reautorización manual.

## Nota sobre los datos

Los documentos del corpus fueron generados con fines de evaluación. Nombres de clientes, domicilios y datos comerciales son ficticios y no corresponden a personas ni organizaciones reales.

## Cómo citar

    Sinatra, J. D., Martínez, M. y Ortíz, E. (2026). SAPP — Sistema de
    Automatización de Procesamiento de Pedidos [Trabajo final de carrera].
    Universidad Tecnológica Nacional, Facultad Regional Mendoza.
    https://doi.org/10.5281/zenodo.22117135

## Licencia

- **Código y flujo de automatización:** MIT
- **Corpus y documentación:** CC BY 4.0
