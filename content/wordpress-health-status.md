+++
title = "Solving WordPress Health Status for wp-load.php"
description = "Quick tip to fix WordPress health status."
date = 2021-02-07
+++

Before migrating to [Zola](https://www.getzola.org/), I started receiving a
"Should be improved" message on the Site Health Status for my blog and, after
running the health check, the critical issue was: “Some files are not writable
by WordPress: `wp-load.php`". Here is how to solve it on HostGator:

- Log into cPanel and look for File Manager;
- In the left panel, select the `public_html` folder and then look for the
`wp-load.php` file and click on it. After that, click on the “Permissions”
button at the top of the page to see the following options:

|Mode | User | Group | World |
:---: | :---: | :---: | :---: |
| **Read** | [X] | [X] | [X] |
| **Write** | [X] | [ ] | [ ] | 
| **Execute** | [ ] | [ ] | [ ] |
| **Permission** | 6 | 4 | 4 |

- In the User column, check the Write mode and click on the Change Permissions
button. Refresh your WordPress dashboard to check the Health Status. It should
show "Good" now.

I hope this helps.
