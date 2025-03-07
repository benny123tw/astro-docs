---
layout: ~/layouts/MainLayout.astro
title: Alias
description: Introducción a los alias en Astro.
i18nReady: true
---

Un **alias** es una forma de crear atajos para tus imports.

Los alias ayudan a mejorar la experiencia de desarrollo en repositorios con muchas carpetas o importaciones relativas.

```astro title="src/pages/about/company.astro" del="../../components" del="../../assets"
---
import Button from '../../components/controls/Button.astro';
import logoUrl from '../../assets/logo.png?url';
---
```

En este ejemplo, un desarrollador necesitaría comprender la relación de archivos entre `src/pages/about/company.astro`, `src/components/controls/Button.astro` y `src/assets/logo.png`. Y luego, si se moviera el archivo `company.astro`, estas importaciones también tendrían que actualizarse.

Puedes agregar alias de importación desde `tsconfig.json` o `jsconfig.json`.

```json title="tsconfig.json" ins={5-6}
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@components/*": ["src/components/*"],
      "@assets/*": ["src/assets/*"]
    }
  }
}
```

:::note
Asegúrate de que `compilerOptions.baseUrl` esté configurado para que las rutas con alias se puedan resolver.
:::

Con este cambio, ahora puedes importar usando los alias desde cualquier parte de su proyecto:

```astro title="src/pages/about/company.astro" ins="@components" ins="@assets"
---
import Button from '@components/Button';
import logoUrl from '@assets/logo.png';
---
```

Estos alias también se integran automáticamente a [VSCode](https://code.visualstudio.com/docs/languages/jsconfig) y otros editores.
