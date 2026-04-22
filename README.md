# 📩 Nudge Apps – Copilot Chat Messaging Script for Microsoft Teams

Este repositorio contiene un script de **PowerShell** diseñado para enviar mensajes a usuarios de **Microsoft Teams** mediante **Adaptive Cards**, utilizando la **API de Microsoft Graph**.

El objetivo es facilitar el envío de prompts preconfigurados para **Copilot Chat**, permitiendo que los usuarios los revisen o editen antes de enviarlos.

***

## 🧭 Descripción General

Este script:

*   Envía mensajes 1:1 en Microsoft Teams a múltiples usuarios.
*   Utiliza **Adaptive Cards** con **seis botones interactivos**.
*   Cada botón contiene un prompt diferente.
*   Al hacer clic:
    *   Se abre Copilot Chat en una nueva pestaña.
    *   El prompt aparece precargado en el cuadro de diálogo.
    *   El usuario puede editarlo antes de enviarlo.

Esto facilita la adopción de Copilot al permitir personalización antes del envío.

***

## 🏗️ Arquitectura de Alto Nivel

El script realiza las siguientes acciones:

1.  Se autentica contra Microsoft Graph.
2.  Lee una lista de usuarios desde Excel.
3.  Carga una Adaptive Card desde un archivo JSON.
4.  Procesa los usuarios en paralelo.
5.  Crea chats 1:1.
6.  Envía la tarjeta como mensaje.
7.  Registra resultados en un archivo CSV.
8.  Cierra la sesión de Graph.

***

## 🔐 Autenticación

El usuario con el cual se autenticará debe ser una cuenta dedicada Ejemplo: <<comunicaciones@contoso.com>> , ya que los mensajes en Teams saldrán de esta cuenta como remitente. 

El script se conecta a Microsoft Graph utilizando autenticación moderna con los siguientes permisos:

*   `Chat.ReadWrite`
*   `User.Read`

Puede requerirse consentimiento administrativo en **Microsoft Entra ID**.

***

## 📥 Datos de Entrada

| Archivo              | Descripción                  |
| -------------------- | ---------------------------- |
| Excel (.xlsx / .xls) | Lista de usuarios destino    |
| JSON                 | Definición del Adaptive Card |

El archivo Excel debe contener una columna llamada:

```text
UPN
```

***
## Configuración de variables en archivo JSON

En la sección Configuration del script, ajusta las siguientes variables:
* RUTA COMPLETA AL ARCHIVO EXCEL DE ENTRADA - LÍNEA 23
<img width="819" height="64" alt="image" src="https://github.com/user-attachments/assets/6a55eb4c-0d4f-4094-be33-6fc2c0072ac0" />

* Personaliza la carpeta que contiene todas las tarjetas (Adaptive Cards) - Línea 55
<img width="940" height="66" alt="image" src="https://github.com/user-attachments/assets/54f32c07-87ec-4dda-9bac-0a824ecc4a09" />

* El script genera automáticamente un nombre de archivo con timestamp. Asegúrate de que el directorio exista o pueda crearse.
  <img width="836" height="76" alt="image" src="https://github.com/user-attachments/assets/c22055ae-9fa9-498a-a9f4-010a89eac9b8" />


## ⚡ Procesamiento en Paralelo

Esto permite:

*   Acelerar el envío de mensajes
*   Reducir operaciones de lectura/escritura
*   Ejecutar múltiples hilos concurrentes

***

## 🔁 Mecanismo de Reintentos

El script implementa:

*   Retry automático
*   Backoff exponencial
*   Jitter aleatorio
*   Manejo del encabezado `Retry-After`
*   
<img width="785" height="48" alt="image" src="https://github.com/user-attachments/assets/2973e116-3e47-4899-96d3-3b4e1d8baa62" />

Esto permite manejar errores transitorios al:

*   Crear chats
*   Enviar mensajes

***

## 📝 Logging Concurrente

Utiliza:

```csharp
[System.Collections.Concurrent.ConcurrentBag]
```

Para:

*   Registrar eventos de éxito
*   Registrar reintentos
*   Registrar errores

Todos los logs se almacenan en memoria y luego se escriben en:

```text
CopilotMessageLog_YYYYMMDD_HHMMSS.csv
```
<img width="836" height="76" alt="image" src="https://github.com/user-attachments/assets/7276a561-4c7b-4372-a25a-8e46f3c8373e" />
***

## ✅ Prerrequisitos

### PowerShell

*   PowerShell 7.0 o superior
Descargalo aquí: https://learn.microsoft.com/es-es/powershell/scripting/install/install-powershell-on-windows?view=powershell-7.5
***

### Módulos Requeridos

Instalar:

```powershell
Install-Module -Name ImportExcel
Install-Module -Name Microsoft.Graph.Teams
Install-Module -Name Microsoft.Graph.Authentication
```


***

### Permisos

La cuenta emisora debe:

*   Tener licencia de Microsoft Teams
*   Poder crear chats
*   Poder enviar mensajes

Permisos requeridos:

```text
Chat.ReadWrite
User.Read
```

***

## 🚀 Despliegue y Configuración

### 1. Guardar Script

Ejemplo:

```text
SendTeamsAdaptiveCard.ps1
```

***

### 2. Ubicación de Archivos

Colocar:

*   Excel de entrada
*   Adaptive Card JSON

En rutas conocidas.

***

### 3. Variables de Configuración

| Variable            | Descripción           |
| ------------------- | --------------------- |
| `$excelFilePath`    | Ruta del Excel        |
| `$adaptiveCardPath` | Carpeta de tarjetas   |
| `$logFile`          | Archivo de log        |
| `$ThrottleLimit`    | Nivel de concurrencia |
| `$retryLimit`       | Número de reintentos  |

Recomendado:

```text
ThrottleLimit = 5–10
```

***

## ▶️ Ejecución del Script

1.  Abrir PowerShell 7 como administrador.
2.  Navegar al directorio:

```powershell
cd <ruta_del_script>
```

3.  Ejecutar:

```powershell
.\Script_Powershell-Envia_AdaptiveCards.ps1
```

4.  Autenticarse cuando se solicite.
<img width="604" height="492" alt="image" src="https://github.com/user-attachments/assets/a3131c22-5076-4fef-8c44-e42ba0e1bd07" />

Durante la primera ejecución:
El script:

*   Procesará UPNs en paralelo
*   Mostrará progreso en consola
*   Solicitará seleccionar la tarjeta por número
*   Cerrará la conexión con Graph al finalizar

<img width="940" height="496" alt="image" src="https://github.com/user-attachments/assets/479bde77-379d-4fd1-8916-abbdee31be4c" />


***

## 📊 Registro de Ejecución

Cada intento registra:

*   Timestamp
*   UPN
*   Estado:
    *   Success
    *   Retry
    *   Error
*   Mensaje descriptivo

***

## 👤 Experiencia del Usuario Final

El usuario recibe una Adaptive Card con botones.

Al hacer clic:

*   Se abre Copilot Chat

<img width="666" height="939" alt="image" src="https://github.com/user-attachments/assets/07b5e3e0-e198-472e-a108-bb6a7bdc211b" />

  
***

## 🛠️ Solución de Problemas

### ForEach-Object -Parallel no reconocido

Verificar versión:

```powershell
$PSVersionTable.PSVersion
```

***

### Execution Policy Restringida

Permitir ejecución temporal:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

***

### Scripts Deshabilitados

Ver:

```powershell
Get-ExecutionPolicy -List
```

Permitir scripts locales:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

***

### Permisos en Graph

Verificar:

*   Chat.ReadWrite
*   User.Read

Otorgar consentimiento administrativo si es necesario.
***

### Módulos No Encontrados

Instalar:

```powershell
Install-Module -Name ImportExcel
Install-Module -Name Microsoft.Graph.Teams
Install-Module -Name Microsoft.Graph.Authentication
```

***

## 🧹 Limpieza

El script desconecta automáticamente:

```powershell
Disconnect-MgGraph
```

***

## ⚠️ Descargo de Responsabilidad

Este script se proporciona únicamente con fines demostrativos.

Es responsabilidad del usuario:

*   Probarlo
*   Ajustarlo
*   Validarlo

Antes de utilizarlo en entornos productivos y asegurar cumplimiento con las políticas organizacionales.

