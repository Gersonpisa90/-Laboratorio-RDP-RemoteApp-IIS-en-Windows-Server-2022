# -Laboratorio-RDP-RemoteApp-IIS-en-Windows-Server-2022

Configurar los servicios de RDP RemoteApp y RDP RemoteApp Web Client en Windows Server 2022, desplegar una página web personalizada en IIS, publicarla como aplicación RemoteApp, y acceder a ella desde un cliente Windows 10 mediante los dos métodos disponibles: el portal web clásico (/RDWeb) y el cliente HTML5 moderno (/RDWeb/webclient).

🗺️ Topología del Laboratorio
┌─────────────────────────────────────────────────────────┐
│                     EVE-NG (Hypervisor)                  │
│                                                         │
│  ┌──────────────────────────┐   ┌─────────────────────┐ │
│  │   Windows Server 2022    │   │    Windows 10        │ │
│  │  WIN-JGPNS8B9AJ5         │   │    (Cliente)         │ │
│  │  IP: 10.15.29.14         │◄──►│  IP: DHCP           │ │
│  │  Dominio: itla.edu.do    │   │  Dominio: itla.edu.do│ │
│  │                          │   │                      │ │
│  │  Roles instalados:       │   │  Acceso via:         │ │
│  │  ✔ AD DS                 │   │  ✔ mstsc (RDP)       │ │
│  │  ✔ DNS                   │   │  ✔ Navegador /RDWeb  │ │
│  │  ✔ IIS (puerto 8080)     │   │  ✔ /RDWeb/webclient  │ │
│  │  ✔ Remote Desktop Svcs   │   │                      │ │
│  └──────────────────────────┘   └─────────────────────┘ │
└─────────────────────────────────────────────────────────┘
Direccionamiento IP
DispositivoHostnameIPRolWindows Server 2022WIN-JGPNS8B9AJ510.15.29.14Servidor RDS / AD DS / IISWindows 10—DHCP (itla.edu.do)Cliente RDP / Web
Puertos utilizados
PuertoProtocoloServicio3389TCPRDP (Remote Desktop)443TCP/HTTPSRD Web Access / Web Client8080TCP/HTTPIIS Página personalizada

⚙️ Roles y Características Instalados
Rol / CaracterísticaDescripciónRemote Desktop ServicesRol principal de RDSRD Connection BrokerGestión de conexiones RemoteAppRD Session HostHost de sesiones remotasRD Web AccessPortal web para RemoteApp (/RDWeb)IIS (Web Server)Servidor web para página personalizadaAD DSActive Directory (dominio itla.edu.do)DNSResolución de nombres del dominio

🔧 Configuraciones Realizadas
1. Instalación de Remote Desktop Services
Se instaló el rol RDS usando la opción Quick Start → Session-based desktop deployment desde Server Manager. Esto instaló automáticamente los sub-roles: RD Connection Broker, RD Session Host y RD Web Access.
powershell# Verificación de roles instalados
Get-WindowsFeature | Where-Object {$_.Installed -eq $true -and $_.Name -like "RDS*"}

2. Publicación de RemoteApp Programs
Las siguientes aplicaciones fueron publicadas en la colección QuickSessionCollection:
AplicaciónAliasVisible en RD Web AccessCalculatorCalculatorSíCalculator (win32)win32calcSíMicrosoft EdgemsedgeSíNotepadnotepadSíPaintPaintSíWordPadWordPadSí
Microsoft Edge fue configurado para abrir directamente la página IIS:
powershellSet-RDRemoteApp `
    -CollectionName "QuickSessionCollection" `
    -ConnectionBroker "WIN-JGPNS8B9AJ5.itla.edu.do" `
    -Alias "msedge" `
    -CommandLineSetting Require `
    -RequiredCommandLine "http://10.15.29.14:8080"

3. Página IIS Personalizada
Se creó un sitio web personalizado en IIS escuchando en el puerto 8080:
powershell# Crear directorio del sitio
New-Item -Path "C:\inetpub\wwwroot\MiSitio" -ItemType Directory

# Crear sitio en IIS
New-WebSite -Name "MiSitioRDS" -Port 8080 `
            -PhysicalPath "C:\inetpub\wwwroot\MiSitio" -Force

# Abrir puerto en Firewall
New-NetFirewallRule -DisplayName "IIS Puerto 8080" `
                    -Direction Inbound -Protocol TCP `
                    -LocalPort 8080 -Action Allow
La página muestra información del servidor en tiempo real:

Nombre del servidor (WIN-JGPNS8B9AJ5)
Estado de los servicios (RDS, RemoteApp, Web Client)
Fecha y hora del servidor

Accesible en: http://10.15.29.14:8080

4. RD Web Client Moderno (HTML5)
Se instaló el paquete del Web Client moderno versión 2.1.65.0:
powershell# Instalar módulo de administración
Install-Module -Name RDWebClientManagement -Force -AcceptLicense

# Crear y exportar certificado
$cert = New-SelfSignedCertificate `
    -DnsName "WIN-JGPNS8B9AJ5.itla.edu.do", "10.15.29.14" `
    -CertStoreLocation "cert:\LocalMachine\My" `
    -NotAfter (Get-Date).AddYears(5)

Export-Certificate -Cert $cert -FilePath "C:\Temp\broker2.cer" -Force

# Importar certificado al Web Client
Import-RDWebClientBrokerCert -Path "C:\Temp\broker2.cer"

# Instalar y publicar el paquete
Install-RDWebClientPackage
Publish-RDWebClientPackage -Type Production -Latest
Accesible en: https://10.15.29.14/RDWeb/webclient

🖥️ Acceso desde el Cliente Windows 10
Método 1 — RD Web Access Clásico

Abrir navegador en Windows 10
Ir a: https://10.15.29.14/RDWeb
Iniciar sesión con: itla.edu.do\Administrator
Hacer click en Microsoft Edge
Se descarga un archivo .rdp que al ejecutarse abre Edge remotamente con la página IIS

Método 2 — RD Web Client HTML5

Abrir navegador en Windows 10
Ir a: https://10.15.29.14/RDWeb/webclient
Iniciar sesión con: itla.edu.do\Administrator
Hacer click en Microsoft Edge directamente desde el navegador
Edge se abre remotamente sin necesidad de descargar ningún archivo

Método 3 — RDP Directo (mstsc)

Presionar Win + R → escribir mstsc
Computer: WIN-JGPNS8B9AJ5.ITLA.EDU.DO
Usuario: ITLA\Administrator
Una vez conectado, abrir Edge y visitar http://10.15.29.14:8080


📊 Resumen de Resultados
TareaEstadoInstalación de rol RDS✅ CompletadoPublicación de RemoteApp Programs✅ CompletadoPágina IIS personalizada (puerto 8080)✅ CompletadoEdge RemoteApp apuntando a página IIS✅ CompletadoRD Web Access clásico (/RDWeb)✅ CompletadoRD Web Client HTML5 (/RDWeb/webclient)✅ CompletadoAcceso por RDP directo desde Windows 10✅ CompletadoPágina IIS visible vía RDP y RemoteApp✅ Completado

📁 Estructura del Repositorio
📦 rdp-remoteapp-lab
 ┣ 📄 README.md          ← Documentación técnica (este archivo)
 ┣ 📄 documentacion.pdf  ← Versión PDF de la documentación
 ┗ 🎬 video_demo.mp4     ← Video de demostración (máx. 8 min)

🔗 URLs de Acceso
ServicioURLPágina IIShttp://10.15.29.14:8080RD Web Accesshttps://10.15.29.14/RDWebRD Web Client HTML5https://10.15.29.14/RDWeb/webclient
