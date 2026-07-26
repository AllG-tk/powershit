# ...para empezar abre tu terminal Powershell
comando para powershell

# Playbook PowerShell - Investigación de Procesos

> **Objetivo:** Investigar un proceso que presenta un consumo elevado de CPU o memoria utilizando únicamente herramientas nativas de PowerShell.

---

# Fase 1 - Identificar el proceso

## Ver los procesos que más CPU consumen

```powershell
Get-Process |
Sort-Object CPU -Descending |
Select-Object -First 15 Name, Id, CPU
```

## Ver los procesos que más memoria consumen

```powershell
Get-Process |
Sort-Object WorkingSet64 -Descending |
Select-Object -First 15 Name, Id,
@{N="RAM_MB";E={[math]::Round($_.WorkingSet64/1MB,2)}}
```

### Preguntas que debes hacerte

- ¿Cuál consume más recursos?
- ¿Conozco este proceso?
- ¿Ese consumo es normal para su función?
- ¿El proceso apareció recientemente?

---

# Fase 2 - Obtener información detallada

Supongamos que el proceso sospechoso es **chrome.exe**.

```powershell
Get-Process chrome | Format-List *
```

Información relevante:

- PID
- Ruta del ejecutable
- Tiempo de CPU
- Memoria
- Handles
- Threads
- Hora de inicio

---

# Fase 3 - Localizar el ejecutable

```powershell
(Get-Process chrome).Path
```

Resultado esperado

```
C:\Program Files\Google\Chrome\Application\chrome.exe
```

## Analizar

✔ Carpeta oficial

```
C:\Program Files\
```

✔ Carpeta del sistema

```
C:\Windows\
```

⚠ Revisar si aparece en:

```
AppData
Temp
Downloads
Desktop
```

---

# Fase 4 - Verificar la firma digital

```powershell
Get-AuthenticodeSignature (Get-Process chrome).Path
```

Campos importantes

- Status
- SignerCertificate

Resultado esperado

```
Status : Valid
```

---

# Fase 5 - Obtener el Hash SHA256

```powershell
Get-FileHash (Get-Process chrome).Path
```

Ejemplo

```
Algorithm : SHA256

Hash :

A64F89213C...
```

Este hash permite:

- Comparar versiones
- Documentar evidencia
- Detectar modificaciones

---

# Fase 6 - Revisar conexiones de red

Supongamos que el PID es **4512**.

```powershell
Get-NetTCPConnection |
Where-Object OwningProcess -eq 4512
```

Información obtenida

- Dirección local
- Puerto local
- Dirección remota
- Puerto remoto
- Estado

---

# Fase 7 - Revisar el destino DNS

```powershell
Resolve-DnsName google.com
```

Para obtener información DNS.

---

# Fase 8 - Hora de inicio

```powershell
Get-Process chrome |
Select Name, StartTime
```

Analizar

- ¿Comenzó después del último reinicio?
- ¿Comenzó justo antes del problema?

---

# Fase 9 - Servicios relacionados

Listar servicios

```powershell
Get-Service
```

Buscar el servicio asociado al PID

```powershell
Get-CimInstance Win32_Service |
Where-Object {$_.ProcessId -eq 4512}
```

---

# Fase 10 - Tareas Programadas

```powershell
Get-ScheduledTask
```

Mostrar únicamente las listas para ejecutarse

```powershell
Get-ScheduledTask |
Where-Object State -eq Ready
```

Buscar tareas desconocidas.

---

# Fase 11 - Programas de inicio

```powershell
Get-CimInstance Win32_StartupCommand
```

Analizar

- Nombre
- Ruta
- Usuario

---

# Fase 12 - Revisar eventos

Eventos recientes del sistema

```powershell
Get-WinEvent -LogName System -MaxEvents 50
```

Eventos de aplicaciones

```powershell
Get-WinEvent -LogName Application -MaxEvents 50
```

Errores únicamente

```powershell
Get-WinEvent |
Where-Object LevelDisplayName -eq "Error"
```

---

# Fase 13 - Rendimiento en tiempo real

CPU

```powershell
Get-Counter '\Processor(_Total)\% Processor Time'
```

RAM disponible

```powershell
Get-Counter '\Memory\Available MBytes'
```

Disco

```powershell
Get-Counter '\PhysicalDisk(_Total)\Disk Transfers/sec'
```

Red

```powershell
Get-Counter '\Network Interface(*)\Bytes Total/sec'
```

---

# Fase 14 - Evaluación

## Verificar

- ¿La ruta es legítima?
- ¿Tiene firma digital?
- ¿Consume demasiados recursos?
- ¿Tiene conexiones activas?
- ¿Se inicia automáticamente?
- ¿Está asociado a un servicio?
- ¿Genera errores?
- ¿Su comportamiento coincide con su función?

---

# Fase 15 - Documentar

Guardar procesos

```powershell
Get-Process |
Export-Csv procesos.csv -NoTypeInformation
```

Guardar conexiones

```powershell
Get-NetTCPConnection |
Export-Csv conexiones.csv -NoTypeInformation
```

Guardar servicios

```powershell
Get-Service |
Export-Csv servicios.csv -NoTypeInformation
```

Guardar eventos

```powershell
Get-WinEvent -LogName System -MaxEvents 200 |
Export-Csv eventos.csv -NoTypeInformation
```

---

# Flujo de Investigación

```text
Proceso con alto consumo
          │
          ▼
Identificar CPU y RAM
          │
          ▼
Obtener información completa
          │
          ▼
Verificar ruta del ejecutable
          │
          ▼
Comprobar firma digital
          │
          ▼
Calcular Hash SHA256
          │
          ▼
Revisar conexiones TCP
          │
          ▼
Analizar servicios
          │
          ▼
Revisar tareas programadas
          │
          ▼
Comprobar programas de inicio
          │
          ▼
Consultar eventos del sistema
          │
          ▼
Documentar hallazgos
          │
          ▼
Conclusión
```

---

# Checklist Final

| Verificación | Estado |
|--------------|--------|
| CPU elevada | ☐ |
| RAM elevada | ☐ |
| Ruta legítima | ☐ |
| Firma válida | ☐ |
| Hash obtenido | ☐ |
| Conexiones revisadas | ☐ |
| Servicios revisados | ☐ |
| Inicio automático revisado | ☐ |
| Eventos analizados | ☐ |
| Evidencia exportada | ☐ |

---

# Comandos más utilizados

| Objetivo | Comando |
|----------|---------|
| Procesos | `Get-Process` |
| Servicios | `Get-Service` |
| Eventos | `Get-WinEvent` |
| Conexiones | `Get-NetTCPConnection` |
| Usuarios | `Get-LocalUser` |
| Programas de inicio | `Get-CimInstance Win32_StartupCommand` |
| Tareas programadas | `Get-ScheduledTask` |
| Hash | `Get-FileHash` |
| Firma digital | `Get-AuthenticodeSignature` |
| Rendimiento | `Get-Counter` |
| Información del sistema | `Get-ComputerInfo` |
| Exportar evidencia | `Export-Csv` |
