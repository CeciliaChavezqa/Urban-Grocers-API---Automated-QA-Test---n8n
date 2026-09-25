# Urban-Grocers-API-Automated-QA-Test-n8n
El proyecto evoluciona desde pruebas manuales hacia una arquitectura de prueba totalmente automatizada con propagación dinámica de autenticación, análisis de valores límite, respuestas reales de la API y estatus/resultados esperados vs resultados actuales para la generación de reportes en tiempo real.

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

## 📋 Detalle del Flujo de Trabajo

**1. Autenticación y Creación de Usuario (POST /api/v1/users)**

Para iniciar cualquier sesión de prueba en Urban Grocers, se ejecuta primero la creación de una cuenta para obtener acceso autorizado.

🧪 Endpoint: POST {{baseUrl}}/api/v1/users

🧪Headers: Content-Type: application/json

**Parámetros:**

🧪firstName (string, requerido): Nombre del usuario.

🧪phone (string, requerido): Número de teléfono.

🧪address (string, requerido): Dirección de entrega.

🧪email (string, opcional): Correo electrónico.

🧪comment (string, opcional): Comentarios adicionales.

Respuesta Exitosa del Servidor (201 Created), n8n captura automáticamente el valor de authToken de la respuesta 201 y lo almacena en memoria. En las ejecuciones posteriores, el token se inyecta dinámicamente en el header de autorización.

**2. Creación de Kit y Tolerancia a Fallos (POST /api/v1/kits)**

El endpoint para crear un kit requiere establecer el contexto asociándolo a un usuario específico o a una tarjeta. Para este paso, utilizamos el contexto del usuario generado en el paso anterior.

**Reglas de Negocio (Autorización):** 

🧪Es obligatorio enviar el encabezado Authorization o el parámetro cardId para poder crear un kit.   

🧪Si la solicitud carece de ambos parámetros, el sistema devolverá un error.   

🧪Si se proporciona un encabezado Authorization con un authToken válido, el kit se asignará automáticamente a ese usuario en particular.   

🧪En caso de enviar tanto el Authorization como el cardId, el encabezado de autorización tiene prioridad absoluta.   

**Headers:**

Authorization: Definido en formato Bearer {authToken}.  

Content-Type: Establecido por defecto como application/json.   

Parámetros del Payload:name (string): El nombre asignado al kit, el cual se escribe directamente en el campo correspondiente de la tabla kit_model.   

cardId (number, opcional): El ID correspondiente a la tabla card_model (omitido en nuestra suite para priorizar la prueba por autorización de usuario). 

El nodo está configurado de manera explícita con Continue Regular Routing. Esto garantiza que cuando la matriz inyecte datos inválidos (como el escenario de nombre vacío documentado en el Bug Report) y el servidor devuelva errores HTTP (400+), el flujo no se interrumpa, permitiendo que el reporte final registre la discrepancia.

## 📋 Estructura del Flujo en n8n

```text
[Start] ➔ [Edit Fields (baseUrl)] ➔ [POST /api/v1/users] ➔ [Code (Test Matrix)] ➔ [POST /api/v1/kits] ➔ [Code (Test Reporter)]


