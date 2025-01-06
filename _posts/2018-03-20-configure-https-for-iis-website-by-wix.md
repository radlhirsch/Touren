---
layout: post
title:  "Configure HTTPS for IIS website by WiX setup"
date:   2018-03-20 19:00:00
date_modified: 2018-04-17 12:00:00
excerpt: 'This article describes how to use a WiX MSI installer to configure SSL encryption for the default web site of IIS with a provided certificate when installing a web application.'
image:
thumb: /assets/img/thumbs/wisl.jpg
tags: [development, msi, wix, iis, https, ssl, certificates]
categories: [posts, development]
comments: true
lang: en
ref: post-configure-https-for-iis-website-by-wix
---

<!-- MDTOC maxdepth:6 firsth1:2 numbering:1 flatten:0 bullets:0 updateOnSave:1 -->

1. [Introduction](#introduction)   
2. [Detailed requirements](#detailed-requirements)   
3. [The solution](#the-solution)   
&emsp;3.1. [Introspection](#introspection)   
&emsp;3.2. [Configuration of the "Default Web Site" of IIS](#configuration-of-the-default-web-site-of-iis)   
&emsp;3.3. [Install an external certificate](#install-an-external-certificate)   
&emsp;3.4. [Optional component](#optional-component)   
4. [Summary](#summary)   

<!-- /MDTOC -->

## Introduction

Due to cybersecurity considerations, I recently had to implement a user story, using an existing MSI installer for an intranet web application at the customer's site, to enable SSL encryption for the Internet Information Server's `Default Web Site` (hereinafter: IIS) when the corresponding PFX file for the server certificate is found at a predefined location. However, the configuration should remain after uninstalling the MSI package.

## Detailed requirements

1. The MSI installer (WiX) installs the web application in IIS under the existing `Default Web Site` (port 80).
1. If a valid certificate is found during the installation in a predefined directory (PFX file with defined password), the HTTPS binding (port 443) - if not already existing - is added to the `Default Web Site`.
1. The previously provided certificate is used for the added HTTPS binding.
1. If the web application is uninstalled via MSI, the web application should be removed from the `Default Web Site` of IIS, but not the `Default Web Site` itself. Their server certificate and the HTTPS binding should also be preserved.
1. Reinstallation should not overwrite an existing HTTPS binding.

## The solution

### Introspection

If you are building an MSI installer for a web application using the Windows Installer Toolkit (WiX) that you want to install on an existing IIS Web site, then you should not create it as a component in the WiX code, but define the existing website (hereinafter referred to as `Default Web Site`) outside of components so that you can reference it in other components of the installer. This prevents the IIS website from being removed during uninstallation:

So you write something like this:

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

However, if you do not just want to reference the `Default Web Site` but also configure it, you have to do it another way!

### Configuration of the "Default Web Site" of IIS

To configure them, you now have to pack them into a component. You also need to know where the IIS root folder is located (usually `c:\inetpub\wwwwroot`):

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

If you would like to deliver a static certificate (does not meet the requirements!), you could simply declare it as binary, e.g.:

```xml
<Binary Id="certBinary" SourceFile="MyServer.cert.pfx"/>
```

Thus, the following component could be put together:

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

So that the `Default Web Site` is not deleted during uninstallation, you have to extend the component by the following tags:

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

### Install an external certificate

In the previous step, we provided a static server certificate, which is not particularly useful for web applications running in popular browsers, since such a static certificate does not have the quality that these web browsers trust the connection (certificate must contain verifiable information about the server).

Let's assume that a special certificate named `MyServerCert.pfx` was created for this web browser with a default password and stored in the following agreed directory:

```
c:\ProgramData\MyCompany\Certificates\
```

So we need to modify our WiX component code as follows:

* remove the following line again:

```xml
<Binary Id="certBinary" SourceFile="MyServer.cert.pfx"/>
```

* Change the component definition (iis:Certificate):

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

Instead of the binary key, the certificate path must now be specified!

_**Attention:**_ You also have to set `Overwrite` to `no`, otherwise the installation will fail.
{: .notice--warning}

### Optional component

Finally, the requirement must now be fulfilled that the configuration only takes place if a certificate was found.
For this we first look for the file `MyServerCert.pfx` and set a property `SERVERCERT`.:

```xml
<Property Id="SERVERCERT">
  <DirectorySearch Id="FindServerCertificate"
        Path="[CommonAppDataFolder]\MyCompany\Certificates"
        Depth ="1">
    <FileSearch Name="MyServerCert.pfx" />
  </DirectorySearch>
</Property>
```

Then we set a condition for the installation of the component:

```xml
<Condition><![CDATA[SERVERCERT <> ""]]></Condition>
```

The configuration of the `Default Web Site` will only be executed if the file was found and the property was filled!

## Summary

The following pseudocode summarizes the presented steps and fragments again:

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
