---
title: Visual Studio Code
permalink: "/digital-garden/vs-code/"
layout: topic
---

## Prettier
### Setup Prettier:

* Go to Extensions and download Prettier plugin
* Create a `.prettierrc.js` file and add custom rules

```
module.exports = {
  trailingComma: "es5",
  tabWidth: 4,
  semi: false,
  singleQuote: true,
};
```
* Go to `code>preferences>settings` or `command + ,`
* Set the `Default Formatter` to Prettier
* Check `Format On Save`
