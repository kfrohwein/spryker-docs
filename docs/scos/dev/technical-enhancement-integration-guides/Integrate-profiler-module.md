---
title: Integrate profiler module
description: Learn how to integrate profiler module
last_updated: March 22, 2023
template: howto-guide-template
---

This document describes how to integrate profiler module into a Spryker project.

## Prerequisites

To start the integration, install the necessary features:

| NAME                  | VERSION          | INTEGRATION GUIDE                                                                                                                                               |
|-----------------------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Spryker Core          | {{page.version}} | [Spryker Core feature integration](/docs/pbc/all/miscellaneous/{{site.version}}/install-and-upgrade/install-features/install-the-spryker-core-feature.html)     |
| Web Profiler for Zed  | {{page.version}} | [Web Profiler feature integration](/docs/scos/dev/technical-enhancement-integration-guides/integrating-development-tools/integrating-web-profiler-for-zed.html) |
| Web Profiler for Yves | {{page.version}} | [Web Profiler feature integration](/docs/scos/dev/technical-enhancement-integration-guides/integrating-development-tools/integrating-web-profiler-widget-for-yves.html) |

## 1) Enable extension

To collect execution traces, enable the `xhprof` extension.

```yaml
version: '0.1'

namespace: spryker
tag: 'dev'

environment: docker.dev
image:
  tag: spryker/php:8.1
  php:
    enabled-extensions:
      - xhprof
```

## 2) Bootstrap the Docker setup

```shell
docker/sdk boot deploy.dev.yml
```

## 3) Install the required modules using Composer

{% info_block warningBox "Verification" %}

Ensure that the following modules have been updated:

| MODULE                   | EXPECTED DIRECTORY                        |
|--------------------------|-------------------------------------------|
| EventDispatcherExtension | vendor/spryker/event-dispatcher-extension |
| WebProfilerExtension     | vendor/spryker/web-profiler-extension     |

{% endinfo_block %}

```shell
composer require --dev spryker/profiler
```

### 4) Set up behavior

1. For `Yves` application, register the following plugins:

| PLUGIN                                 | SPECIFICATION                                      | PREREQUISITES            | NAMESPACE                                                                                    |
|----------------------------------------|----------------------------------------------------|--------------------------|----------------------------------------------------------------------------------------------|
| ProfilerRequestEventDispatcherPlugin   | Starts collecting the data on request events.      | EventDispatcherExtension | Spryker\Zed\Profiler\Communication\Plugin\EventDispatcher                                    |
| WebProfilerProfilerDataCollectorPlugin | Shows the traces data in the web profiler toolbar. | WebProfilerExtension     | Spryker\Zed\Profiler\Communication\Plugin\WebProfiler\WebProfilerProfilerDataCollectorPlugin |


```php
<?php

namespace Pyz\Yves\EventDispatcher;

use Spryker\Yves\EventDispatcher\EventDispatcherDependencyProvider as SprykerEventDispatcherDependencyProvider;
use Spryker\Yves\Profiler\Plugin\EventDispatcher\ProfilerRequestEventDispatcherPlugin;


class EventDispatcherDependencyProvider extends SprykerEventDispatcherDependencyProvider
{
    /**
     * @return array<\Spryker\Shared\EventDispatcherExtension\Dependency\Plugin\EventDispatcherPluginInterface>
     */
    protected function getEventDispatcherPlugins(): array
    {
        $plugins = [
            //...
        ];
        
        if (class_exists(ProfilerRequestEventDispatcherPlugin::class)) {
            $plugins[] = new ProfilerRequestEventDispatcherPlugin();
        }
    
        return $plugins;
    }
}
```

```php
<?php

namespace Pyz\Yves\WebProfilerWidget;

use Spryker\Yves\Profiler\Plugin\WebProfiler\WebProfilerProfilerDataCollectorPlugin;
use SprykerShop\Yves\WebProfilerWidget\WebProfilerWidgetDependencyProvider as SprykerWebProfilerDependencyProvider;

class WebProfilerWidgetDependencyProvider extends SprykerWebProfilerDependencyProvider
{
    /**
     * @return array<\Spryker\Shared\WebProfilerExtension\Dependency\Plugin\WebProfilerDataCollectorPluginInterface>
     */
    public function getDataCollectorPlugins(): array
    {
        $plugins = [
            //...
        ];
        
        if (class_exists(WebProfilerProfilerDataCollectorPlugin::class)) {
            $plugins[] = new WebProfilerProfilerDataCollectorPlugin();
        }

        return $plugins;
    }
}

```

2. Register the following plugins for `Zed` application:

| PLUGIN                                 | SPECIFICATION                                      | PREREQUISITES            | NAMESPACE                                                                                    |
|----------------------------------------|----------------------------------------------------|--------------------------|----------------------------------------------------------------------------------------------|
| ProfilerRequestEventDispatcherPlugin   | Starts collecting the data on request events.      | EventDispatcherExtension | Spryker\Zed\Profiler\Communication\Plugin\EventDispatcher                                    |
| WebProfilerProfilerDataCollectorPlugin | Shows the traces data in the web profiler toolbar. | WebProfilerExtension     | Spryker\Zed\Profiler\Communication\Plugin\WebProfiler\WebProfilerProfilerDataCollectorPlugin |

```php
<?php

namespace Pyz\Zed\EventDispatcher;

use Spryker\Zed\Profiler\Communication\Plugin\EventDispatcher\ProfilerRequestEventDispatcherPlugin;
use Spryker\Zed\EventDispatcher\EventDispatcherDependencyProvider as SprykerEventDispatcherDependencyProvider;

class EventDispatcherDependencyProvider extends SprykerEventDispatcherDependencyProvider
{
    /**
     * @return array<\Spryker\Shared\EventDispatcherExtension\Dependency\Plugin\EventDispatcherPluginInterface>
     */
    protected function getEventDispatcherPlugins(): array
    {
        $plugins = [
            //...
        ];
        
        if (class_exists(ProfilerRequestEventDispatcherPlugin::class)) {
            $plugins[] = new ProfilerRequestEventDispatcherPlugin();
        }

        return $plugins;
    }
}
```

```php
<?php

namespace Pyz\Zed\WebProfiler;

use Spryker\Zed\Profiler\Communication\Plugin\WebProfiler\WebProfilerProfilerDataCollectorPlugin;
use Spryker\Zed\WebProfiler\WebProfilerDependencyProvider as SprykerWebProfilerDependencyProvider;

class WebProfilerDependencyProvider extends SprykerWebProfilerDependencyProvider
{
    /**
     * @return array<\Spryker\Shared\WebProfilerExtension\Dependency\Plugin\WebProfilerDataCollectorPluginInterface>
     */
    public function getDataCollectorPlugins(): array
    {
        $plugins = [
            //...
        ];
        
        if (class_exists(WebProfilerProfilerDataCollectorPlugin::class)) {
            $plugins[] = new WebProfilerProfilerDataCollectorPlugin();
        }

        return $plugins;
    }
}
```
