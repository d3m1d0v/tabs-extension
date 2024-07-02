# Diplodoc tabs extension

[![NPM version](https://img.shields.io/npm/v/@diplodoc/tabs-extension.svg?style=flat)](https://www.npmjs.org/package/@diplodoc/tabs-extension)

This is an extension of the Diplodoc platform, which allows adding switchable tabs in the documentation.

The extension contains some parts:
- [Prepared runtime](#prepared-runtime)
- [MarkdownIt transform plugin](#markdownit-transform-plugin)
- [Tabs API](#api)
- [React hook for smart control](#react-hook-for-smart-control)

## Quickstart

Attach the plugin to the transformer:

```js
import tabsExtension from '@diplodoc/tabs-extension';
import transform from '@diplodoc/transform';

const {result} = await transform(`
{% list tabs %}

- Tab 1
- Tab 2
- Tab 3

{% endlist %}
`, {
    plugins: [
        tabsExtension.transform({ bundle: false })
    ]
});
```

## Prepared runtime

It is necessary to add `runtime` scripts to make tabs interactive on your page.<br/>
You can add assets files which were generated by the [MarkdownIt transform plugin](#markdownit-transform-plugin).
```html
<html>
    <head>
        <!-- Read more about '_assets/tabs-extension.js' and '_assets/tabs-extension.css' in 'Transform plugin' section -->
        <script src='_assets/tabs-extension.js' async></script>
        <link rel='stylesheet' href='_assets/tabs-extension.css' />
    </head>
    <body>
        ${result.html}
    </body>
</html>
```

Or you can just include runtime's source code in your bundle.
```js
import '@diplodoc/tabs-extension/runtime'
import '@diplodoc/tabs-extension/runtime/styles.css'
```

## MarkdownIt transform plugin

Plugin for [@diplodoc/transform](https://github.com/diplodoc-platform/transform) package.

Options:
- `runtimeJsPath` - name on runtime script which will be exposed in results `script` section.<br>
  Default: `_assets/tabs-extension.js`<br>

- `runtimeCssPath` - name on runtime css file which will be exposed in results `style` section.<br>
  (Default: `_assets/tabs-extension.css`)<br>

- `bundle` - boolean flag to enable/disable copying of bundled runtime to target directory.<br>
  Where target directore is `<transformer output option>/<plugin runtime option>`<br>
  Default: `true`<br>

- `containerClasses` - additional classes which will be added to tab's container node. It allows to customize the tabs view.<br>
  Example: `my-own-class and-other-class`<br>
  Default: `undefined`<br>

## API

### Syntax

You can synchronize the opening of tabs between different tabs groups on the page. To do this, you just need to add optional property `group=<group_key>` in `list tab` command. The active tabs with the same keys in one tabs group will be synchronized.

Example:
```
{% list tabs group=group_1 %}

- Tab 1
- Tab 2
- Tab 3

{% endlist %}

{% list tabs group=group_1 %}

- Tab 1
- Tab 2
- Tab 3

{% endlist %}
```

Additionally, you can use radiobatons using a contruction

```
{% list tabs radio %}

  - Tab 1

    Text 1.

    * You can use list
    * And **other** features.

  - Tab 2

    Text 2.

{% endlist %}
```

The keys for the tabs are generated automatically. They are based on the tab's names using the github anchors style.

You can set your own keys for tabs with this statement:
```
{% list tabs group=group_1 %}

- Tab 1 {#my-tab-1}
- Tab 2 {#my-tab-2}

{% endlist %}
```

## React hook for smart control

You can use the React hook to handle active tab changing or to select opened tab programmatically.

```TypeScript
import React, { useEffect } from 'react'
import {UseDiplodocTabsCallback, useDiplodocTabs, Tab} from '@diplodoc/tabs-extension/react';

export const App: React.FC = () => {
    const selectTabHandler = useCallback<UseDiplodocTabsCallback>(
        (tab: Tab, currentTabId?: string) => {
            const {group, key} = tab;
            // Group could be empty
            if (group) {
                // ...
            }
        },
        [],
    );

    const {selectTab, selectTabById} = useDiplodocTabs(selectTabHandler);

    useEffect(() => {
        selectTab({ group: 'group_1', key: 'my-key' });
        // selectTabById('my-key-2');
    }, [selectTab, selectTabById]);

}
```