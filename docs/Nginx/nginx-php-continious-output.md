---
id: 205
title: Nginx + PHP continious output
type: post
date: 2020-03-23 07:22:17
lastmod: 2023-04-01 23:58:49
categories:
- PHP
---


Some times we requrire the page to display partial content, such as progress while the PHP is still working in background. In apache, disabling output buffering works fine, however for nginx, the following code need to be added on top of any PHP script (before output stats).





```php
header('X-Accel-Buffering: no');
ob_implicit_flush(1);

```


