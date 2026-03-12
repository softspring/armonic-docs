---
title: "CMS Bundle Collections"
description: "Extend the CMS with reusable module collections distributed as composer packages and loaded through configuration."
---

# Extending with collections

You can define your own reusable CMS components collections.

You require to create a composer package with *cms* folder structure and add it to the CMS configuration:

```yaml
sfs_cms:
    collections:
         - vendor/softspring/cms-module-collection
```
