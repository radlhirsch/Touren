---
layout: post
title:  "Feature Toggles"
date:   2016-02-17
excerpt: "A simple feature toggling framework for .net."
image:
thumb:
project: true
categories: [projects, development]
tag:
- features 
- toggles
- featuretoggles
- dotnet
- c#
comments: true
lang: en
ref: project-feature-toggles
---

The source code of this project can be found here: [MS.Features](https://github.com/mcpride/MS.Features)


## Feature specification

```
[FeatureToggleSettingsStrategy(Key = "Sample Feature")]
[AppSettingsStrategy(Key = "Sample_Feature")]
public class SampleFeature : FeatureBase
{
}
```

## Configuration

### Appsettings

* app.config:

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="Sample_Feature" value="true" />
  </appSettings>
</configuration>
```

### Separate FeatureToggle Settings file

* app.config:

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <configSections>
    <section name="featureToggles" type="MS.Features.Configuration.FeatureToggleSettings, MS.Features" />
  </configSections>
  <featureToggles configSource="featureToggles.config" />
</configuration>
```

* featureToggles.config:

```
<featureToggles>
  <features>
    <feature name="Sample Feature" enabled="true"/>
    <feature name="FeatureY" enabled="false"/>
  </features>
</featureToggles>
```

## Bootstrap code

* auto discovery mode:

```
new FeatureSetBuilder().Build();
```

## Query features

```
FeatureContext.IsEnabled<SampleFeature>()
```
