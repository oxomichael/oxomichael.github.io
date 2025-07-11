---
translationKey: "2017-07-14-customize-symfony-for-development-environment"
categories: symfony php development
date: "2017-07-14T11:45:00Z"
ref: 2017-07-14-customize-symfony-for-development-environment
title: Customize Symfony for development environment
---

When you use a development environment similar to the production, you can't use the `app_dev.php` as given.
- With vagrant your front IP could be 192.168.33.10
- With docker you can define a front IP as 172.10.0.1

So open `app_dev.php` and change the following code

{{< highlight php >}}
if (isset($_SERVER['HTTP_CLIENT_IP'])
    || isset($_SERVER['HTTP_X_FORWARDED_FOR'])
    || !(in_array(@$_SERVER['REMOTE_ADDR'], ['127.0.0.1', '::1', '192.168.33.10', '172.10.0.1'], true)
    || PHP_SAPI === 'cli-server')
) {
    header('HTTP/1.0 403 Forbidden');
    exit('You are not allowed to access this file. Check '.basename(__FILE__).' for more information.');
}
{{< / highlight >}}

**Keep in mind, always use private IP and don't include this file during deployment to your production**
