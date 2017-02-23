# Boilerplate Neos distribution

The idea of this repo is to serve as a boilerplate for all [Neos](https://neos.io) projects that [we](https://psmb.github.io) start. It helps us to save _A LOT_ of times when kickstarting new Neos projects.

**WARNING: Don't use it directly, as it has a lot of code that is specific to our infrastructure! Rather clone and tune it for your own needs.**

## Workflow

1. Create a docker-compose.yml similar to the one listed below and run `docker-compose up -d`
2. Monitor `docker-compose logs web` until you see something like `INFO success: php-fpm entered RUNNING state`. Then go to the website url specified in docker-compose.yml and make sure it works.
3. Go to the location of the code (`data/www/releases/current`) and remove git files with `rm -rf .git`.
4. Edit all files as needed to start new project, create a new git repository and push it to Github when ready. Edit `docker-compose.yml` and `circle.yml` to point to new Github repo. Do the same on production/staging server when ready to go live.

## Example docker-compose.yml

```
dbdata:
  image: busybox:latest
  command: echo dbdata
  volumes:
    - /var/lib/mysql

db:
  image: million12/mariadb:10.2
  expose:
    - '3306'
  volumes_from:
    - dbdata
  environment:
    MARIADB_PASS: pass

webdata:
  image: busybox:latest
  command: echo webdata
  volumes:
    - ./data:/data
    - ./Persistent:/data/www/shared/Data/Persistent

web:
  image: dimaip/neos-bare:2.0
  ports:
    - '80'
  links:
    - db:db
  volumes_from:
    - webdata
  environment:
    VIRTUAL_HOST: 'domain.loc,dev.domain.loc,www.domain.loc'
    REPOSITORY_URL: 'https://github.com/psmb/BoilerplateDistr'
    SITE_PACKAGE: 'Sfi.Site'
    ADMIN_PASSWORD: 'password'

ssh:
  image: million12/php-app-ssh:php-70
  ports:
    - '22'
  links:
    - db:db
    - web:web
  volumes_from:
    - webdata
  environment:
    IMPORT_GITHUB_PUB_KEYS: dimaip
```
