---
title: Migrating PatternFly v4 to v5
description: Hints and tips to make a successful migration
slug: pf4-to-pf5-migration
date: 2023-12-19 00:00:00+0000
image: andrew-ridley-jR4Zf-riEjI-unsplash.jpg
categories:
  - PatternFly
---

# Introduction

[PatternFly 5](https://www.patternfly.org/) version was recently released in August 2023. This is a major upgrade of the previous version and has landed with some exciting new features. To mention some:

- It supports React versions 17 and 18
- They introduce new federated modules: a collection of independent remote software modules (associated with microfrontends)
- Updated implementation of some popular components: Wizard, Select, Dropdown, and Table
- Acessibility improvements

The PatternFly team has also announced that they will adopt an annual major release cadence, so we can expect a PatternFly 6 library for mid-late 2024. This also means that the previous version of PatternFly 4 library will be only maintained for addressing high-priority issues until PF6 is available, because they will be commited to maintain two PatternFly versions at the same time.

As a consumer of the previous one (PF4), this sounds really exciting and I couldn't wait to check the newly released PatternFly 5 library. So I would like to share my experience adapting this library to the [modern FreeIPA WebUI project](https://github.com/freeipa/freeipa-webui) and providing some hints that I learned during the process. I hope this will help you for a successful migration ;-)

# Migration steps

## Upgrade PF version

The first step will be to upgrade the PatternFly libraries to its latest version. Easy as that.

In my case, as I'm using NodeJS I can rely on the `npm` command to upgrade the three specific PF libraries in a single line:

```console
npm install @patternfly/patternfly@latest @patternfly/react-core@latest @patternfly/react-table@latest
```

The result:

```ts
"dependencies": {
    "@patternfly/patternfly": "^5.1.0",
    "@patternfly/react-core": "^5.1.1",
    "@patternfly/react-table": "^5.1.1",
    // (...)
},
```

If your project uses any other PF library, the command mentioned above still will do the trick. Feel free to check the [promoted package versions](https://www.patternfly.org/get-started/release-highlights/#promoted-package-versions)) for further information.

## PatternFly Codemods

To make the migration easier, the PatternFly team has released a set of codemods to easily change, identify, and fix major issues. These can automatically adapt and resolve a large amount of problems on the code and flag any component that might require manual update.

In the [oficial guidelines](https://www.patternfly.org/get-started/upgrade), they recommend to use two specific codemods:

- [pf-codemods](https://github.com/patternfly/pf-codemods/tree/main/packages/pf-codemods) is an eslint wrapper to update `@patternfly/react-core@4.x.x` code to `5.x.x`. It will also assign some deprecated components with the `@deprecated` library.
- [class-name-updater](https://github.com/patternfly/pf-codemods/tree/main/packages/class-name-updater) automatically identifies Patternfly class names that need to be updated after the introduction of versioned class names in Patternfly v5. For example: the class `pf-u-mt-lg` will be transformed into `pf-v5-u-mt-lg`.

Those can be run in several steps regardless of the order. And there is something else to take into account: those codemods are meant to be used **before** making any manual changes to the code. That means that those errors and warnings can still be there even when you have resolved them, leading to the "what-did-just-happened??" loop. So just to take it into account :-)

### pf-codemods

To use this codemod and check the errors and warnings, we just execute the following command:

```console
npx @patternfly/pf-codemods@latest <path-to-source-code>
```

To allow the autofix to apply changes into our code (I strongly recommend this approach), we add the `--fix` argument:

```console
npx @patternfly/pf-codemods@latest <path-to-source-code> --fix
```

### class-name-updater

This codemod works very similar as the one that was mentioned before: it can be executed to _check_ the occurrencies or to _fix_ them. The last one is the most useful in this case, IMHO.

```console
npx @patternfly/class-name-updater <path-to-source-code> --fix
```

## Upgrade deprecated components

PatternFly 5 deprecates old implementations of some components in favor of new ones. These are:

- [Select](https://www.patternfly.org/components/menus/select/react-deprecated)
- [Dropdown](https://www.patternfly.org/components/menus/dropdown/react-deprecated)
- [Table](https://www.patternfly.org/components/table/react-deprecated)
- [Wizard](https://www.patternfly.org/components/wizard/react-deprecated)
- [Context selector](https://www.patternfly.org/components/menus/context-selector/)
- [Application launcher](https://www.patternfly.org/components/menus/application-launcher)
- [Options menu](https://www.patternfly.org/components/menus/options-menu)
- [Page header](https://www.patternfly.org/components/page/react-deprecated)

Those will remain in the code base until, at least, the next major release (PF6).

So for those projects that started with the previous PF4 version (like the [new FreeIPA WebUI](https://github.com/freeipa/freeipa-webui)), is important to adapt those components to use the new ones. In order to do so, we just need to search in your code any PatternFly library that has the `deprecated` suffix. E.g. using the `Select` components:

```ts
import {
  Select,
  SelectOption,
  SelectVariant,
  SelectDirection,
} from "@patternfly/react-core/deprecated";
```

Once we identify the deprecated components in our code, we just need to replace them by the non-deprecated ones referencing the `@patternfly/react-core` library.

```ts
import {
  Select,
  SelectOption,
  SelectVariant,
  SelectDirection,
} from "@patternfly/react-core";
```

It is important to note that some parameters and variables can be different from the deprecated version to the current one, so those will need to be changed. This effort may vary depending on how big is our project and the components used. Following the same example with the `Select` example:

Deprecated component:

```ts
<Select
  id="revocation-certificate-authority"
  variant="single"
  placeholderText=" "
  aria-label="Select a certificate authority for the revocation"
  aria-labelledby="revocation certificate authority"
  selections={CASelected}
  isOpen={isCAOpen}
  onToggle={(_event, isOpen: boolean) => onCAToggle(isOpen)}
  onSelect={onCASelect}
>
  {CAOptions.map((option, index) => (
    <SelectOption key={index} value={option} />
  ))}
</Select>
```

New component:

```ts
<Select
  id="revocation-certificate-authority"
  aria-label="Select a certificate authority for the revocation"
  aria-labelledby="revocation certificate authority"
  selected={CASelected}
  isOpen={isCAOpen}
  toggle={toggleCASelect}
  onSelect={onCASelect}
>
  {CAOptions.map((option, index) => (
    <SelectOption key={index} value={option}>
      {option}
    </SelectOption>
  ))}
</Select>
```

Not only the **component parameters**, but also the expected **method parameters** can change. Following with the example as a reference, the `onToggle` parameter has been transformed into `toggle` and it must be returned as a reference:

New component:

```ts
// Toggle
const toggleCASelect = (toggleRef: React.Ref<MenuToggleElement>) => (
  <MenuToggle ref={toggleRef} onClick={onCAToggle} className="pf-v5-u-w-100">
    {CASelected}
  </MenuToggle>
);
```

## Leftovers on building

At this point, most (if not all) problems should be fixed. However, I strongly recommend to build your project and address any possible errors and leftovers in the code.

As the [new FreeIPA WebUI](https://github.com/freeipa/freeipa-webui) is relying on Webpack, I just had to use the alias (`npm run start`). But if your React project uses another bundler such as [create-react-app](https://create-react-app.dev/), you can build it via `npm run build`. In any case, a double-check on the project to detect any unexpected errors won't hurt.

## Bonus: Cypress

Although this is not mentioned in the official guide, it is good to keep in mind when migrating in React projects.

If your project is using [Cypress](https://www.cypress.io/) to perform integration and/or e2e tests, you might also want to adapt the tests and replace old the class names to PatternFly 5 ones. As some of the components have changed their core structure, this will likely cause tests to fail if they are referencing deprecated structures. This only applies to the newly implemented components that we mentioned above.

E.g.: Cypress code using deprecated class names (`dropdown__toggle-text`):

```ts
 cy.get(
    "div.pf-v5-c-masthead__content button span.pf-v5-c-dropdown__toggle-text"
).then(($ele) => { ... };
```

E.g.: Cypress code using PF5 class names (`menu-toggle__text`):

```ts
cy.get(
    "div.pf-v5-c-masthead__content button span.pf-v5-c-menu-toggle__text"
).then(($ele) => { ... };
```

# Conclusions

We have introduced the main PatternFly 5 key features and offered a comprehensive guide on migrating any React project to the latest version of the library. I think the PatternFly team did (and are already doing) a great job on this major release. Looking forward to see the next features.

As this guide is base on my personal experience, some gaps and corner cases might have been not covered enough, so any comments or extra tips are more than welcome. Thanks for reading!

> Photo by <a href="https://unsplash.com/@aridley88?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Andrew Ridley</a> on <a href="https://unsplash.com/photos/a-multicolored-tile-wall-with-a-pattern-of-small-squares-jR4Zf-riEjI?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
