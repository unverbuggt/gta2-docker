# GTA2 Multiplayer in Docker Containers

## Introduction

The multiplayer mode of games like the 2D GTA series (GTA1, GTA London, GTA2) don't work well over the Internet (with VPN to mimic LAN play).
The main problem is, that the players and NPCs (cars and pedestrians) need to be synchronized between all the clients
and even little latency slows the whole game down. 
A friend came up with the idea to put the game instances in containers to eliminate the latency issue 
and stream it to the players via remote play to the browser.

## Setup

Clone this repository with `git clone https://github.com/unverbuggt/gta2-docker.git`.

Edit `docker-compose.yaml` to your needs.
The `environment` section is currently configured for GPU accellerated h264 encoding (I use amdgpu)
and limits the resolution and bitrate to fit my Internet bandwidth.
Visit the documentation of [linuxserver/winegui](https://docs.linuxserver.io/images/docker-winegui/) for full reference.

Execute `source create_containers` to create the containers.

Start the first container with `docker container start wine-gta2-1`.

Now do the following:
1. Connect to [https://localhost:3001/](https://localhost:3001/).
2. It will show the Error "Could not open registry file!".
3. Click on Start (Wine symbol in the lower left) -> Utilities -> Winetricks.
4. Cancel "Wine Mono Installer".
5. Click "OK" a few times until "Winetricks - chose a wineprefix" is shown. Press OK.
6. Decide whether or not to help Winetricks development.
7. Check "Install Windows DLL or component" -> OK.
8. Check "directplay" and press OK.
9. Press OK a few times and wait for Installation to finish. Press OK on the regsvr32 windows.
10. Press Cancel a few times.
11. Download GTA2 from [Rockstar Classics](https://gta.com.ua/rockstargames-classics-free-download.phtml).
12. Extract GTA2.exe from the archive GTA2INSTALLER.ZIP (md5sum 4bf0b5f995d659090b681dd7b410499e) to `./game`.
13. In WineGUI click on "Run Program..." and select "C:\GTA2.exe" -> "Open".
14. Note that installation may fail (white screen). You'll have to stop the container `docker container stop wine-gta2-1` and retry the previous step.
15. "Next >", accept agreement, Change "Install GTA2 to" to "C:\GTA2", "Next >", "Install" and "Finish".
16. Download [vike's GTA2 update patch enhancement upgrade fix v11.44](https://gtamp.com/GTA2/gta2-installer.exe) (md5sum 53af6eea34a395cbf6a9efd1180190e1) and save to `./game`.
17. In WineGUI click on "Run Program..." and select "C:\gta2-installer.exe" -> "Open".
18. Install to the default folder: "C:\users\abc\Documents\GTA2".
19. Execute `source copy_patch1144`.
15. Start "GTA2 Manager".
16. At the "Video" Tab select "800x600".
17. At "Control" press "Default Keyboard" or change to your liking.
18. Start the game once with the "GTA2" Button.
19. Close all windows.
20. Stop the container with `docker container stop wine-gta2-1`.
21. Run `source copy_wine_1_to_23456`.
22. Run `source start_containers`.

The six containers expose port 3001/3002/3003/3004/3005/3006 (https with self signed certificate) and port 3011/3012/3013/3014/3015/3016 (http).
The browser clients should always connect via https, as some features like hardware accellerated video decoding won't work otherwise.

I use the http ports and `SUBFOLDER` env variables for these containers as I want to proxy them to the Internet (https)
via nginx reverse proxy like this:

```
...
        location /wine1/ {
                # WebSocket support (nginx 1.4)
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
                proxy_read_timeout 86400;

                proxy_pass http://[IP with the running containers]:3011;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forward-Proto $scheme;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                auth_basic "GTA-MAX";
                auth_basic_user_file /var/www/gta2-htpasswd;
        }
        location /wine2/ {
                # WebSocket support (nginx 1.4)
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
                proxy_read_timeout 86400;

                proxy_pass http://[IP with the running containers]:3012;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forward-Proto $scheme;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                auth_basic "GTA-MAX";
                auth_basic_user_file /var/www/gta2-htpasswd;
        }
and so forth...
```

## Game

Connect to the docker containers via a modern web browser:

- `https://[IP with the running containers]:3001`
- `https://[IP with the running containers]:3002`
- `https://[IP with the running containers]:3003`
- `https://[IP with the running containers]:3004`
- `https://[IP with the running containers]:3005`
- `https://[IP with the running containers]:3006`

To start multiplayer gaming:
1. In "GTA2 Manager" switch to tab "Network" and click "Start Network Game".
2. One Player need to create a game, the others join.

Check out the [GTA 1/GTA 2 Direct IP multiplayer guide](https://gtaforums.com/topic/982736-gta-1gta-2-direct-ip-multiplayer-guide/) for details.

