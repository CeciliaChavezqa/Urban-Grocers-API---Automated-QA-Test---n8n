# Urban-Grocers-API-Automated-QA-Test-n8n
El proyecto evoluciona desde pruebas manuales hacia una arquitectura de prueba totalmente automatizada con propagación dinámica de autenticación, análisis de valores límite, respuestas reales de la API y estatus/resultados esperados vs resultados actuales para la generación de reportes en tiempo real.

Este repositorio alberga el marco de automatización **End-to-End y Data-Driven (DDT)** diseñado para validar la API REST de **Urban Grocers**. 

El proyecto resuelve la ineficiencia y el margen de error de las validaciones manuales mediante la orquestación de un flujo resiliente en **n8n**, capaz de gestionar estados de autenticación dinámicos, inyectar matrices de prueba de valores límite y auditar respuestas HTTP en tiempo real.

**Estrategias de QA Aplicadas:**

💡Nombres válidos cortos, caracteres especiales y cadenas vacías (0 caracteres).

💡Propagación dinámica de cabeceras de autorización entre nodos sin dependencias externas.

💡Configuración de ruteo tolerante a fallos (400+ Bad Request) para evitar el colapso del workflow durante la ejecución de escenarios límite.

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

**⬜1. Autenticación y Creación de Usuario (POST /api/v1/users)**

Para iniciar cualquier sesión de prueba en Urban Grocers, se ejecuta primero la creación de una cuenta para obtener acceso autorizado.

🧪 Endpoint: POST {{baseUrl}}/api/v1/users

🧪Headers: Content-Type: application/json

**Parámetros:**

🧪firstName (string, requerido): Nombre del usuario.

🧪phone (string, requerido): Número de teléfono.

🧪address (string, requerido): Dirección de entrega.

🧪email (string, opcional): Correo electrónico.

🧪comment (string, opcional): Comentarios adicionales.

**Manejo de Contexto:** Respuesta exitosa `201 Created`. n8n captura automáticamente el valor de `authToken` de la respuesta y lo almacena en memoria. En las ejecuciones posteriores, el token se inyecta dinámicamente en el header de autorización (`Authorization: Bearer {{ $json.authToken }}`).

**⬜2. Creación de Kit y Tolerancia a Fallos (POST /api/v1/kits)**

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

**⚪Matriz de Pruebas Data-Driven (`Code Node: Test Matrix`)**
Genera la lista de escenarios límites para validar la creación de Kits:

Compara el código de estado real recibido contra el esperado y genera el reporte consolidado:

```javascript
return [
  { json: { testName: "Nombre válido corto (1 carácter)", kitName: "A", expectedStatus: 201 } },
  { json: { testName: "Nombre vacío (0 caracteres)", kitName: "", expectedStatus: 400 } },
  { json: { testName: "Caracteres especiales", kitName: "Kit_#$&%", expectedStatus: 201 } }
];
```
**⬜3. Motor de Aserciones (Code Node: Test Reporter)**

Compara el código de estado real recibido contra el esperado y genera el reporte consolidado:

const results = [];
const testMatrix = $('Code in JavaScript').all(); 

for (let i = 0; i < items.length; i++) {
    const actualStatus = items[i].json.statusCode;
    const expectedStatus = testMatrix[i].json.expectedStatus;
    const testName = testMatrix[i].json.testName;
    
    let testResult = (actualStatus === expectedStatus) ? "PASS ✅" : "FAIL ❌";

    results.push({
        json: {
            "Test Name": testName,
            "Expected": expectedStatus,
            "Actual": actualStatus,
            "Status": testResult,
            "Body": items[i].json.body
        }
    });
}
return results;

## 🔄 Estructura del Flujo en n8n

[Start] ➔ [Edit Fields (baseUrl)] ➔ [POST /api/v1/users] ➔ [Code (Test Matrix)] ➔ [POST /api/v1/kits] ➔ [Code (Test Reporter)]

## 📦Cómo Importar y Ejecutar este Proyecto.

**1. Requisitos previos: Tener Node.js e iniciar n8n localmente:**

- npx n8n

**2.Importar en n8n:**

- Abre la interfaz web de n8n (http://localhost:5678).

- Haz clic en los tres puntos (...) en la esquina superior izquierda del canvas.

- Selecciona Import from File y carga el archivo urban-grocers-e2e-suite.json.

**3.Configurar la URL Base:**

- Abre el nodo Edit Fields y actualiza el valor de baseUrl con la URL activa de tu servidor de Urban Grocers.

**4.Ejecutar:**

- Haz clic en el botón Execute Workflow en la parte inferior.

## 📸 Evidencia de Ejecución

**Workflow Ejecutado.**

<img width="1915" height="873" alt="Captura de pantalla 2026-10-05 141249" src="https://github.com/user-attachments/assets/b6f63bac-9e39-4327-9ed9-b8d7887b0635" />



**Tabla de Resultados del Reporter**



<img width="1908" height="873" alt="image" src="https://github.com/user-attachments/assets/c9085ce2-2d0b-40a4-9cf3-a3602e4cee5b" />




## 🎬 Demo de Ejecución en Tiempo Real. 

https://github.com/user-attachments/assets/49897cee-87ce-46a7-bfa0-7fc3f9889ac7









