---
title: jQuery animate scrollTop not working in IE 7
tags: [javascript jquery]
date: 2016/04/05
---

The following works in Chrome / FF etc...

```
$('body').animate({scrollTop : 0}, 0);
```

However, in IE 7, it doesn't do anything.

Is there an alternative?

**Answers**

in IE8， i use ```$(document).scrollTop()``` to get the scrollTop property, ```$('body').scrollTop()``` or ```$('html').scrollTop()``` will always return 0.

Maybe you can use

```
$(document).animate({scrollTop: 0}, 0);
$('html,body').animate({scrollTop: 0}, 0);
```

to make it works on all browser.