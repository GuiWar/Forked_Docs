<!-- Do not edit this file. It is automatically generated by API Documenter. -->

[Home](./index.md) &gt; [@genesislcap/foundation-layout](./foundation-layout.md) &gt; [FoundationLayout](./foundation-layout.foundationlayout.md) &gt; [registeredItems](./foundation-layout.foundationlayout.registereditems.md)

## FoundationLayout.registeredItems() method

Gets all of the currently registered names

**Signature:**

```typescript
registeredItems(): string[];
```
**Returns:**

string\[\]

string\[\] - an item for each registered element currently registered with the layout system

## Remarks

You can use this with [layoutRequiredRegistrations](./foundation-layout.foundationlayout.layoutrequiredregistrations.md) to work out what items you need to register with [registerItem()](./foundation-layout.foundationlayout.registeritem.md) before loading the layout with [loadLayout()](./foundation-layout.foundationlayout.loadlayout.md)

