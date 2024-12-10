```console
# pkg install vm-bhyve grub2-bhyve bhyve-firmware
# sysrc vm_enable="YES"
# zfs create zroot/vm
# sysrc vm_dir="zfs:zroot/vm"
# vm init
# cp /usr/local/share/examples/vm-bhyve/centos7.conf /zroot/vm/.templates/
# cd /zroot/vm/.templates/
# cp centos7.conf overleaf.conf
# vi overleaf.conf 
```

```conf
loader="uefi"
graphics="yes"
xhci_mouse="yes"
cpu=12
memory=16384M
network0_type="virtio-net"
network0_switch="public"
disk0_type="virtio-blk"
disk0_name="overleaf-vio0.img"
```

```console
# vm switch create public
# vm switch add public nfe0
# vm switch address public 10.0.0.1/24
# vm iso https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.7.0-amd64-netinst.iso
/zroot/vm/.iso/debian-12.7.0-amd64-netinst.iso         631 MB   27 MBps    23s
# vm create -t debian -s 150G overleaf
# vm list
NAME      DATASTORE  LOADER  CPU  MEMORY  VNC  AUTO  STATE
overleaf  default    grub    12   24576M  -    No    Stopped
# 
```

overleaf/the_usual

```
 
                               ,`""',
                              _;____;_
                              \__/\__/
                          ,,,  ;`,',;
                         ;,` ; ;' ` ;   ,',
                         ;`,'; ;`,',;  ;,' ;
                         ;',`; ;` ' ; ;`'`';
                         ;` '',''` `,',`',;
                          `''`'; ', ;`'`'
                               ;' `';
                               ; ',';
                               ;,' ';
                           dwb\|/\|/;\|/jm
          
Welcome to JACKTUS, a battered Sun X4440 running FreeBSD!
For inquiries: mailto:j.maltagliati@campus.unimib.it or look for Jacopo in room
1049 (1st floor, end of right wing, bldg U14).

If you're tired of seeing this cactus, touch ~/.hushlogin



```console
# cat /etc/pf.conf 
master="nfe0"
vm_subnet="10.0.0.0/24"
overleaf_vm_addr="10.0.0.2"

nat on $master from {$vm_subnet} to any -> ($master)

rdr pass proto tcp from any to any port http -> $overleaf_vm_addr port http
rdr pass proto tcp from any to any port https -> $overleaf_vm_addr port https
```

## Overleaf on Debian

Trying to follow the quickstart guide for Overleaf-CE will fail in a non-obvious way, until you look at the logs of the Redis container: it turns out that Redis 5.0+ needs the AVX extensions, and my ancient Opterons don't have it. Luckily I'm not the only ~~idiot~~ chap who wishes to run Overleaf on old junk, so [someone solved this problem already](https://github.com/overleaf/toolkit/issues/216).

I was able to download the `docker-compose.yml` and use it without issues, but I had to add a network so I could access the external package archives.

```yaml
services:
  sharelatex:
    restart: always
    image: sharelatex/sharelatex:4.2.8
    container_name: sharelatex
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_started
    ports:
      - 80:80 #ext:cont
    links:
      - mongo
      - redis
    stop_grace_period: 60s
    volumes:
      - ~/overleaf/sharelatex:/var/lib/sharelatex
    environment:
      SHARELATEX_APP_NAME: Overleaf 4.2.8 on JACKTUS
      SHARELATEX_MONGO_URL: mongodb://mongo/sharelatex
      SHARELATEX_REDIS_HOST: redis
      REDIS_HOST: redis
      ENABLED_LINKED_FILE_TYPES: "project_file,project_output_file"
      ENABLE_CONVERSIONS: "true"
      EMAIL_CONFIRMATION_DISABLED: "true"
      # temporary fix for LuaLaTex compiles
      # see https://github.com/overleaf/overleaf/issues/695
      TEXMFVAR: /var/lib/sharelatex/tmp/texmf-var
    networks:
      overleaf-network

  mongo:
      restart: always
      image: mongo:4.4
      container_name: mongo
      command: "--replSet overleaf"
      expose:
        - 27017
      volumes:
        - ~/overleaf/mongo:/data/db
      healthcheck:
        test: echo 'db.stats().ok' | mongo localhost:27017/test --quiet
        interval: 10s
        timeout: 10s
        retries: 5

  redis:
      restart: always
      image: redis:6.2
      container_name: redis
      expose:
        - 6379
      volumes:
        - ~/overleaf/redis:/data

networks:
  overleaf-network:
    driver: bridge
    external: true
```


```console
# docker compose up -d
# docker exec mongo mongo --eval "rs.initiate({ _id: \"overleaf\", members: [ { _id: 0, host: \"mongo:27017\" } ] })"
```

Now Overleaf will start and you should be able to create a new project. However, the TeXLive package archive will not work. This is because the TeXLive version is outdated and we need to switch to a different repository:

```console
# docker exec overleaf bash
overleaf# tlmgr option repository https://ftp.tu-chemnitz.de/pub/tug/historic/systems/texlive/2023/tlnet-final/
overleaf# tlmgr update --self
overleaf# tlmgr update --all
```

```
1:M 16 Oct 2024 12:16:01.638 # WARNING Memory overcommit must be enabled! Without it, a background save or rep
lication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, s
ee https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf an
d then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.

```





