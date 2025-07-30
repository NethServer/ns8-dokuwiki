# Dokuwiki

Start and configure a Dokuwiki instance.
The module uses [Bitnami DokuWiki image](https://github.com/bitnami/bitnami-docker-dokuwiki).

## Install

Instantiate the module:
```
add-module dokuwiki 1
```

The output of the command will return the instance name.
Output example:
```
{"module_id": "dokuwiki1", "image_name": "Dokuwiki", "image_url": "ghcr.io/nethserver/dokuwiki:latest"}
```

## Configure

Let's assume that the dokuwiki istance is named `dokuwiki1`.

Then launch `configure-module`, by setting the following parameters:
- `wiki_name`: wiki name
- `username`: administrator user
- `password`: administrator password
- `email`: administrator mail address
- `user_full_name`: administrator full name
- `host`: a fully qualified domain name for the wiki
- `http2https`: enable or disable HTTP to HTTPS redirection
- `lets_encrypt`: enable or disable Let's Encrypt certificate

Example:
```
api-cli run configure-module --agent module/dokuwiki1 --data - <<EOF
{
  "wiki_name": "mywiki",
  "username": "admin",
  "password": "admin",
  "email": "admin@test.local",
  "user_full_name": "Administrator",
  "host": "dokuwiki.test.local",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

The above command will:
- start and configure the Dokuwiki instance
- configure a virtual host for trafik to access the instance

## PHP settings

you can modify in the environment file some PHP settings, you need to restart the container after the modification

```
PHP_MEMORYLIMIT=512M
PHP_TIMEZONE=UTC
PHP_UPLOADLIMIT=256M
```

## Uninstall

To uninstall the instance and remove traefik virtual host:
```
remove-module dokuwiki1 --no-preserve
```

## API reference

See https://github.com/NethServer/ns8-dokuwiki/tree/apidoc

    remove-module --no-preserve dokuwiki1

## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/ns8-dokuwiki:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project
