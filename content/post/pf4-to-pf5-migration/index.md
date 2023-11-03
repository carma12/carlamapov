---
title: Migrating PatternFly v4 to v5
description: Hints and tips to make a successful migration
slug: pf4-to-pf5-migration
date: 2023-11-03 00:00:00+0000
image: cover.jpg
categories:
    - PatternFly
---

# Introduction
PatternFly 5 version was recently released in August 2023. As a consumer of the previous one (PF4), I feel necessary a guidance on how to efectively conduct the migration into the newer version, making use of the newly released [codemods](https://github.com/patternfly/pf-codemods/). Here is my contribution into that :)

# Migration steps
## Upgrade PF version

Open the `package.json` file and change the version of all PatternFly libraries (based on its [promoted package versions](https://www.patternfly.org/get-started/release-highlights/#promoted-package-versions)). In the example below I'm taking my project libraries as a reference.

```ts
"dependencies": {
    "@patternfly/patternfly": "^5.1.0",
    "@patternfly/react-core": "^5.1.1",
    "@patternfly/react-table": "^5.1.1",
    // (...)
},
```

Save the file and install the new packages from the command line:
```
>> npm install @patternfly/patternfly @patternfly/react-core @patternfly/react-table --save
```

You should have now the PatternFly libraries updated to v.5. The following step will be to adapt the existing components using the codemods.

## PF Code-mods
