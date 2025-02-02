+++
publishDate = 2025-02-02
title = "Conquer Issues with Pasta in Rootless Docker"
description = "Docker Rootless Networking with Pasta Network Driver"
tags = ["containers", "docker", "rootless", "pasta", "systemd"]
categories = ["tools", "security", "networking"]
series = ["docker"]
images = ["/images/shipping.jpg"]
+++

In front of geopolitical tensions and increasing risk of cyber-attacks, I am making efforts to
gain more and more hard skills in the area of cyber security.

Some time ago I switched from running docker daemon as root to so called [rootless containers](https://github.com/rootless-containers/rootlesskit). It allows to run docker daemon as regular user with reduced privileges. The whole process is very well described in the [docker documentation](https://docs.docker.com/engine/security/rootless/). However there are some limitations and trade-offs.

One important aspect which still needs improvement is networking, especially in terms of network performance. There are several network drivers for rootless docker which help to mitigate the problem. I decided to test [pasta](https://passt.top/) driver.

Even if Pasta is still in experimental phase, it works pretty well. However, in my case I had to solve one issue. In this article I want to describe a problem and present the solution I came up with. 

{{< figure alt="Shipping - cargo ship full of containers in a dock" src="/images/shipping.jpg" caption="Docker containers" >}}

## Problem Statement

What I observed, after setting up rootless docker in my Debian and Ubuntu systems, is problem during system boot. The docker daemon, configured for a user, didn't start.

When I checked the service status with command:
```shell
systemctl --user status docker.service
```

I saw this error message:
```
docker.service: Start request repeated too quickly.
docker.service: Failed with result 'exit-code'.
```

After digging in the system logs with:
```shell
journalctl --user -xeu docker.service
```

I've noticed that `pasta` gave up:
```
Jan 17 07:50:06 jupiter pasta[1223]: No routable interface for IPv4: IPv4 is disabled
Jan 17 07:50:06 jupiter pasta[1223]: External interface not usable
```

It made me think that the network was not ready when pasta was starting.

## Failed Attempts

My first idea was to make sure that network is ready and only then trigger `docker.service`. I read about `network-online.target` on [systemd.io](https://systemd.io/NETWORK_ONLINE/) and it was promising.

Unfortunately, setting dependency between `network-online.target` and `docker.service` didn't help. Apparently this approach is only valid for system services and don't work with user services.

## Working Solution

So what I did instead? I went back to the original error message telling me that 'Start request repeated too quickly'. It seemed like `systemd` was trying to restart docker several times, but it eventually gave up.

The answer was to just restart `docker.service`, but not 'too quickly'. I've changed the service definition under `~/.config/systemd/user/docker.service`. I added following statements under `[Unit]` section:
```
[Unit]
StartLimitBurst=3
StartLimitIntervalSec=60
```

And extended the time between restart attempts:
```
[Service]
TimeoutSec=0
RestartSec=10
```

Also I removed unnecessary settings from `[Service]` section (they should be in the `[Unit]` section according to the [manual](https://www.man7.org/linux/man-pages/man5/systemd.unit.5.html)):
```
[Service]
StartLimitBurst=3
StartLimitInterval=60
```

## Alternatives

Another solution might be use of `ExecStartPre` option inside `docker.service` definition. For example:
```
[Service]
ExecStartPre=sh -c 'until ping -c 1 pawelweselak.com; do sleep 1; done'
```

Although, I haven't tested it yet. I'm OK with restarting docker once during boot to mitigate the problem.

## Bonus

I forgot to mention that I had one more issue with Pasta. It was specific to Debian distro. In this case `pasta` was blocked by `apparmor`, because of missing `apparmor` profile specific for `pasta` executable.

To solve that I changed symlinks to hardlinks:
```shell
cd /usr/bin
sudo rm pasta pasta.avx2
sudo ln passt pasta
sudo ln passt.avx2 pasta.avx2
```

And restarted `apparmor`:
```shell
sudo aa-teardown
sudo systemctl restart apparmor
```

## Additional resources
- [List of available network drivers for Docker](https://docs.docker.com/engine/security/rootless/#networking-errors)
- [RootlessKit Network Drivers documentation](https://github.com/rootless-containers/rootlesskit/blob/master/docs/network.md)
- [`bypass4netns` driver](https://github.com/rootless-containers/bypass4netns)