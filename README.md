# Calibre Web

[![Docker Stars](https://img.shields.io/docker/stars/technosoft2000/calibre-web.svg)]()
[![Docker Pulls](https://img.shields.io/docker/pulls/technosoft2000/calibre-web.svg)]()
[![](https://images.microbadger.com/badges/image/technosoft2000/calibre-web.svg)](https://microbadger.com/images/technosoft2000/calibre-web "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/technosoft2000/calibre-web.svg)](https://microbadger.com/images/technosoft2000/calibre-web "Get your own version badge on microbadger.com")

## Calibre Web - Manage your Calibre e-book collection ##

[Calibre Web](https://github.com/janeczku/calibre-web) is a web app providing a clean interface for browsing, reading and downloading eBooks using an **existing Calibre database**.

![screenshot](https://raw.githubusercontent.com/janeczku/docker-calibre-web/master/screenshot.png)

__Calibre Web__ comes with the following features:

 * Bootstrap 3 HTML5 interface
 * full graphical setup
 * User management
 * Admin interface
 * User Interface in english, french, german, polish, simplified chinese, spanish
 * OPDS feed for eBook reader apps
 * Filter and search by titles, authors, tags, series and language
 * Create custom book collection (shelves)
 * Support for editing eBook metadata
 * Support for converting eBooks from EPUB to Kindle format (mobi/azw)
 * Restrict eBook download to logged-in users
 * Support for public user registration
 * Send eBooks to Kindle devices with the click of a button
 * Support for reading eBooks directly in the browser (.txt, .epub, .pdf)
 * Upload new books in PDF, epub, fb2 format
 * Support for Calibre custom columns
 * Fine grained per-user permissions
 * Self update capability

If you want to know more you can head over to the __Calibre Web__ project site: https://github.com/janeczku/calibre-web.

And if you are interested in the original __Calibre__ ebook management tool then look at the project site: https://calibre-ebook.com/.

## Updates ##

**2017-11-04 - v1.1.10**

 * added support for a configuration directory (as asked in ticket #13), 
   where the configuration related files like `app.db` and `gdrive.db` will be stored;
   be aware that `metadata.db` will be still stored at the books directory which is required by the original Calibre application
 * new options `-v <your Calibre Web config folder>:/calibre-web/config` and `-e USE_CONFIG_DIR=true` to setup the configuration directory

For previous changes see at [full changelog](CHANGELOG.md).

## Features ##

 * running Calibre Web under its own user (not root)
 * changing of the UID and GID for the Calibre Web user
 * no usage of NGINX inside the container, only the Calibre Web application is served as single application without any supervisor
 * Google Drive integration is included
 * creation of gdrive.db symlink at `/books/gdrive.db` for external access like backup possibility
 * creation of app.db symlink at `/books/app.db` for external access like backup possibility

## Hints & Tips ##
 
 * if you need SSL support similiar to the original Docker Container [janeczku/calibre-web](https://hub.docker.com/r/janeczku/calibre-web/) then use an additional NGINX or Apache HTTP Server as Reverse-Proxy, e.g see [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/)
 * if you don't specify __PGID__ and __PUID__ values the default __PGID__ and __PUID__ of the image are used,
and if they are used then the mapped host volume/directory which is alligned to `/books` must have _read-write-execute_ permission for **_others_** , otherwise the configuration of Calibre-Web can't be finished :-|
 * for Synology Users - don't map a top-level volume directory from the NAS as `/books` volume, e.g. `/volume1/books` because it results into problems with directory permissons. Create instead a subdirectory __calibre__ at `/volume1/books` and map then `/volume1/books/calibre` as volume for `/books`

## Configuration at first launch ##
 1. Point your browser to `http://hostname:<HTTP PORT>` e.g. `http://hostname:8083`
 2. Set Location of your Calibre books folder to the path of the folder where you mounted your Calibre folder in the container, which is by default `\books`.
    So enter at the field __Location of Calibre database__ the mapped volume `\books`.
 3. Hit __Submit__ then __Login__.

Default admin login:
 * __Username:__ admin
 * __Password:__ admin123

After successful login change the default password and set the email adress.

To access the OPDS catalog feed, point your Ebook Reader to `http://hostname:<HTTP PORT>/opds`

## Usage ##

__Create the container:__

```
docker create --name=calibre-web --restart=always \
-v <your Calibre books folder>:/books \
[-v <your Calibre Web config folder>:/calibre-web/config \]
[-e USE_CONFIG_DIR=true \]
[-e APP_REPO=https://github.com/janeczku/calibre-web.git \]
[-e APP_BRANCH=master \]
[-e SET_CONTAINER_TIMEZONE=true \]
[-e CONTAINER_TIMEZONE=<container timezone value> \]
[-e PGID=<group ID (gid)> -e PUID=<user ID (uid)> \]
-p <HTTP PORT>:8083 \
technosoft2000/calibre-web
```

__Example:__

```
docker create --name=calibre-web --restart=always \
-v /volume1/books/calibre:/books \
-v /etc/localtime:/etc/localtime:ro \
-e PGID=65539 -e PUID=1029 \
-p 8083:8083 \
technosoft2000/calibre-web
```

or

```
docker create --name=calibre-web --restart=always \
-v /volume1/books/calibre:/books \
-v /volume1/docker/apps/calibre-web/config:/calibre-web/config \
-e USE_CONFIG_DIR=true \
-e SET_CONTAINER_TIMEZONE=true \
-e CONTAINER_TIMEZONE=Europe/Vienna \
-e PGID=65539 -e PUID=1029 \
-p 8083:8083 \
technosoft2000/calibre-web
```

__Start the container:__
```
docker start calibre-web
```

## Parameters ##

### Introduction ###
The parameters are split into two parts which are separated via colon.
The left side describes the host and the right side the container. 
For example a port definition looks like this ```-p external:internal``` and defines the port mapping from internal (the container) to external (the host).
So ```-p 8080:80``` would expose port __80__ from inside the container to be accessible from the host's IP on port __8080__.
Accessing http://'host':8080 (e.g. http://192.168.0.10:8080) would then show you what's running **INSIDE** the container on port __80__.

### Details ###
* `-p 8083` - http port for the web user interface
* `-v /books` - local path which contains the Calibre books and the necessary `metadata.db`  which holds all collected meta-information of the books
* `-v /etc/localhost` - for timesync - __optional__
* `-v /calibre-web/config` - local path for Calibre Web config files, like `app.db` and `gdrive.db`; **IMPORTANT**: the environment variable `USE_CONFIG_DIR` must be set to `true` - __optional__
* `-e USE_CONFIG_DIR=true` - activate the usage of a dedicated configuration directory, otherwise the `books` directory will be used (default) - __optional__
* `-e APP_REPO` - set it to the Calibre Web GitHub repository; by default it uses https://github.com/janeczku/calibre-web.git - __optional__
* `-e APP_BRANCH` - set which Calibre Web GitHub repository branch you want to use, __master__ (default branch) - __optional__
* `-e SET_CONTAINER_TIMEZONE` - set it to `true` if the specified `CONTAINER_TIMEZONE` should be used - __optional__
* `-e CONTAINER_TIMEZONE` - container timezone as found under the directory `/usr/share/zoneinfo/` - __optional__
* `-e PGID` for GroupID - see below for explanation - __optional__
* `-e PUID` for UserID - see below for explanation - __optional__

### Container Timezone

In the case of the Synology NAS it is not possible to map `/etc/localtime` for timesync, and for this and similar case
set `SET_CONTAINER_TIMEZONE` to `true` and specify with `CONTAINER_TIMEZONE` which timezone should be used.
The possible container timezones can be found under the directory `/usr/share/zoneinfo/`.

Examples:

 * ```UTC``` - __this is the default value if no value is set__
 * ```Europe/Berlin```
 * ```Europe/Vienna```
 * ```America/New_York```
 * ...

Once the container is running you can get all possible timezones as tree via the command ```docker exec -it <CONTAINER> tree /usr/share/zoneinfo```

See also at [possible timezone values](TIMEZONES.md).

__Don't use the value__ `localtime` because it results into: `failed to access '/etc/localtime': Too many levels of symbolic links`

## User / Group Identifiers ##
Sometimes when using data volumes (-v flags) permissions issues can arise between the host OS and the container. We avoid this issue by allowing you to specify the user PUID and group PGID. Ensure the data volume directory on the host is owned by the same user you specify and it will "just work" ™.

In this instance PUID=1001 and PGID=1001. To find yours use id user as below:

```
  $ id <dockeruser>
    uid=1001(dockeruser) gid=1001(dockergroup) groups=1001(dockergroup)
```

## Additional ##
Shell access whilst the container is running: `docker exec -it calibre-web /bin/bash`

Upgrade to the latest version of Calibre Web: `docker restart calibre-web`

To monitor the logs of the container in realtime: `docker logs -f calibre-web`

To monitor the logs of Calibre Web: `docker exec -it calibre-web tail -f /calibre-web/app/calibre-web.log`

Show used base image version number of Calibre Web: `docker inspect -f '{{ index .Config.Labels "image.base.version" }}' calibre-web`

Show used image version number of Calibre Web: `docker inspect -f '{{ index .Config.Labels "image.version" }}' calibre-web`

---

## For Synology NAS users ##

Login into the DSM Web Management
* Open the Control Panel
* Control _Panel_ > _Privilege_ > _Group_ and create a new one with the name 'docker'
* add the permissions for the directories 'downloads', 'video' and so on
* disallow the permissons to use the applications
* Control _Panel_ > _Privilege_ > _User_ and create a new on with name 'docker' and assign this user to the group 'docker'

Connect with SSH to your NAS
* after sucessful connection change to the root account via
```
sudo -i
```
or
```
sudo su -
```
for the password use the same one which was used for the SSH authentication.

* create a 'docker' directory on your volume (if such doesn't exist)
```
mkdir -p /volume1/docker/
chown root:root /volume1/docker/
```

* get your Docker User ID and Group ID of your previously created user and group
```
id docker
uid=1029(docker) gid=100(users) groups=100(users),65539(docker)
```

* get the Docker image
```
docker pull technosoft2000/calibre-web
```

* create a Docker container (take care regarding the user ID and group ID, change timezone and port as needed)
```
docker create --name=calibre-web --restart=always \
-v /volume1/books/calibre:/books \
-e SET_CONTAINER_TIMEZONE=true \
-e CONTAINER_TIMEZONE=Europe/Vienna \
-e PGID=65539 -e PUID=1029 \
-p 8083:8083 \
technosoft2000/calibre-web
```

* check if the Docker container was created successfully
```
docker ps -a
CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS               NAMES
40cc1bfaf7be        technosoft2000/calibre-web      "/bin/bash -c /init/s" 8 seconds ago       Created 
```

* start the Docker container
```
docker start calibre-web
```

* analyze the log (stop it with CTRL+C)
```
docker logs -f calibre-web
        ,----,                                   
      ,/   .`|                                   
    ,`   .'  : .--.--.        ,----,        ,-.  
  ;    ;     //  /    '.    .'   .' \   ,--/ /|  
.'___,/    ,'|  :  /`. /  ,----,'    |,--. :/ |  
|    :     | ;  |  |--`   |    :  .  ;:  : ' /   
;    |.';  ; |  :  ;_     ;    |.'  / |  '  /    
`----'  |  |  \  \    `.  `----'/  ;  '  |  :    
    '   :  ;   `----.   \   /  ;  /   |  |   \   
    |   |  '   __ \  \  |  ;  /  /-,  '  : |. \  
    '   :  |  /  /`--'  / /  /  /.`|  |  | ' \ \ 
    ;   |.'  '--'.     /./__;      :  '  : |--'  
    '---'      `--'---' |   :    .'   ;  |,'     
                        ;   | .'      '--'       
                        `---'                    
      PRESENTS ANOTHER AWESOME DOCKER IMAGE
      ~~~~~         Calibre Web       ~~~~~
[INFO] Docker image version: 1.1.10
[INFO] Alpine Linux version: 3.6.2
[WARNING] A group with id 100 exists already [in use by users] and will be modified.
[WARNING] The group users will be renamed to calibre
[INFO] Create user calibre with id 1029
[INFO] Current active timezone is UTC
Sat Nov  4 16:16:42 CET 2017
[INFO] Container timezone is changed to: Europe/Vienna
[INFO] Change the ownership of /calibre-web (including subfolders) to calibre:calibre
[INFO] Current git version is:
git version 2.13.5
[INFO] Checkout the latest Calibre-Web version ...
[INFO] ... git clone -b master --single-branch https://github.com/janeczku/calibre-web.git /calibre-web/app -v
Cloning into '/calibre-web/app'...
POST git-upload-pack (189 bytes)
[INFO] Autoupdate is active, try to pull the latest sources for Calibre-Web ...
[INFO] ... current git status is
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working tree clean
af8d908bbac22409e7cd5f3bcc7ea34e96193961
[INFO] ... pulling sources
Already up-to-date.
[INFO] ... git status after update is
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working tree clean
af8d908bbac22409e7cd5f3bcc7ea34e96193961
[INFO] Config directory option is ACTIVATED
> due this the directory /calibre-web/config will be used to store the configuration
[INFO] Change the ownership of /calibre-web/config (including subfolders) to calibre:calibre
[INFO] Checking permissions of the config directory: /calibre-web/config
> Output is: 755 calibre 100 calibre 1029
> Permissions: 755
> Assigned group: calibre
> Assigned group ID: 100
> Assigned owner: calibre
> Assigned owner ID: 1029
> Using permissions for checks: 0755
> The user calibre:1029 is the owner and has write access at /calibre-web/config
[INFO] 'app.db' and 'gdrive.db' will be linked into /calibre-web/config
> create 'app.db' link /calibre-web/app/app.db assigned to source /calibre-web/config/app.db
> create 'gdrive.db' link /calibre-web/app/gdrive.db assigned to source /calibre-web/config/gdrive.db
[INFO] Checking permissions of the books directory: /books
> Output is: 772 calibre 100 UNKNOWN 1026
> Permissions: 772
> Assigned group: calibre
> Assigned group ID: 100
> Assigned owner: UNKNOWN
> Assigned owner ID: 1026
> Using permissions for checks: 0772
> Everyone has write access at /books
[INFO] The mapped volume for /books contains a Calibre database file 'metadata.db' which will be used
[INFO] kindlegen (Amazon Kindle Generator) will be linked into /calibre-web/app/vendor
[INFO] Creating the vendor directory: /calibre-web/app/vendor
[INFO] Change the ownership of /calibre-web/app/vendor (including subfolders) to calibre:calibre
> create kindlegen link /calibre-web/app/vendor/kindlegen assigned to source /calibre-web/kindlegen/kindlegen
[INFO] Creating directory for temporary directories and files: /tmp
[INFO] Change the ownership of /tmp (including subfolders) to calibre:calibre
[INFO] Launching Calibre-Web ...
```
