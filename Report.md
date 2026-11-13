
```bash
                         ATTACK
                            │
                           Kali
                            │
                         INTERNET
                            │
                            │
                       [ FortiGate ]
                            │
              ┌─────────────┼─────────────┐
              │             │             │
          SERVERS         USERS          SOC
              │             │             │
       ┌──────┴──────┐      │             │
       │             │      │             │
   AD/DNS          Wazuh   Windows 11    Windows 11
 Windows Server    Ubuntu 64             SOC Analyst
```

```
VMnet1 → 192.168.219.0/24

DC01 / AD + DNS  → 192.168.219.10
Ubuntu / Wazuh   → 192.168.219.20
Windows 7        → 192.168.219.30
Windows 11 SOC   → 192.168.219.40
```

VMnet1 192.168.219.0
- Windows Server 2012
- Ubuntu Server
- Windows 7 x64
- Windows 11 x64
VMnet2 172.16.1.0
- Kali Linux
VMnet1 y VMnet8
-  FortiOS



# Windows Server 2012

## Configuración de red 

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816142307.png)

**Cambio de nombre de equipo**

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816142920.png)

### Comprobación de Configuración de Red

Windows Server está listo para instalar AD DS y convertirlo en Domain Controller.

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816143332.png)

## Instalación de AD en Windows Sever 2012

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816143821.png)

**Promover este servidor a controlador de dominio**

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816145250.png)

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816145455.png)

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816150107.png)

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816151011.png)

`Credenciales: Cibhack123`

```
Dominio DNS:       soclab.local
NetBIOS:           SOCLAB
Domain Controller: DC01
IP:                192.168.219.10
```
### Comprobación de instalación de AD en Windows Sever 2012

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260816152503.png)

```
                 SOCLAB.LOCAL
                      │
                   DC01
              192.168.219.10
                      │
             ┌────────┴────────┐
             │                 │
      Active Directory        DNS
```

**AD (Active Directory)** administra usuarios, equipos, grupos y políticas del dominio.

**DNS (Domain Name System)** permite que los equipos encuentren servicios del dominio, especialmente el Domain Controller.

Después de comprobar esto, el **siguiente paso será preparar Ubuntu Server para Wazuh**.

# Ubuntu Server

![](Images/WAZUH01-Server/Pasted%20image%2020260816154646.png)

Habilitamos una seguida interfaz para poder mientras obtener conexión a internet y hacer la instalación correcta de Wazuh posteriormente.
## Configuracion de red Ubuntu Server

`sudo nano /etc/netplan/50-cloud-init.yaml`

```bash
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 192.168.219.20/24
    ens37:
      dhcp4: true
```

![](Images/WAZUH01-Server/Pasted%20image%2020260816155933.png)

### Comprobación de Configuración de red Ubuntu Server

![](Images/WAZUH01-Server/Pasted%20image%2020260816160238.png)

De esta forma validamos que aun mantenemos la salida a internet que necesitamos por ahora en este equipo y asignamos la ip que deseamos al adaptador de red que utilizaremos al tener montado el AD.

![](Images/WAZUH01-Server/Pasted%20image%2020260816160835.png)

Validamos que tenemos conexión con el Windows Server y salida temporal a internet en la interfaz ens37.

```
VMnet1 — LAN del laboratorio
        │
        ├── DC01 / AD + DNS
        │     192.168.219.10
        │
        └── Ubuntu / Wazuh
              192.168.219.20

VMnet8 — NAT / Internet
        │
        └── Ubuntu ens37
```

## Instalación de Wazuh en Ubuntu Server

Actualización de sistema y paquetes.

`sudo apt update & sudo apt upgrade -y`

![](Images/WAZUH01-Server/Pasted%20image%2020260816161321.png)

`curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh`

`sudo bash ./wazuh-install.sh -a`

![](Images/WAZUH01-Server/Pasted%20image%2020260816183213.png)

![](Images/WAZUH01-Server/Pasted%20image%2020260816183411.png)

```
User: admin
Password: AF2JV0GfhLQJQcYZmzT+DtqYzV2V2Ssg
```

```
User: admin
Password: AF2JV0GfhLQJQcYZmzT+DtqYzV2V2Ssg
```

```
                    SOC LAB
                       │
              ┌────────┴────────┐
              │                 │
          AD / DNS           Wazuh
           DC01              Ubuntu
      192.168.219.10     192.168.219.20
              │                 │
              └────────┬────────┘
                       │
                 VMnet1 / LAN
```

### Prueba de acceso a Wazuh

Como tenemos salida en el servidor Ubuntu a nuestra red de internet intentaremos acceder directamente del pc personal para validar el acceso antes de integrar los agentes en los clientes.

`https://192.168.219.20`

![](Images/WAZUH01-Server/Pasted%20image%2020260816184119.png)

![](Images/WAZUH01-Server/Pasted%20image%2020260816184244.png)

```
Windows 11 SOC
      │
      ├── ping 192.168.219.10 → DC01
      │
      └── https://192.168.219.20 → Wazuh
```

## Cambio de nombre Ubuntu Sever

`sudo sed -i 's/127.0.1.1 test1/127.0.1.1 WAZUH01/' /etc/hosts`

`cat /etc/hosts`

![](Images/WAZUH01-Server/Pasted%20image%2020260822124306.png)

Reiniciamos para aplicar los cambios

### Comprobacion de cambio de nombre Ubuntu Server

![](Images/WAZUH-SIEM/Pasted%20image%2020260822125119.png)

## Política de firewall en WAZUH01 permitida del puerto 514  tcp/udp para registros desde FW01

```
sudo ufw allow 514/tcp 
sudo ufw allow 514/udp 
sudo ufw status
```

![](Images/WAZUH01-Server/Pasted%20image%2020260913155302.png)

`sudo systemctl restart wazuh-manager`
`sudo ss -tulnp | grep 514`

![](Images/WAZUH01-Server/Pasted%20image%2020260913160010.png)

### Comprobacion de conexion establecida en envio de logs FW01 a WAZUH01

Generamos ping hacia  la ip `192.168.1.185` que corresponde a la interfaz WAN que posee FW01

![](Images/WAZUH01-Server/Pasted%20image%2020260913160329.png)
# Windows 11 SOC

## Configuración de red Windows 11 SOC

`ncpa.cpl`

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817125558.png)

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817124310.png)

### Prueba de conectividad con Windows Server (DC), Ubuntu Server (Wazuh) y DNS

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817130003.png)

```
Windows 11 SOC
       │
       └── 192.168.219.40
              │
            VMnet1
              │
       ┌──────┴──────┐
       │             │
    DC01 .10      Wazuh .20
```

## Inclusión de Windows 11 al dominio

`sysdm.cpl`

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817130535.png)

Nos pedira las credenciales, en este caso utilizaremos las del administrador del DC configurado.

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817130837.png)

## Comprobación de Inclusión de Windows 11 al dominio

Reiniciamos para que se apliquen los cambios y validamos

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817131533.png)

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817131756.png)

## Instalación de agente Wazuh en cliente Windows 11 SOC

`https://192.168.219.20/`

![](Images/WAZUH-SIEM/Pasted%20image%2020260817140425.png)

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.7-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='192.168.219.20' WAZUH_AGENT_NAME='defense'
```

`NET START Wazuh`

`Get-Service WazuhSvc` = Comprobación de que se encuentre corriendo
`Get-CimInstance Win32_Service -Filter "Name='WazuhSvc'" | Select Name, State, StartMode` = Comprobación que inicie automáticamente con el sistema.

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817144735.png)

```
Windows 11
192.168.219.40
      │
      │ Wazuh Agent
      ▼
Wazuh Manager
192.168.219.20
      │
      ▼
Wazuh Dashboard
```

### Validación de instalación de agente Wazuh en Windows 11 SOC

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817165230.png)

El **Manager/servidor Wazuh puede recolectar determinados logs locales**, mientras que el agente es lo que usamos para monitorizar endpoints.

Validamos que evistan logs ya registrandose

![](Images/SOC01-Monitoreo/Pasted%20image%2020260817165817.png)

```
                 WAZUH SERVER
                  test1
             192.168.219.20
                    │
       ┌────────────┴────────────┐
       │                         │
   Manager                  Indexer
       │                         │
       └────────────┬────────────┘
                    │
                Dashboard
                    │
        ┌───────────┴───────────┐
        │                       │
   test1 / Ubuntu          defense / Windows
   eventos locales          Wazuh Agent
```

## Cambio de nombre de Windows 11 SOC

`Rename-Computer -NewName "SOC01" -DomainCredential (Get-Credential) -Restart`

![](Images/SOC01-Monitoreo/Pasted%20image%2020260822121209.png)

`hostname`

`(Get-CimInstance Win32_ComputerSystem) | Select-Object Name,Domain,PartOfDomain`

![](Images/SOC01-Monitoreo/Pasted%20image%2020260822122224.png)


**Pasamos a ejecutar esto en el Ubuntu Server**

`sudo sed -i '/^001[[:space:]]/s/[[:space:]]defense[[:space:]]/ SOC01 /' /var/ossec/etc/client.keys`

`sudo grep '^001[[:space:]]' /var/ossec/etc/client.keys`

![](Images/WAZUH01-Server/Pasted%20image%2020260822132628.png)

**Pasamos a ejecutar en Windows 11 SOC**

`(Get-Content "C:\Program Files (x86)\ossec-agent\client.keys") -replace '^001\s+defense\s+', '001 SOC01 ' | Set-Content "C:\Program Files (x86)\ossec-agent\client.keys" -Encoding ASCII`

`Get-Content "C:\Program Files (x86)\ossec-agent\client.keys"`

Arrancamos nuevamente el agente

`Start-Service -Name wazuh`

`Get-Service -Name wazuh`

![](Images/SOC01-Monitoreo/Pasted%20image%2020260822133511.png)

### Comprobación cambio de nombre Windows 11 SOC

![](Images/WAZUH-SIEM/Pasted%20image%2020260822134923.png)
# Cliente Windows 11
## Configuración de red Cliente Windows 11

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260817184626.png)

### Comprobación de red Cliente Windows 11

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260817184846.png)

## Inclusión de Cliente Windows 11 en AD

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260817185839.png)

### Comprovacion de Inclusión de Cliente Windows 11 en AD

`(Get-CimInstance Win32_ComputerSystem).Domain`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260817191037.png)

## Instalación de agente Wazuh en Cliente Windows 11

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.7-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='192.168.219.20' WAZUH_AGENT_NAME='defense'
```

`NET START Wazuh`

`Get-Service WazuhSvc`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260817194317.png)

![](Images/WAZUH-SIEM/Pasted%20image%2020260817195529.png)

![](Images/WAZUH-SIEM/Pasted%20image%2020260817195758.png)

## Instalación de Sysmon en Cliente Windows 11

`Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "$env:TEMP\Sysmon.zip"`

`Expand-Archive "$env:TEMP\Sysmon.zip" -DestinationPath "C:\Sysmon" -Force`

`.\Sysmon64.exe -accepteula -i`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818133704.png)

### Comprobación de instalación Sysmon en cliente Windows 11

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818134139.png)

### Agregando Sysmon a la telemetria para Wazuh

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818140246.png)

```
<localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
```

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818135457.png)

`Get-Content "C:\Program Files (x86)\ossec-agent\ossec.conf" | Select-String "Sysmon"`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818140222.png)

`[xml](Get-Content "C:\Program Files (x86)\ossec-agent\ossec.conf")`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818140641.png)

`Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |Select-Object TimeCreated, Id, ProviderName, Message`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818140907.png)

`Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50` = Nos permitira validar si el agente wazuh en el cliente windows 11 esta enviando la telemetria adecuadamente.

### Activando telemetria sysmon en WAZUH01

`sudo nano /var/ossec/etc/ossec.conf`

Modificamos

```bash
<logall>no</logall>
<logall_json>no</logall_json>
```
por
```bash
<logall>yes</logall>
<logall_json>yes</logall_json>
```

![](Images/WAZUH-SIEM/Pasted%20image%2020260818151109.png)

Reiniciamos para aplicar y probar los cambios

`sudo systemctl restart wazuh-manager`

### Comprobación de activación de telemetría sysmon en windows 11

Ejecutamos en CLIENT01 para realizar la prueba de telemetria
`Start-Process notepad.exe`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818153617.png)

Ejecutamos en WAZUH01
`sudo grep -i "notepad.exe" /var/ossec/logs/archives/archives.json | tail`
![](Images/WAZUH01-Server/Pasted%20image%2020260818153521.png)


La documentación oficial confirma que para visualizar los archivos archivados en el Dashboard hay que habilitar `archives` en Filebeat y después utilizar el índice `wazuh-archives-*`

`sudo grep -A5 -B2 "archives:" /etc/filebeat/filebeat.yml`

![](Images/WAZUH01-Server/Pasted%20image%2020260818154039.png)

`sudo nano /etc/filebeat/filebeat.yml`

![](Images/WAZUH01-Server/Pasted%20image%2020260818154224.png)

Reiniciamos para aplicar cambios
`sudo systemctl restart filebeat`
`sudo systemctl status filebeat --no-pager`

![](Images/WAZUH01-Server/Pasted%20image%2020260818154436.png)

### Comprobación de telemetría recepcionada en Wazuh de Sysmon en Cliente Windows 11

Creamos un nuevo index llamado `wazuh-archives-*` 
Ya vemos telemtria del servidor ubuntu y CLIENT01

![](Images/WAZUH01-Server/Pasted%20image%2020260818155043.png)

![](Images/WAZUH01-Server/Pasted%20image%2020260818155413.png)

## Habilitación PowerShell Logging en Cliente Windows 11

Esto captura los scripts completos ejecutados en un sistema, incluyendo comandos ofuscados o decodificados en tiempo de ejecución, lo cual es crucial para detectar actividades maliciosas como la ejecución de código, la escalada de privilegios o los intentos de persistencia por parte de atacantes

PowerShell Logging es el conjunto de mecanismos de registro (logging) que Microsoft proporciona en Windows PowerShell para capturar y auditar la actividad de scripts, comandos y sesiones de PowerShell.
1.Module Logging 
2.Script Block Logging
3.Transcription

`$PSVersionTable.PSVersion`

Revision de version actual powershell instalada

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818155922.png)

`Get-WinEvent -ListLog "*PowerShell*" | Select-Object LogName, IsEnabled`

Revision de logs disponibles actualmente

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818160029.png)

### Habilitar Script Block Logging en Cliente Windows 11

`New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force`
`New-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name "EnableScriptBlockLogging" -Value 1 -PropertyType DWORD -Force`

**Comprobacion que quedo habilitado**
`Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"`

**Resultado**
`EnableScriptBlockLogging : 1`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818160705.png)

### Comprobacion de Script Block Loggin evento 4104 en Cliente Windows 11

`Write-Output "SOC LAB - PowerShell Test"`

`Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 10 | Select-Object TimeCreated, Id, Message`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818161111.png)

### Inclusion de telemetria Powershell a Wazuh en Cliente Windows 11

`notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"`

```
<localfile>
    <location>Microsoft-Windows-PowerShell/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
```

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818161502.png)

Reiniciamos el servicio para que se apliquen los cambios realizados.
`Restart-Service Wazuh`
`Get-Service Wazuh`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818161953.png)

### Comprobacion de visualizacion de eventos powershell en wazuh de Cliente Windows 11

Generamos otro evento 4104 de prueba
`Write-Output "SOC LAB - Wazuh PowerShell Test2"`
![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818162111.png)

![](Images/WAZUH-SIEM/Pasted%20image%2020260818172301.png)

Prepare un bloque de script rapido y sencillo y vemos los logs

```
Write-Output "SOC-TEST-START"

Get-Process

Get-Service

whoami

Write-Output "SOC-TEST-END"
```

`powershell.exe -File C:\Temp\SOC-Test.ps1`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260818180808.png)

![](Images/WAZUH-SIEM/Pasted%20image%2020260818180651.png)

### Habilitacion de Module Logging en Cliente Windows 11

**Module Logging / Registro de módulos** registra actividad de comandos y cmdlets ejecutados mediante módulos de PowerShell.

`New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging" -Force`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820151702.png)

`New-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging" -Name "EnableModuleLogging" -PropertyType DWORD -Value 1 -Force`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820151749.png)

Ahora creamos la lista de módulos:

`New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging\ModuleNames" -Force`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820152000.png)

Y configuramos `*` para registrar **todos los módulos**:

`New-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging\ModuleNames" -Name "*" -PropertyType String -Value "*" -Force`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820152114.png)

### Verificacion de Module Logging en Cliente Windows 11

`Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging"`

`Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging\ModuleNames"`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820152359.png)

### Generando actividad para validar module logging activo en Cliente Windows 11

`Get-Service`

`Get-Process`

`Get-ChildItem C:\Windows`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820152816.png)

Ahora comprobamos si Windows generó el evento **4103**:

`Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 30 |Select-Object TimeCreated, Id, Message`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820153421.png)

![](Images/WAZUH-SIEM/Pasted%20image%2020260820154041.png)

```
                 CLIENT01
                    │
             PowerShell
                    │
        ┌───────────┴───────────┐
        │                       │
   Script Block             Module Logging
      4104                       4103
        │                         │
        └───────────┬─────────────┘
                    │
               Wazuh Agent
                    │
                    ▼
             Wazuh Archives
```

### Habilitacion de PowerShell Transcription en Cliente Windows 11

Crear carpeta para las transcripciones

`New-Item -Path "C:\ProgramData\PowerShellTranscripts" -ItemType Directory -Force`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820160116.png)

`icacls "C:\ProgramData\PowerShellTranscripts" /grant "*S-1-5-32-545:(OI)(CI)M"`

`icacls "C:\ProgramData\PowerShellTranscripts"`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820160141.png)

`New-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription" -Name EnableTranscripting -PropertyType DWord -Value 1 -Force`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820160915.png)

`New-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription" -Name EnableInvocationHeader -PropertyType DWord -Value 1 -Force`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820161012.png)

**Verificamos**

`Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription"`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820161205.png)

Ahora vamos a decir donde guardar las transcripciones, sera donde creamos la carpeta ya creada en `C:\ProgramData\PowerShellTranscripts`

`New-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription" -Name OutputDirectory -PropertyType String -Value "C:\ProgramData\PowerShellTranscripts" -Force`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820161838.png)

### Verificacion de PowerShell Transcription habilitado en Cliente Windows 11

`Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription"`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820162002.png)

### Comprobación de PowerShell Transcription en Cliente Windows 11

Ejecutamos comando de prueba
`whoami`
`Get-Process`
`Get-Service`
`Write-Output "WAZUH-TRANSCRIPTION-TEST"`

`Get-ChildItem "C:\ProgramData\PowerShellTranscripts" -Recurse`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820163019.png)

El archivo de **16.409 bytes** significa que **PowerShell Transcription está funcionando correctamente y está escribiendo la actividad de PowerShell en el transcript**.

`Get-Content "C:\ProgramData\PowerShellTranscripts\20260820\PowerShell_transcript.WIN11-CLIENT01*.txt" -Tail 30`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820163734.png)

Vemos que ya en el documento se estan registrando comandos. **Eso confirma que PowerShell Transcription está funcionando correctamente.**

### Inclusión de PowerShell Transcripts en Wazuh Agent de Cliente Windows 11

El siguiente paso es hacer que **Wazuh Agent lea esos `.txt`** y los mande al servidor.
En `ossec.conf` del **CLIENT01**, agregaremos un `localfile` para: `C:\ProgramData\PowerShellTranscripts\*\*.txt`

`notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"`

```
<localfile>
    <location>C:\ProgramData\PowerShellTranscripts\20260820\*\*.txt</location>
    <log_format>syslog</log_format>
</localfile>
```

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820170845.png)

**Comprovacion que powershel transcription quedo activo**

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260820174342.png)

![](Images/WAZUH-SIEM/Pasted%20image%2020260820174259.png)

![](Images/WAZUH-SIEM/Pasted%20image%2020260820173843.png)

## Habilitación de registros de red en VICTIM01 para visualización en Wazuh

`auditpol /set /subcategory:"Conexión de plataforma de filtrado" /success:enable /failure:enable`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913150658.png)
# FortiOS (FW01)

## Configuración de Red FortiOS

![](Images/FW01-Firewall/Pasted%20image%2020260824142920.png)

![](Images/FW01-Firewall/Pasted%20image%2020260824143209.png)

Nos exige que la contraseña cumpla las políticas básicas de seguridad

```
admin
Cibhack123.123
```

![](Images/FW01-Firewall/Pasted%20image%2020260824143258.png)

`show system interface`

Esto nos mostrará las interfaces configuradas.

![](Images/FW01-Firewall/Pasted%20image%2020260824143457.png)

`get system interface physical`

Quiero ver qué interfaces aparecen y cuáles están `up`.

![](Images/FW01-Firewall/Pasted%20image%2020260824143549.png)

`get router info routing-table all`

Esto nos permitirá comprobar si FortiOS ya recibió una IP por DHCP en la interfaz conectada a **VMnet8**.
![](Images/FW01-Firewall/Pasted%20image%2020260824143654.png)

`execute ping 8.8.8.8`

Comprobamos correcta salida a internet

![](Images/FW01-Firewall/Pasted%20image%2020260824143755.png)

```
config system interface
edit port2
set ip 192.168.219.1 255.255.255.0
set allowaccess ping https ssh
next
end
```

Estamos convirtiendo **FW01 en el gateway de la LAN**, de esta menera FortiOS podrá registrar y controlar ese tráfico.

![](Images/FW01-Firewall/Pasted%20image%2020260824144245.png)

`get system interface physical`

![](Images/FW01-Firewall/Pasted%20image%2020260824151748.png)

```
admin
Cibhack123.123
```

Validamos la licencia ingresando nuestras credenciales privadas de FortiCare y s procede a realizar Reboot automatico.

![](Images/FW01-Firewall/Pasted%20image%2020260824165242.png)

![](Images/FW01-Firewall/Pasted%20image%2020260824165255.png)

![](Images/FW01-Firewall/Pasted%20image%2020260824171505.png)

![](Images/FW01-Firewall/Pasted%20image%2020260824171516.png)

Colocaremos que los parches tengan un delay para que se apliquen en un rango de horario especifico.

![](Images/FW01-Firewall/Pasted%20image%2020260824172105.png)

Accesso al dashboard de FortiGate

![](Images/FW01-Firewall/Pasted%20image%2020260824172230.png)

Firmware: v8.-.- build0167(Feature)

| Interfaz | Rol | Red esperada                |
| -------- | --- | --------------------------- |
| `port1`  | WAN | VMnet8 / DHCP / Internet    |
| `port2`  | LAN | `192.168.219.1/24` / VMnet1 |

Crearemos primero politica de firewall para la salida a internet

## Implementación e integración de FortiGate

**Configuración de Firewall Policy LAN → WAN con NAT y logging**

![](Images/FW01-Firewall/Pasted%20image%2020260828142451.png)

![](Images/FW01-Firewall/Pasted%20image%2020260828142506.png)

![](Images/FW01-Firewall/Pasted%20image%2020260828141749.png)

A medida que desarrollemos el proyecto en FortiOS iremos implementando Security Profiles

![](Images/FW01-Firewall/Pasted%20image%2020260828141808.png)

**All Sessions** registra también el tráfico permitido normal, lo cual nos permitira detectar anomalias. Activaremos inspeccion basica SSL por el momento.

![](Images/FW01-Firewall/Pasted%20image%2020260828142643.png)

### Limitación de HTTP y HTTPS expuesto a la WAN para paneles de administracion

![](Images/FW01-Firewall/Pasted%20image%2020260912181039.png)

![](Images/FW01-Firewall/Pasted%20image%2020260912181241.png)

### Configuracion FortiOS en SOC01

Actiamos el default gateway en SOC01

![](Images/SOC01-Monitoreo/Pasted%20image%2020260828143243.png)

`route -p add 0.0.0.0 mask 0.0.0.0 192.168.219.1`

![](Images/SOC01-Monitoreo/Pasted%20image%2020260828144417.png)

`nslookup google.com 8.8.8.8`

![](Images/SOC01-Monitoreo/Pasted%20image%2020260828144717.png)

Aqui lo que hicimos fue forzar el nslookup mediante la ip, sin embargo tenemos asociado un servidor DNS que esta presente en al DC01 donde ahora configuraremos teniendo todo mas claro la ip del gateway lo cual nos permitira tener esta salida sin forzar mediante ip.

### Configuracion Gateway en DC01

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260828150606.png)

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260828150706.png)

**Comprobación desde SOC01 uso DNS de DC01 con y sin forzado de IP**

![](Images/SOC01-Monitoreo/Pasted%20image%2020260828151506.png)

![](Images/SOC01-Monitoreo/Pasted%20image%2020260828151547.png)

![](Images/SOC01-Monitoreo/Pasted%20image%2020260828151634.png)

**Validamos sesiones de SOC01 en Forward Traffic FortiGate**

![](Images/FW01-Firewall/Pasted%20image%2020260828152929.png)

## Accesso a panel Admin FW01 exclusivo para SOC01

Es una muy buena practica para usuarios y sobre todo administrador activar el doble factor.

![](Images/FW01-Firewall/Pasted%20image%2020260912180207.png)

## Configuración de logs de FortiGate hacia Wazuh

Una vez comprobemos que los logs aparecen en FortiGate, configuraremos el envío hacia **WAZUH01** mediante **Syslog**.

```
<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>tcp</protocol>
  <allowed-ips>192.168.219.1/32</allowed-ips>
  <local_ip>192.168.219.20</local_ip>
</remote>

</ossec_config>
```

![](Images/WAZUH01-Server/Pasted%20image%2020260828154010.png)

![](Images/WAZUH01-Server/Pasted%20image%2020260828162823.png)

## Configuramos FortiGate para enviar Syslog

Envía mis logs al servidor Syslog `192.168.219.20`, por TCP/514.

```
config log syslogd setting
    set status enable
    set server "192.168.219.20"
    set mode reliable
    set port 514
    set source-ip 192.168.219.1
end
```

De los diferentes tipos de logs que genero, envía también los Forward Traffic.

```
config log syslogd filter
    set forward-traffic enable
end
```

![](Images/FW01-Firewall/Pasted%20image%2020260828164053.png)

```config log syslogd filter
    set severity information
    set forward-traffic enable
end
```

Tenemos confirmado que **FortiGate está configurado para enviar los logs de Forward Traffic por Syslog**.

![](Images/FW01-Firewall/Pasted%20image%2020260828164728.png)

**Comprobacion de deteccion de actividad SOC01 en WAZUH01**

![](Images/AD-Windows%20Server%202012/Pasted%20image%2020260828165030.png)

WAZUH01 recibiendo tráfico Syslog en TCP/514.

### Comprobación de registros entre FW01 a WAZUH01

Generaremos trafico en SOC01 y verificaremos al actividad en el servidor Wazuh01

`sudo tcpdump -ni any tcp port 514`

![](Images/WAZUH01-Server/Pasted%20image%2020260902141536.png)

Ahora generamos trafico nuevamente mediante `nslookup google.com` en SOC01 y verificaremos si la fuente se registra en syslog en WAZUH01.

`sudo tcpdump -ni any -A tcp port 514`

![](Images/WAZUH01-Server/Pasted%20image%2020260902141931.png)

### Configuración gateway VICTIM01

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260902170000.png)
# Host

Procedemos a modificar la ip del adaptador de red VMnet1 del pc host a otra direccion para que DC01 no tenga problemas en la salida hacia FW01 y pueda trabajar con normalidad el DNS

![](Images/HOST-VMs/Pasted%20image%2020260902133453.png)

---

## Regla para evitar acceso al panel de login de wazuh mediante VICTIM01

```bash
sudo ufw allow 22/tcp
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw deny from 192.168.219.30 to any port 443 proto tcp
sudo ufw allow 443/tcp
sudo ufw show added
```

![](Images/WAZUH01-Server/Pasted%20image%2020260905151153.png)

Habilitamos el firewall UFW en Wazuh01 y revisamos las reglas aplicadas

![](Images/WAZUH01-Server/Pasted%20image%2020260905151407.png)

### Comprobacion de regla de bloqueo UFW

Se valida que VICTIM01 específicamente ya no tiene acceso al panel de Wazuh sin embargo SOC01 el cual es especifico para monitoreo tiene el acceso al panel de wazuh.

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260905152055.png)

## Regla para evitar acceso a panel de login Fortinet desde VICTIM01

![](Images/FW01-Firewall/Pasted%20image%2020260905154238.png)

### Comprobación de regla de bloqueo Fortinet DESDE VICTIM01

![](Images/FW01-Firewall/Pasted%20image%2020260905154832.png)

# Kali Linux (Atacante)

## Configuración de red 

En este caso utilizaremos otro host en el cual estará montado mediante virtualbox la maquina kali linux atacante, para que la maquina kali tenga visibilidad con las interfaces creadas para la infraestructura, cambiamos el VMnet8 que simplemente actuaba como saliente a internet por bridged mode y dejando la interfaz de red interna intacta para no alterar otros aspectos de la red.

![](Images/FW01-Firewall/Pasted%20image%2020260906162237.png)

![](Images/FW01-Firewall/Pasted%20image%2020260906161904.png)

De esta forma dejaremos el acceso libre para la maquina KALI01 a la infraestructura permitiendo ataques de prueba, importante mencionar que la maquina kali tambien se encuentra en bridged mode.

### Comprobación de PING de KALI01 a FW01

`ping 192.128.1.185`

![](Images/KALI01-Atacante/Pasted%20image%2020260906163012.png)

# Exposición a algunas vulnerabilidades basicas a propósito para el entorno de pruebas.

## Habilitando Usuario débil

Usuario débil PowerShell: `net user admin2 Summer2024 /add`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910143301.png)

## Habilitando servicio SMB  Share visible

Share SMB visible Carpeta `C:\CompartidoPublico` con `credenciales.txt` dentro ( `victim01:1997`) → Properties > Sharing > Advanced Sharing > Share this folder > Permissions: Everyone (Read). Luego: `net user guest /active:yes`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910160306.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910160427.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910170820.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913164547.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913164741.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913165905.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913165923.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913170105.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913170138.png)

`net user guest`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913172003.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913172142.png)

**Habilitamos los permisos necesario para terminar de exponer el servicio y principalmente los recursos concretos en la carpeta compartida**

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260913175755.png)

## Habilitando servicio SSH

SSHSettings > Apps > Optional Features > Add a feature > 'OpenSSH Server' > Install. Luego: `Start-Service sshd` y `Set-Service -Name sshd -StartupType 'Automatic'`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910163227.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910165954.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910170644.png)

### Aplicar registros OpenSSH al agente Wazuh en VICTIM01

`notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"`

```powershell
<localfile>
  <location>OpenSSH/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260920105810.png)

## Habilitando servicio RDP

RDP Settings > System > Remote Desktop > Enable. Luego: `net localgroup "Remote Desktop Users" labuser /add`

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910161001.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910161023.png)

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260910161647.png)

## Exponiendo servicios a la WAN para VICTIM01 en FW01

### Creación Virtual IP RDP

![](Images/FW01-Firewall/Pasted%20image%2020260912161848.png)

### Creación Virtual IP SMB

![](Images/FW01-Firewall/Pasted%20image%2020260912165806.png)

### Creación Virtual IP SSH

![](Images/FW01-Firewall/Pasted%20image%2020260912165837.png)


![](Images/FW01-Firewall/Pasted%20image%2020260912165924.png)

### Creación de política única de firewall utilizando VIPs creados

![](Images/FW01-Firewall/Pasted%20image%2020260912170156.png)

![](Images/FW01-Firewall/Pasted%20image%2020260913153314.png)

![](Images/FW01-Firewall/Pasted%20image%2020260912170300.png)

# Ataques basicos a los servicios expuestos en la WAN a VICTIM01 mediante KALI01

## Escaneo de Interfaz WAN expuesta FW01 con visualización a servicios expuestos en maquina del AD

`nmap -Pn -sV -p- 192.168.1.185 -T5 -vvv`

![](Images/KALI01-Atacante/Pasted%20image%2020260912182043.png)

## Acceso mediante protocolo SMB expuesto en una de las maquinas del AD (VICTIM01)

`smbclient -L //192.168.1.185 -N`

`smbclient //192.168.1.185/Restore -N`

`smb: \> ls`

`smb: \> get credenciales.txt`

`smb: \> exit`

`cat credenciales.txt`

![](Images/KALI01-Atacante/Pasted%20image%2020260916133524.png)

Se encontraron las credenciales admin2:Summer2024, clara mala practica si encontráramos un caso de estas características real, en entornos de desarrollo se ha visto esta practica peligrosa.

## Acceso mediante protocolo RDP expuesto en una de las maquinas del AD (VICTIM01)

`xfreerdp /v:192.168.1.185 /u:admin2 /p:Summer2024`

![](Images/KALI01-Atacante/Pasted%20image%2020260916140135.png)

### Enumeracion basica en la cuenta accedida en la maquina dentro del AD VICTIM01

Si quisiéramos hacer una enumeración mas fuerte podríamos hacer uso de herramientas como winPEAS he incluso con ella escalada de privilegios, sin embargo hay muchas mas opciones si lo quisiesemos.

####  Ver el usuario actual y sus grupos

`ipconfig /all`

![](Images/KALI01-Atacante/Pasted%20image%2020260916141337.png)

`whoami /groups`

![](Images/KALI01-Atacante/Pasted%20image%2020260916141404.png)

#### Ver privilegios específicos de la cuenta (útil para saber si puedes escalar)

`whoami /priv`

![](Images/KALI01-Atacante/Pasted%20image%2020260916141431.png)

#### Ver si la cuenta es administrador local

`net user admin2`

![](Images/KALI01-Atacante/Pasted%20image%2020260916141455.png)

## Acceso mediante protocolo SSH expuesto en una de las maquinas del AD (VICTIM01)

`ssh admin2@192.168.1.185`

![](Images/KALI01-Atacante/Pasted%20image%2020260916142811.png)

`whoami` : identificación de usuario actual

`hostname` : identificación del lugar donde nos encontramos

`whoami /groups` : Verificación de Grupos

`whoami /priv` : Verificación de privilegios

`echo %COMSPEC%` : Nos permite verificar rápidamente si nos entrego una CMD o Powershell

`net user` : reconocimiento de otros usuarios en la maquina

![](Images/KALI01-Atacante/Pasted%20image%2020260916143157.png)

*ESCALADA DE PRIVILEGIOS PENDIENTE POR AHORA NO SE SI HACERLO CON LO TRABAJADO  CON LO DE SMB RDP Y SSH  AUN*
--

## Aplicar registros OpenSSH al agente en VICTIM01

`notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"`

```powershell
<localfile>
  <location>OpenSSH/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

![](Images/VICTIM01-Point%20of%20Access/Pasted%20image%2020260920105810.png)

# Creación de Reglas de detección en Wazuh 

# ![](Images/WAZUH-SIEM/Pasted%20image%2020260920113555.png)

## Reglas de detección - Acceso exitoso vía RDP, SSH y SMB

**Objetivo:** Detectar accesos exitosos a VICTIM01 a través de los 3 vectores expuestos hacia Internet (RDP, SSH, SMB), generando una alerta clara en Wazuh por cada uno, en vez de depender de revisar logs crudos manualmente.

```powershell
<group name="windows,lab_attack,">

  <!-- RDP: login exitoso -->
  <rule id="100110" level="8">
    <if_group>windows</if_group>
    <field name="win.system.eventID">^4624$</field>
    <field name="win.eventdata.logonType">^10$</field>
    <description>Login RDP exitoso detectado en $(win.system.computer)</description>
    <mitre>
      <id>T1021.001</id>
    </mitre>
  </rule>

  <!-- SSH: login exitoso -->
  <rule id="100111" level="8">
    <if_group>windows</if_group>
    <field name="win.system.channel">^OpenSSH/Operational$</field>
    <match>Accepted password</match>
    <description>Login SSH exitoso detectado en $(win.system.computer)</description>
    <mitre>
      <id>T1021.004</id>
    </mitre>
  </rule>

  <!-- SMB: acceso a share -->
  <rule id="100112" level="6">
    <if_group>windows</if_group>
    <field name="win.system.eventID">^5140$</field>
    <description>Acceso a recurso compartido SMB en $(win.system.computer)</description>
    <mitre>
      <id>T1021.002</id>
    </mitre>
  </rule>

</group>
```

![](Images/WAZUH-SIEM/Pasted%20image%2020260920120322.png)

### Regla 100110 - Login RDP exitoso

**Lógica:** El Event ID **4624** (inicio de sesión exitoso) es genérico - por sí solo puede corresponder a un login local, de red, de servicio, etc. Por eso la regla exige **dos condiciones simultáneas (AND)**: que el evento sea 4624 **y** que el campo `LogonType` sea exactamente **10** (`RemoteInteractive`), que es el valor específico que Windows asigna únicamente a sesiones iniciadas por Escritorio Remoto. Sin ese segundo filtro, la regla dispararía con cualquier tipo de login, no solo RDP.

### Regla 100111 - Login SSH exitoso

**Lógica:** Filtra específicamente ese canal, y dentro de él busca el texto **"Accepted password"** — la cadena literal que OpenSSH escribe cuando una autenticación por contraseña se completa con éxito

### Regla 100112 - Acceso a recurso compartido SMB

**Lógica:** El Event ID **5140** se genera específicamente cuando alguien establece una conexión hacia un recurso compartido de red (share SMB)

-----

**Personal Note**
1. ✅ AD
2. ✅ Wazuh Server
3. ✅ Windows SOC
4. ✅ CLIENT01
5. ✅ Wazuh Agent
6. ✅ Evento 4624
7. ✅ Sysmon
8. ✅ PowerShell Logging
9. ✅ Comprobar que Wazuh recibe esa telemetría
10. ✅ Kali
11. ✅ Ataques controlados
12. ⏭️ Investigación SOC L1
13. ⏭️ Wazuh Reglas
14. ⏭️ FortiOS

`sudo /var/ossec/bin/agent_control -l` = comando para ver agentes activos o no , ejecutar en ubuntu si

AD01       → VMnet1
WAZUH01    → VMnet1
SOC01      → VMnet1
CLIENT01   → VMnet1
FW01       → VMnet1 + VMnet8