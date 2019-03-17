---
layout: detail
title: Riot FAQ
description: Questions and Answers about Riot, User Interface (UI) Micro-Libary
---

## Frequently asked questions

### Why is this project called Riot?
Riot is against the current trend of boilerplate and unneeded complexity. We think that a small, powerful API and concise syntax are extremely important things on a client-side library.

### Is Riot free?
Riot is free, open source and licensed under the MIT License. There are no [Additional Grant of Patent Rights](https://github.com/facebook/react/blob/master/PATENTS).

### Can I use Riot in production?
Absolutely. It's a mature framework [embraced daily](https://twitter.com/search?q=riotjs).

### Which browsers are supported?

Riot.js 4 supports all the major modern browsers. Browsers like IE11 are not supported: if you need to support such old browsers you might consider using an [older Riot version](/v3)

### Should I use dash on the tag name?
W3C specification demands you use a dash in the tag name. Instead of `<person>` you must write `<my-person>`. Obey this rule if you care about W3C. Both work fine.

### Why are there no semicolons in the source code?
Leaving out semicolons makes the code less crowded. This is aligned with our general minimalistic approach. We use single quotes for the same reason. If you contribute to Riot, please leave out semicolons and double quotes.

### Can I use `style` tags in a .riot file?
Yes. You can use CSS normally inside a riot tag file. The web component standard also has a mechanism of encapsulating of CSS. However, it's unlikely that this improves the overall manageability of your CSS.

### Isn't `onclick` evil?
It's not evil, it just looks "old". To have JS and HTML under the same module is more important than aesthetics. The minimal Riot syntax makes event handlers look decent.

### Any future plans?

Sure. We mostly focus on [stability and performance](https://github.com/riot/riot/issues) and try to provide more [examples](https://github.com/riot/examples).
