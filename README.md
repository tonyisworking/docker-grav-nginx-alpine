# Lightweight PHP-FPM & Nginx Docker Image for Grav CMS

A Docker for quickly getting [Grav](https://getgrav.org/) CMS running on PHP 7.1 and Nginx under Alpine.

The Automated Docker Build of this repo is located [here](https://hub.docker.com/r/tonyisworking/grav-php7/). It should working and ready to go, if you have issues feel free to ask.

**Use at your own risk!!!**

## Overview

The web root folder is located at `/var/www/grav`

Grav is already preloaded into the web root folder, so you can just start the container and you're up and running. 

If you want to mount your own grav installation feel free to do so by overriding the web root location. 

If you want to load your own `user` folder, then mount your files into `/var/www/grav/user`. An example of this is in the sample docker compose file.

The container is set to run `bin/grav update` and `bin/grav install` on every initialization so it should keep itself updated

If the container is loaded with environment  `INSTALL_CMS=true` then grav will be re-installed upon initialization into the web root folder. Useful if you need to reinstall to an empty directory. 

You can turn on php opcache with `BUILD_ENV=prod`

I've also loaded a copy of the [html5boilerplate](https://html5boilerplate.com/) 404 page as the 404 page for nginx. 

To run `grav` cli commands just exec into the container with `docker exec -ti {container_name} /bin/bash` then you can `bin/gpm` or `bin/grav` all you want.



## Docker-Compose Sample

I'm making use of [jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy) to easily handle routing among multiple containers. Feel free to not use it.

```
version: '2'
services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    ports:
    - '80:80'
    - '443:443'
    volumes:
    - '/var/run/docker.sock:/tmp/docker.sock:ro'

  grav:
    image: tonyisworking/grav-php7
    container_name: grav
    ports:
    - '8001:80'
    volumes:
    - '{path_to_grav_user_folder}:/var/www/grav/user'
    environment: 
    - 'VIRTUAL_HOST=grav.dev'
    - 'VIRTUAL_PORT=80'
    - 'INSTALL_CMS=TRUE'
    - 'BUILD_ENV=prod'

```
## Steps to get going:

- Put the above into a `docker-compose.yml` file. 
- Replace `{path_to_grav_user_folder}` with your user folder path or delete this line as well as the `volumes:` line
- Add `127.0.0.1 grav.dev` to your hosts file.
- Open up your terminal to your where you put `docker-compose.yml` and run `docker-compose up -d`
- Open your browser and navigate to `grav.dev`
- You should see Grav saying it is up and running!


## Deploying User folder from Bitbucket

I host my user folder on bitbucket and want to be able to deploy automatically, so I added a script that replaces the user folder with one from bitbucket.

Adding `DEPLOY_GIT=true` tells the container to use the deployment script.

Steps to deployment:
- Host your `user` folder as bitbucket git project
- Need to setup [Bitbucket Access Keys](https://confluence.atlassian.com/bitbucket/use-access-keys-294486051.html)
- Add your generated private key `id_rsa` to the container by mounting it to this file: `/home/id_key`
- Add your Bitbucket SSH Deployment url to this environment variable `GIT_URL={URL HERE}`
- You can also specify the git branch using `GIT_BRANCH`

Sample Docker should now look similar to this:
```
version: '2'
services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    ports:
    - '80:80'
    - '443:443'
    volumes:
    - '/var/run/docker.sock:/tmp/docker.sock:ro'

  grav:
    image: tonyisworking/grav-php7
    container_name: grav
    ports:
    - '8001:80'
    volumes:
    - ''{path_to_grav_user_folder}:/var/www/grav/user'
    - '{path_to_id_rsa_file}:/home/id_key'
    environment: 
    - 'VIRTUAL_HOST=grav.dev'
    - 'VIRTUAL_PORT=80'
    - 'INSTALL_CMS=false'
    - 'BUILD_ENV=prod'
    - 'GIT_URL=git@bitbucket.org:username/gitproject.git'
    - 'DEPLOY_GIT=true'

```
Everytime the container is started, it will delete the user folder, clone in the git user folder, and clean the caches. It should work.

## What's inside this container:
- php7
- php-fpm7
- composer
- nginx
- alpine
- s6-overlay

## License
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html)

This docker was based off of [docker-wordpress](https://github.com/devgeniem/docker-wordpress) settings.


