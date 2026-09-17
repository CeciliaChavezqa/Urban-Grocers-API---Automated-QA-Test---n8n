# Urban-Grocers-API-Automated-QA-Test-n8n
El proyecto evoluciona desde pruebas manuales hacia una arquitectura de prueba totalmente automatizada con propagación dinámica de autenticación, análisis de valores límite, respuestas reales de la API y estatus/resukltados esperados vs resultados actuales para la generación de reportes en tiempo real.

🚀 **Características Principales**

* **Arquitectura E2E Completa:** Creación dinámica de usuarios, captura de `authToken` (JWT) y reutilización de contexto entre peticiones.
* **Data-Driven Testing (DDT):** Inyección de matrices de prueba en JavaScript para evaluación de valores límite (*Boundary Value Analysis*).
* **Manejo Controlado de Errores:** Configuración de nodos con `Continue Regular Routing` para procesar escenarios negativos (errores HTTP 400+) sin detener la suite.
* **Motor de Aserciones:** Comparación automatizada entre el código de estado esperado (`expectedStatus`) y el recibido (`actualStatus`).
* **Reporte de Ejecución:** Salida tabular consolidada con resultados `PASS ✅` / `FAIL ❌`.

## 🛠️ Tecnologías Utilizadas

* **Plataforma de Automatización:** [n8n](https://n8n.io/) (Instancia local)
* **Lenguaje:** JavaScript (Nodos `Code` para matriz de datos y aserciones)
* **Protocolo:** HTTP / REST API (JSON)

## 📋 Estructura del Flujo en n8n

```text
[Start] ➔ [Edit Fields (baseUrl)] ➔ [POST /api/v1/users] ➔ [Code (Test Matrix)] ➔ [POST /api/v1/kits] ➔ [Code (Test Reporter)]

📋 **Detalle del Flujo de Trabajo**
