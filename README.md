SailfishOS Android Signature Spoofing
===

This is a compiled set of instructions and tools wrapped in Docker image to fetch, deodex, patch and upload back 
AlienDalvik files on Sailfish phones.

Most of the existing tools assume `adb` as transport. In Sailfish it's a bit tricky so I replaced it with Rsync. 
Please not that Rsync is run in completely insecure manner, so don't leave it running in public unprotected networks.

Overview of the steps performed by the scripts:
 * fetch via rsync `/opt/alien/system/{framework,app,priv-app}`
 * deodex using [simple-deodexer](https://github.com/aureljared/simple-deodexer)
 * apply `hook` and `core` patches from [haystack](https://github.com/Lanchon/haystack)
 * push back changed files, saving backups in `/opt/alien/system/{framework,app,priv-app}.pre_haystack`

Instructions
===

**Starting Rsync daemon on Sailfish**

* Make sure Android subsystem is stopped
* Make sure you PC is in connected to the same WiFi network as your phone
* Figure out your phone's IP address. It's shown in "Developer mode". We will use it later
* Enable [developer mode](https://jolla.zendesk.com/hc/en-us/articles/202011863-How-to-enable-Developer-Mode)
* Open terminal app or connect via SSH
* Become root by executing `devel-su`
* Create minimalistic Rsync config

```bash
echo > /home/nemo/rsyncd-alien.conf << 'EOF'
[alien]
 path=/opt/alien
 readonly=false
 uid=root
 gid=root 
EOF
```

* run daemon in foreground with logging

```bash
rsync --daemon --no-detach --verbose --config=/home/nemo/rsyncd-alien.conf --log-file=/dev/stdout
```

**Build and execute docker image**

Checkout this repo. Make sure docker is available on you machine and running
* https://www.docker.com/docker-windows
* https://www.docker.com/docker-mac

```bash
docker build -t haystack . && docker run --env SAILFISH=<PHONE_IP_ADDRESS> haystack
```

**Final steps**
* kill running rsync by pressing Ctrl-C
* start Android subsystem (or just run some app). *This will take time, depending on number of apps you have*
* From that point you can install [microG](https://microg.org/download.html) (nightly) [F-Droid](https://f-droid.org). 
Don't forget to enable "Unstable updates" from "Expert mode"


Reverting the changes (if needed)
===
```bash
cd /opt/alien/system
cp -r --reply=yes -v framework.pre_haystack/* framework/
cp -r --reply=yes -v app.pre_haystack/* app/
cp -r --reply=yes -v priv-app.pre_haystack/* priv-app/

cd /opt/alien/system_jolla
cp -r --reply=yes -v framework.pre_haystack/* framework/
cp -r --reply=yes -v app.pre_haystack/* app/
cp -r --reply=yes -v priv-app.pre_haystack/* priv-app/
```


