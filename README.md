
# google-chrome in docker

*Need to test some latest Chrome version's features?* but hestitant to
upgrade your main browser to unstable? this chrome-in-docker project can help you

## Features

- It downloads a google-chrome Linux version from chrome channels, either
  stable, or beta, or developer version; install and pack into a docker
  container, that can run on anywhere you have docker daemon;
  https://www.chromium.org/getting-involved/dev-channel#TOC-Linux

- It turns google-chrome into a headless browser, can be used together
  with Selenium with chrome webdriver, or with Chrome's native Remote
  Debugging Protocol you can program with
  https://developer.chrome.com/devtools/docs/debugger-protocol
  https://github.com/cyrus-and/chrome-remote-interface
  that makes it a better headless browser than PhantomJS or SlimerJS,
  better programability in my opinion;
  while if need debugging, you have a VNC session to see the actual browser,
  and do whatever you want, or you can even use it as your everyday main browser.

# Usage

You may either just pull my prebuilt docker image at https://hub.docker.com/r/liuzheng712/chrome-in-docker/

    $ docker pull liuzheng712/chrome-in-docker
    $ docker run -it --rm liuzheng712/chrome-in-docker google-chrome --version
    Google Chrome 70.0.3538.67

Or build it locally with Dockerfile here

    $ docker build -t chrome:20181018 .

Check what Chrome version is builtin, and tag a version:

    $ docker run -it --rm chrome:20181018 google-chrome --version
    Google Chrome 70.0.3538.67
    $ docker tag chrome:20181018 chrome:70.0.3538.67

The extra `get-latest-chrome.sh` script here is to get latest versions of
Chrome Stable, Beta, or Unstable version, for testing some latest features,
here you may modify the Dockerfile to build a different image with each one,
while, since the beta and unstable versions are changing fast, may be updating
every week or every day, you don't have to rebuild docker images everyday,
with this `get-latest-chrome.sh` and local volume bind, you can run a different
container with the same image; that way, within a relatively longer time range
you don't have to rebuild the base docker image; the reasons of a same base image
can be reused is dependencies of the different channels (stable, beta, or -dev)
are most probably the same, or changing much less often; anyway, if there is
any problem that stable can run but unstable cannot, you may always have a no-cache
rebuild: by `docker build --pull --no-cache ...` to force pull latest ubuntu base
and latest Chrome binary packages.

    $ ./get-latest-chrome.sh
    [... downloading latest Chrome and extracting to ./opt ...]

You may test run it one time first to check what's exact version of each Chrome channel:

    $ docker run -it --rm -v $PWD/opt:/opt:ro chrome:20181018 \
                             /opt/google/chrome-unstable/google-chrome-unstable --version
    Google Chrome 71.0.3578.10 dev

    $ docker run -it --rm -v $PWD/opt:/opt:ro chrome:20181018 \
                             /opt/google/chrome-beta/google-chrome-beta --version
    Google Chrome 70.0.3538.67 beta

    $ docker run -it --rm -v $PWD/opt:/opt:ro chrome:20181018 \
                             /opt/google/chrome/google-chrome --version
    Google Chrome 70.0.3538.67

Then run 3 different containers with the same base docker image:

```console
$ docker run -dt --privileged \
             --name Chrome-dev-71.0.3578.10 \
             -h chrome-dev-71.local \
             -v $PWD/opt:/opt:ro \
             -e CHROME=/opt/google/chrome-unstable/google-chrome-unstable \
             -p 9221:9222 \
         chrome:20181018
56417156ffea4a55642cfa59cf5e9758a2be144144b2df39e91aa9265f098b75
$ docker run -dt --privileged \
             --name Chrome-beta-70.0.3538.67 \
             -h chrome-beta-70.local \
             -v $PWD/opt:/opt:ro \
             -e CHROME=/opt/google/chrome-beta/google-chrome-beta \
             -p 9222:9222 \
         chrome:20181018
d5b784cbe9ac7d3a52b43c7fb6918b28366c8b939293b10fb9b1808de7b46e2e
$ docker run -dt --privileged \
             --name Chrome-stable-70.0.3538.67 \
             -h chrome-beta-70.local \
             -v $PWD/opt:/opt:ro \
             -p 9223:9222 \
             chrome:20181018
35974a5247cf8650da25d03d9f279749ae4cf1e5b0c57349af1d511b8ac99545

$ docker ps -a
CONTAINER ID  IMAGE            COMMAND      CREATED  STATUS  PORTS  NAMES
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
f6e8d2d311e0        chrome:20181018     "/entry.sh"         5 seconds ago       Up 3 seconds        0.0.0.0:9223->9222/tcp   Chrome-stable-70.0.3538.67
7574033cb9fd        chrome:20181018     "/entry.sh"         12 seconds ago      Up 11 seconds       0.0.0.0:9222->9222/tcp   Chrome-beta-70.0.3538.67
8602d8f59014        chrome:20181018     "/entry.sh"         18 seconds ago      Up 17 seconds       0.0.0.0:9221->9222/tcp   Chrome-dev-71.0.3578.10

$ curl -s localhost:9221/json/version
{
   "Browser": "Chrome/71.0.3578.10",
   "Protocol-Version": "1.3",
   "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.10 Safari/537.36",
   "V8-Version": "7.1.302.3",
   "WebKit-Version": "537.36 (@a04d4e218f0813212aa325e038e8961b62fd40ff)",
   "webSocketDebuggerUrl": "ws://localhost:9221/devtools/browser/5ff8258a-3cdb-46f1-b22e-f5d9c70ca156"
} 
$ curl -s localhost:9222/json/version
{
   "Browser": "Chrome/70.0.3538.67",
   "Protocol-Version": "1.3",
   "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.67 Safari/537.36",
   "V8-Version": "7.0.276.28",
   "WebKit-Version": "537.36 (@9ab0cfab84ded083718d3a4ff830726efd38869f)",
   "webSocketDebuggerUrl": "ws://localhost:9222/devtools/browser/a49fd215-8555-4d57-b69c-ac6c9ca27009"
}
$ curl -s localhost:9223/json/version
{
   "Browser": "Chrome/70.0.3538.67",
   "Protocol-Version": "1.3",
   "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.67 Safari/537.36",
   "V8-Version": "7.0.276.28",
   "WebKit-Version": "537.36 (@9ab0cfab84ded083718d3a4ff830726efd38869f)",
   "webSocketDebuggerUrl": "ws://localhost:9223/devtools/browser/ef2d5cae-d236-4369-a500-a919ce6cd45c"
}
```

You may try https://github.com/cyrus-and/chrome-har-capturer with more har capturing commands
like `chrome-har-capturer -t 172.18.0.2 urls...`

## Debugging

VNC session listens default on the container's 5900 port, if you figured out the container's
IP address (by above inspect command), an VNC session can be opened by your favorite
VNC client connect to this ip address, or you may use another `-p localport:5900`
to set up another port forwarding to be able to use it from a 3rd computer.

## Env variables to customize

1. the default VNC password is `hola`; you may pass additional env var to docker run
   by `-e VNC_PASSWORD=xxx` to change to use a different VNC password;
2. the default CHROME is `/opt/google/chrome/google-chrome`, if you use local
   volume bind to have different chrome versions, you may pass additional env var
   by `-e CHROME=/path/to/chrome or chromium`

# Design

## Docker Image Build Time

1. The Dockerfile defined process of where as start, it's starting from latest

2. Ubuntu as base image, then install VNC and some network utilties like curl and socat,
xvfb, x11vnc as Graphic layer for Chrome graphical output, xterm as debugging term window
supervisor as processes manager, sudo also for debugging, not technically required.

3. Then add Google-Chrome's apt source and install google-chrome-stable version,
and it will handle all runtime dependencies by Chrome;
This static version will be packed as part of the docker image, when you're not
using local volume bind, this version will be used. It depends how often do you
rebuild, but with above `./get-latest-chrome.sh` script, you don't have to rebuild
very often.

3. Then add a regular user at 1000:100 for improved security and run all services
under this regular user; sudo can be used for debugging.
Copying supervisord.conf as definition of process structure; and entry.sh as
container entrypoint.

## Container Spawn
At container spawn time (`docker run ...`), it starts from the entrypoint `entry.sh`
there it handles default VNC password `hola`, and check CHROME environment,
set it default to the stable version `/opt/google/chrome/google-chrome`;

Then it exec to supervisord to spawn more processes defined in `supervisord.conf`

## Process Management

Supervisord is the process manager, it spawns 4 processes:

1. Xvfb ... as X server
2. x11vnc ... as VNC on top of X Server
3. fluxbox as window manager, this is technically not required,
   any X11 application can directly run on X server, but with a window
   manager, it's easier for debugging, when need to move window, resize,
   maximize, and minimize, etc.
4. xterm, same for debugging
5. start chrome from CHROME environment variable, with `--remote-debugging-port=19222`
   to enable Remote Debugging Protocol
4. socat, as a forwarding channel, chrome can only listen on local loopback
   interface (127.0.0.1); hence not accepting any request from outside
   so a tcp forwarding tool like socat is necessary here.

Supervisord will respawn any managed processes if it crashed.

Ideally here should define dependencies between the processes, but due to
https://github.com/Supervisor/supervisor/issues/122 it lacks such feature.

# Some further improvements

- [ ] Chromium nightly https://download-chromium.appspot.com/
- [ ] VNC in browser, see https://github.com/fcwu/docker-ubuntu-vnc-desktop
      have an openbox version, or lxde, an lightweight also full featured
      Ubuntu desktop
- [ ] setup iptables instead of socat
- [ ] find replacement of supervisord, need a lightweight mananger also has
      dependencies management. But sysvinit, upstart, or systemd is too heavy.
