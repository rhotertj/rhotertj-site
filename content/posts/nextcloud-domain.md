---
title: "Changing the domain of your dockerized Nextcloud instance."
date: 2022-08-01T17:30:40+02:00
draft: false
---

I recently purchased a new domain and wanted to move my dockerized nextcloud installation.

At the time of writing this post, this feature was not available as stated in the [documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/domain_change.html).

The following way workaround made it possible for me, although i solely use nextcloud for file syncing. This may or may not create issues for other purposes like calender or contact syncing.

#### How to do it:

First, update the domain you set in the `.env` file in your nextcloud directory.

Make sure that your container is running since we will need to make some config changes from the inside.

Have a look at the html/config/config.php file. You may find some key-value pairs containing you old domain.

For me, this includes:

 * `trusted_domains`
 * `overwritehost`
 * `overwrite.cli.url`

Instead of changing them directly, run the following command for each of them:

```zsh
docker exec -it -u www-data nextcloud php occ config:system:set $KEY --value=example.com
```

Restart your container and you should be good to go - That's it!
