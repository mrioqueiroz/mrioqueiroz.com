+++
title = "Enabling monitoring and some security"
description = "Intrusion Detection and Prevention Systems with SSHGuard and Fail2ban, and monitoring with Grafana and Prometheus."
date = 2021-05-23
+++

# Intrusion detection

It still impress me how just after a few minutes after having a server up we
start to get a lot of SSH connection attempts.

Here is an summary of just a few
seconds of `systemctl --follow --unit sshd` to demonstrate how bad this is:

```
May 22 23:10:15 sshd[..]: error: PAM: Authentication failure for root from 62.178.173.4
May 22 23:10:15 sshd[..]: Failed keyboard-interactive/pam for root from 62.178.173.4 port 40639 ssh2
May 22 23:10:16 sshd[..]: error: PAM: Authentication failure for root from 62.178.173.4
May 22 23:10:16 sshd[..]: Failed keyboard-interactive/pam for root from 62.178.173.4 port 40639 ssh2
May 22 23:10:16 sshd[..]: error: PAM: Authentication failure for root from 62.178.173.4
May 22 23:10:16 sshd[..]: Failed keyboard-interactive/pam for root from 62.178.173.4 port 40639 ssh2
May 22 23:10:16 sshd[..]: Received disconnect from 62.178.173.4 port 40639:11:  [preauth]
May 22 23:10:16 sshd[..]: Disconnected from authenticating user root 62.178.173.4 port 40639 [preauth]
May 22 23:10:32 sshd[..]: Connection from 62.178.173.4 port 5201. on 10.0.0.4 port 22 rdomain ""
May 22 23:10:35 sshd[..]: error: PAM: Authentication failure for root from 62.178.173.4
May 22 23:10:35 sshd[..]: Failed keyboard-interactive/pam for root from 62.178.173.4 port 5201. ssh2
May 22 23:10:35 sshd[..]: error: PAM: Authentication failure for root from 62.178.173.4
May 22 23:10:35 sshd[..]: Failed keyboard-interactive/pam for root from 62.178.173.4 port 5201. ssh2
May 22 23:10:35 sshd[..]: error: PAM: Authentication failure for root from 62.178.173.4
May 22 23:10:35 sshd[..]: Failed keyboard-interactive/pam for root from 62.178.173.4 port 5201. ssh2
May 22 23:10:35 sshd[..]: Received disconnect from 62.178.173.4 port 5201.:11:  [preauth]
May 22 23:10:35 sshd[..]: Disconnected from authenticating user root 62.178.173.4 port 5201. [preauth]
```

And the logs for this service grows extremely fast...

As an attempt to mitigate this problem, first I enabled the
[`sshguard`](https://sshguard.net/) service (a tool that *protects hosts from
brute-force attacks against SSH and other services. It aggregates system logs
and blocks repeat offenders using one of several firewall backends*).

To do it in NixOS, it is just needed to add this line to `configuration.nix`
file (or deployment file if using NixOps):

```nix
{ services.sshguard.enable = true; }
```

To make it a little more restrictive in this server, this is what I put on
`deployment.nix`:

```nix
{
  services = {
    sshguard = {
      enable = true;
      blocktime = 900;
      attack_threshold = 5;
      blacklist_threshold = 10;
    };
  };
}
```

Here is a summary of `systemctl --follow --unit sshguard`:

```
May 22 23:34:12 sshguard[..]: Blocking "62.178.173.4/32" for 960 secs (3 attacks in 0 secs, after 4 abuses over 1138 secs.)
May 22 23:34:16 sshguard[..]: Attack from "222.186.180.130" on service SSH with danger 2.
May 22 23:38:25 sshguard[..]: Attack from "62.178.173.4" on service SSH with danger 10.
May 22 23:38:25 sshguard[..]: Blocking "62.178.173.4/32" for 900 secs (1 attacks in 0 secs, after 1 abuses over 0 secs.)
May 22 23:39:30 sshguard[..]: Attack from "222.187.238.136" on service SSH with danger 2.
May 22 23:42:00 sshguard[..]: Attack from "185.36.81.186" on service SSH with danger 10.
May 22 23:42:00 sshguard[..]: Blocking "185.36.81.186/32" for 900 secs (1 attacks in 0 secs, after 1 abuses over 0 secs.)
May 22 23:46:00 sshguard[..]: Attack from "62.178.173.4" on service SSH with danger 10.
May 22 23:46:00 sshguard[..]: Blocking "62.178.173.4/32" forever (1 attacks in 0 secs, after 1 abuses over 0 secs.)
May 22 23:49:59 sshguard[..]: Attack from "201.141.43.13" on service SSH with danger 10.
May 22 23:49:59 sshguard[..]: Blocking "201.141.43.13/32" forever (1 attacks in 0 secs, after 1 abuses over 0 secs.)
May 22 23:54:20 sshguard[..]: Attack from "221.182.185.220" on service SSH with danger 10.
May 22 23:54:20 sshguard[..]: Blocking "221.182.185.220/32" forever (1 attacks in 0 secs, after 1 abuses over 0 secs.)
May 22 23:57:28 sshguard[..]: Attack from "222.186.42.7" on service SSH with danger 10.
May 22 23:57:28 sshguard[..]: Blocking "222.186.42.7/32" forever (1 attacks in 0 secs, after 1 abuses over 0 secs.)
May 23 00:08:24 sshguard[..]: Attack from "201.141.45.88" on service SSH with danger 10.
May 23 00:08:24 sshguard[..]: Blocking "201.141.45.88/32" forever (1 attacks in 0 secs, after 1 abuses over 0 secs.)
May 23 00:27:52 sshguard[..]: Attack from "221.182.185.222." on service SSH with danger 10.
May 23 00:27:52 sshguard[..]: Blocking "221.182.185.222./32" forever (1 attacks in 0 secs, after 1 abuses over 0 secs.)
May 23 00:40:51 sshguard[..]: Attack from "222.186.42.137" on service SSH with danger 2.
```

After that, I enabled the [`fail2ban`](https://www.fail2ban.org) service too,
which is an Intrusion Prevention System that *scans log files and bans IPs that
show the malicious signs - too many password failures, seeking for exploits,
etc.*

As I still did not configure it properly and `sshguard` already blocked a
lot of IPs, it did not get many results yet. `journalctl --follow --unit
fail2ban` shows:

```
May 23 00:57:42 .fail2ban-serve[..]: fail2ban.filter [..]: INFO [sshd] Found 168.72.0.205 - 2021-05-23 00:55:10
May 23 01:50:22 .fail2ban-serve[..]: fail2ban.filter [..]: INFO [sshd] Found 63.232.71.116 - 2021-05-23 01:50:22
May 23 09:08:48 .fail2ban-serve[..]: fail2ban.actions [..]: NOTICE [sshd] Ban 186.220.102.246
May 23 09:18:07 .fail2ban-serve[..]: fail2ban.actions [..]: NOTICE [sshd] Unban 186.220.102.241
May 23 09:18:09 .fail2ban-serve[..]: fail2ban.actions [..]: NOTICE [sshd] Unban 186.220.101.211
May 23 09:18:41 .fail2ban-serve[..]: fail2ban.actions [..]: NOTICE [sshd] Unban 198.144.121.93
May 23 09:18:47 .fail2ban-serve[..]: fail2ban.actions [..]: NOTICE [sshd] Unban 186.220.102.253
May 23 09:18:49 .fail2ban-serve[..]: fail2ban.actions [..]: NOTICE [sshd] Unban 186.220.102.246
```

I can say that after enabling just these two services the amount of login
attempt events reduced a lot, and the list of IPs being blocked keeps growing.

For the next days I plan to try other similar tools and see how the server
security can be improved even more. Maybe add alerts and create a dashboard on
Grafana focused on security.

# Monitoring with Prometheus and Grafana

After reading some good things about Prometheus and Grafana, I decided to give
them a try.

To configure this initial monitoring solution, I basically followed [this
video](https://www.youtube.com/watch?v=4WWW2ZLEg74) and adapted the same
settings to the Nix language. All the
[options](https://search.nixos.org/options) are already there. It was just a
matter of adjust them to the language syntax.

The first service I enabled was Prometheus. The initial settings were as
follows:

```nix
{
  prometheus = {
    enable = true;
    exporters = { node = { enable = true; enabledCollectors = [ "systemd" ]; }; };
    scrapeConfigs = [
      { job_name = "prometheus"; static_configs = [ { targets = [ "localhost:9090" ]; } ]; }
      { job_name = "node"; static_configs = [ { targets = [ "localhost:9100" ]; } ]; }
    ];
  };
}
```

Having the Prometheus service up and running, to make Grafana works was a
matter of adding these lines:

```nix
{
  grafana = {
    enable = true;
    provision = {
      enable = true;
      datasources = [
        {
          name = "Prometheus";
          type = "prometheus";
          url = "http://localhost:9090";
          access = "proxy";
        }
      ];
    };
  };
}
```

A problem I had that I still did not find a way to solve was the initial
`adminUser` and `adminPassword` that I tried to set via deployment: when I
accessed the Grafana page I had to use the default credentials for "admin". The
credentials I set on `deployment.nix` were not deployed(?). I may have done
something wrong, but I will figure it out later.

It is important to note that, if setting the credentials this way, the password
will be stored as plain text in the Nix store. So this is another thing I will
need to solve.

Finally, to expose it all to the outside world I added a new record to
Cloudflare and, here is the Nginx configuration:

```nix
{
  nginx = {
    enable = true;
    virtualHosts = {
      "grafana.mrioqueiroz.com" = {
        locations."/".proxyPass = "http://localhost:3000";
        addSSL = true;
        enableACME = true;
      };
    };
  };
}
```

## Making it all work

To deploy all this configuration to the server, this is all it takes:

```bash
nix-shell
nixops create ./deployment.nix --deployment blog
nixops deploy --deployment blog
```

This builds the configuration locally, sends it to the server and activates the
profile. As simple as that. Without the need to SSH directly into the machine
and change these settings manually on a lot of different files.

# Final notes

- To finish the day, I got a new domain that will redirect to this blog. So now
when I find myself too lazy to type the full URL, I can just use
[mrio.dev](https://mrio.dev);
- In this post, I'm not using the actual IPs that were sending brute-force
attacks to the server;
- You can find the version of the code for this deployment
[here](https://github.com/mrioqueiroz/nix-hosts/tree/4c01d5145d95a0fc711ee2d3fd4ea960c90decef);
- Grafana is available [here](https://grafana.mrioqueiroz.com). I still need to
make some dashboard public, but this is another project.
