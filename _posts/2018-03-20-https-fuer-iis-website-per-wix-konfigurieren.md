---
layout: post
title:  "HTTPS für IIS-Webseite per WiX konfigurieren"
date:   2018-03-20 19:00:00
date_modified: 2018-04-17 12:00:00
excerpt: 'Der Artikel beschreibt, wie man mithilfe eines WiX-MSI-Installers beim Installieren einer Webanwendung die SSL-Verschlüsselung für die "Default Web Site" des IIS mit einem bereitgestellten Zertifikat konfigurieren kann.'
image:
thumb: /assets/img/thumbs/wisl.jpg
tags: [entwicklung, msi, wix, iis, https, ssl, zertifikate]
categories: [posts, development]
comments: true
lang: de
ref: post-configure-https-for-iis-website-by-wix
---

<!-- MDTOC maxdepth:6 firsth1:2 numbering:1 flatten:0 bullets:0 updateOnSave:1 -->

1. [Einleitung](#einleitung)   
2. [Anforderungen im Detail](#anforderungen-im-detail)   
3. [Die Lösung](#die-lösung)   
&emsp;3.1. [Vorbetrachtungen](#vorbetrachtungen)   
&emsp;3.2. [Konfiguration der "Default Web Site" des IIS](#konfiguration-der-default-web-site-des-iis)   
&emsp;3.3. [Externes Zertifikat installieren](#externes-zertifikat-installieren)   
&emsp;3.4. [Optionale Komponente](#optionale-komponente)   
4. [Zusammenfassung](#zusammenfassung)   

<!-- /MDTOC -->

## Einleitung

Aufgrund von Cybersecurity-Betrachtungen hatte ich neulich die User Story umzusetzen, mithilfe eines bereits existierenden MSI-Installers für eine Intranet-Webanwendung beim Kunden nun auch gleich SSL-Verschlüsselung für die `Default Web Site` des Internet Information Server(nachfolgend: IIS) anzuschalten, wenn an einem vordefinierten Ort die entsprechende PFX-Datei für das Serverzertifikat gefunden wird. Die Konfiguration sollte aber auch nach der Deinstallation des MSI-Pakets bestehen bleiben.

## Anforderungen im Detail

1. Der MSI-Installer (WiX) installiert die Webanwendung im IIS unter der bereits vorhandenen `Default Web Site` (Port 80).
1. Wird während der Installation in einem vordefinierten Verzeichnis ein valides Zertifikat gefunden (PFX-Datei mit festgelegtem Passwort), so wird die HTTPS-Bindung (Port 443) - falls noch nicht existierend - der `Default Web Site` hinzugefügt.
1. Für die hinzugefügte HTTPS-Bindung wird das zuvor bereitgestellte Zertifikat benutzt.
1. Wenn die Webanwendung per MSI deinstalliert wird, so soll zwar die Webanwendung aus der `Default Web Site` des IIS entfernt werden - nicht jedoch die `Default Web Site` selbst. Ebenso soll deren Serverzertifikat und die HTTPS-Bindung erhalten bleiben.
1. Eine erneute Installation soll eine schon vorhandene HTTPS-Bindung nicht überschreiben.

## Die Lösung

### Vorbetrachtungen

Wenn man per Windows Installer Toolkit (WiX) einen MSI-Installer für eine Webanwendung baut, die in eine bereits vorhandene IIS-Website installiert werden soll, dann sollte man diese nicht als Komponente im WiX-Code anlegen, sondern die vorhandene Website (nachfolgend gleichzusetzen mit `Default Web Site`) außerhalb von Komponenten definieren, damit man sie in den Komponenten der Webanwendung referenzieren kann. Dadurch vermeidet man, dass die IIS-Website bei der Deinstallation entfernt wird:

Man schreibt also in etwa Folgendes:

```xml
<?xml version="1.0" encoding="Windows-1252"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi"
     xmlns:iis="http://schemas.microsoft.com/wix/IIsExtension">
    <Product  ...>
    ...
        <iis:WebSite Id="DefaultWebSite"
                     Description="Default Web Site"
                     Directory="INSTALLDIR">
            <!--Following line just needed for compilation-->
            <iis:WebAddress Id="AllUnassigned"
                            Port="80"
                            IP="*" />
        </iis:WebSite>
    ...
        <ComponentGroup Id="IIS">
            <Component  Directory="MyWebDir"
                        Id="MyWebComponent"
                        Guid="..."
                        KeyPath="yes">
                <!--"DefaultWebSite" referenced in next line-->
                <iis:WebVirtualDir  WebSite="DefaultWebSite"
                                    Id="MyWebApp"
                                    Alias="MyWebApp"
                                    Directory="MyWebDir" >
                    <iis:WebApplication Id="MyWebApplication"
                                        Name="MyWeb"
                                        WebAppPool="MyAppPool" />
                </iis:WebVirtualDir>
                <iis:WebAppPool Id="MyAppPool"
                                Name="MyAppPool"
                                MaxCpuUsage="75"
                                ManagedRuntimeVersion="v4.0"
                                ManagedPipelineMode="Integrated">
                </iis:WebAppPool>
            </Component>
        </ComponentGroup>
    ...
    </Product>
</Wix>
```

Will man jedoch die `Default Web Site` nicht nur referenzieren sondern auch konfigurieren, dann muss eine andere Vorgehensweise her!

### Konfiguration der "Default Web Site" des IIS

Um diese konfigurieren zu können, muss man sie nun doch in eine Komponente packen. Außerdem benötigt man noch die Information, wo sich der IIS-Rootfolder (normalerweise `c:\inetpub\wwwroot`) befindet:

```xml
<Property Id="IIS_ROOT">
  <RegistrySearch Id="FindInetPubFolder"
                Root="HKLM"
                Key="SOFTWARE\Microsoft\InetStp"
                Name="PathWWWRoot"
                Type="directory" />
</Property>

<Directory  Id="TARGETDIR"
            Name="SourceDir">
    <Directory  Id="WWWROOT"
                Name="wwwroot" />
</Directory>

<CustomAction Id="SetWWRootDirFromIIS"
                Return="check"
                Property="WWWROOT"
                Value="[IIS_ROOT]"
                Execute="firstSequence" />

<InstallUISequence>
    <Custom Action="SetWWRootDirFromIIS"
            After="AppSearch">
        (MaintenanceMode="Modify" OR NOT INSTALLED) AND IIS_ROOT
    </Custom>
</InstallUISequence>

<InstallExecuteSequence>
    <Custom Action="SetWWRootDirFromIIS"
            After="AppSearch">
        (MaintenanceMode="Modify" OR NOT INSTALLED) AND IIS_ROOT
    </Custom>
</InstallExecuteSequence>

```

Würde man ein statisches Zertifikat mit ausliefern wollen (entspricht nicht den Anforderungen!), so könnte man es einfach als Binary deklarieren, z.B.:

```xml
<Binary Id="certBinary" SourceFile="MyServer.cert.pfx"/>
```

Somit ließe sich folgende Komponente zusammenstellen:

```xml
<!-- ATTENTION: This component configures
the existing IIS "Default Web Site" with ssl. -->
<Component Id="InstallHttps"
        Guid="..."
        KeyPath="yes"
        Directory="WWWROOT" >
  <iis:Certificate Id="cert"
        BinaryKey="certBinary"
        Name="MyServerCert"
        StoreLocation="localMachine"
        StoreName="personal"
        PFXPassword="password123"
        Request="no"
        Overwrite="yes" />
  <iis:WebSite Id="DefaultWebSite"
        Description="Default Web Site"
        ConfigureIfExists="yes"
        Directory="WWWROOT" >
    <iis:WebAddress Id="AllUnassignedHttps"
            Port="443"
            IP="*"
            Secure="yes" />
    <iis:CertificateRef Id='cert' />
  </iis:WebSite>
</Component>
```

Damit die `Default Web Site` bei der Deinstallation nicht gelöscht wird, muss man die Komponente um folgende Tags erweitern:

* `Permanent="yes"` und `NeverOverwrite="yes"`

```xml
<!-- ATTENTION: This component configures
the existing IIS "Default Web Site" with ssl.
It must be marked with Permanent="yes"!
Otherwise the IIS "Default Web Site"
will be removed on uninstall. -->
<Component Id="InstallHttps"
        Guid="..."
        KeyPath="yes"
        Directory="WWWROOT"
        NeverOverwrite="yes"
        Permanent="yes" >
  <iis:Certificate Id="cert"
        BinaryKey="certBinary"
        Name="MyServerCert"
        StoreLocation="localMachine"
        StoreName="personal"
        PFXPassword="password123"
        Request="no"
        Overwrite="yes" />
  <iis:WebSite Id="DefaultWebSite"
        Description="Default Web Site"
        ConfigureIfExists="yes"
        Directory="WWWROOT" >
    <iis:WebAddress Id="AllUnassignedHttps"
            Port="443"
            IP="*"
            Secure="yes" />
    <iis:CertificateRef Id='cert' />
  </iis:WebSite>
</Component>
```

### Externes Zertifikat installieren

Im vorangegangenen Schritt haben wir ein statisches Serverzertifikat mitgeliefert, was für Webanwendungen, die in populären Browsern ausgeführt werden, jedoch nicht besonders sinnvoll ist, da ein solch statisches Zertifikat nicht die Qualität hat, dass diese Webbrowser der Verbindung vertrauen (Zerifikat muss überprüfbare Informationen zum Server enthalten).

Nehmen wir also an, dass für diesen Webbrowser ein spezielles Zertifikat namens `MyServerCert.pfx` mit einem vorgegebenen Passwort erstellt und im folgenden vereinbarten Verzeichnis abgelegt wurde:

```
c:\ProgramData\MyCompany\Certificates\
```

Somit müssen wir unseren WiX-Komponenten-Code wie folgt modifizieren:

* entferne folgende Zeile wieder:

```xml
<Binary Id="certBinary" SourceFile="MyServer.cert.pfx"/>
```

* Ändere die Komponentendefinition (iis:Certificate):

```xml
<!-- ATTENTION: This component configures
the existing IIS "Default Web Site" with ssl.
It must be marked with Permanent="yes"!
Otherwise the IIS "Default Web Site"
will be removed on uninstall. -->
<Component Id="InstallHttps"
        Guid="..."
        KeyPath="yes"
        Directory="WWWROOT"
        NeverOverwrite="yes"
        Permanent="yes" >
  <iis:Certificate Id="cert"
        CertificatePath="[CommonAppDataFolder]\MyCompany\Certificates\MyServerCert.pfx"
        Name="MyServerCert"
        StoreLocation="localMachine"
        StoreName="personal"
        PFXPassword="password123"
        Request="no"
        Overwrite="no" />
  <iis:WebSite Id="DefaultWebSite"
        Description="Default Web Site"
        ConfigureIfExists="yes"
        Directory="WWWROOT" >
    <iis:WebAddress Id="AllUnassignedHttps"
            Port="443"
            IP="*"
            Secure="yes" />
    <iis:CertificateRef Id='cert' />
  </iis:WebSite>
</Component>
```

Statt des Binärschlüssels muss nun also der Zertifikatspfad angegeben werden!

_**Achtung:**_ Außerdem muss man `Overwrite` auf `no` setzen, sonst schlägt die Installation fehl.
{: .notice--warning}

### Optionale Komponente

Zum Schluss muss nun noch die Anforderung erfüllt werden, dass die Konfiguration nur erfolgt, wenn ein Zertifikat gefunden wurde.
Dazu suchen wir zunächst mal nach der Datei `MyServerCert.pfx` und setzen eine Property `SERVERCERT`:

```xml
<Property Id="SERVERCERT">
  <DirectorySearch Id="FindServerCertificate"
        Path="[CommonAppDataFolder]\MyCompany\Certificates"
        Depth ="1">
    <FileSearch Name="MyServerCert.pfx" />
  </DirectorySearch>
</Property>
```

Dann setzen wir für die Installation der Komponente eine Bedingung:

```xml
<Condition><![CDATA[SERVERCERT <> ""]]></Condition>
```

Die Konfiguration der `Default Web Site` wird also nur ausgeführt, wenn die Datei gefunden und somit die Property gefüllt wurde!

## Zusammenfassung

Nachfolgender Pseudocode fasst die vorgestellten Schritte und Fragmente nochmals zusammen:

```xml
<?xml version="1.0" encoding="Windows-1252"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi"
     xmlns:iis="http://schemas.microsoft.com/wix/IIsExtension">
    <Product  ...>
    ...
        <Property Id="SERVERCERT">
            <DirectorySearch Id="FindServerCertificate"
                    Path="[CommonAppDataFolder]\MyCompany\Certificates"
                    Depth ="1">
                <FileSearch Name="MyServerCert.pfx" />
            </DirectorySearch>
        </Property>
    ...
        <Property Id="IIS_ROOT">
            <RegistrySearch Id="FindInetPubFolder"
                            Root="HKLM"
                            Key="SOFTWARE\Microsoft\InetStp"
                            Name="PathWWWRoot"
                            Type="directory" />
        </Property>
    ...
        <CustomAction Id="SetWWRootDirFromIIS"
                        Return="check"
                        Property="WWWROOT"
                        Value="[IIS_ROOT]"
                        Execute="firstSequence" />
    ...
        <Directory  Id="TARGETDIR"
                    Name="SourceDir">
            <Directory  Id="WWWROOT"
                        Name="wwwroot" />
        </Directory>
    ...
        <ComponentGroup Id="IIS">
            <!-- ATTENTION: This component configures
            the existing IIS "Default Web Site" with ssl.
            It must be marked with Permanent="yes"!
            Otherwise the IIS "Default Web Site"
            will be removed on uninstall. -->
            <Component Id="InstallHttps"
                    Guid="..."
                    KeyPath="yes"
                    Directory="WWWROOT"
                    NeverOverwrite="yes"
                    Permanent="yes" >
                <Condition><![CDATA[SERVERCERT <> ""]]></Condition>
                <iis:Certificate Id="cert"
                        CertificatePath="[CommonAppDataFolder]\MyCompany\Certificates\MyServerCert.pfx"
                        Name="MyServerCert"
                        StoreLocation="localMachine"
                        StoreName="personal"
                        PFXPassword="password123"
                        Request="no"
                        Overwrite="no" />
                <iis:WebSite Id="DefaultWebSite"
                        Description="Default Web Site"
                        ConfigureIfExists="yes"
                        Directory="WWWROOT" >
                <iis:WebAddress Id="AllUnassignedHttps"
                        Port="443"
                        IP="*"
                        Secure="yes" />
                <iis:CertificateRef Id='cert' />
            </iis:WebSite>
            </Component>
    ...
            <Component  Directory="MyWebDir"
                        Id="MyWebComponent"
                        Guid="..."
                        KeyPath="yes">
                <!--"DefaultWebSite" referenced in next line-->
                <iis:WebVirtualDir  WebSite="DefaultWebSite"
                                    Id="MyWebApp"
                                    Alias="MyWebApp"
                                    Directory="MyWebDir" >
                    <iis:WebApplication Id="MyWebApplication"
                                        Name="MyWeb"
                                        WebAppPool="MyAppPool" />
                </iis:WebVirtualDir>
                <iis:WebAppPool Id="MyAppPool"
                                Name="MyAppPool"
                                MaxCpuUsage="75"
                                ManagedRuntimeVersion="v4.0"
                                ManagedPipelineMode="Integrated">
                </iis:WebAppPool>
            </Component>
        </ComponentGroup>
    ...
        <InstallUISequence>
            <Custom Action="SetWWRootDirFromIIS"
                    After="AppSearch">
                (MaintenanceMode="Modify" OR NOT INSTALLED) AND IIS_ROOT
            </Custom>
        </InstallUISequence>

        <InstallExecuteSequence>
            <Custom Action="SetWWRootDirFromIIS"
                    After="AppSearch">
                (MaintenanceMode="Modify" OR NOT INSTALLED) AND IIS_ROOT
            </Custom>
        </InstallExecuteSequence>
    ...
    </Product>
</Wix>
```
