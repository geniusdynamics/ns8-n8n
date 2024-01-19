# ns8-paperless ngx
Paperless-ngx is a community-supported open-source document management system that transforms your physical documents into a searchable online archive so you can keep, well, less paper.

## Install

Instantiate the module with:

    add-module ghcr.io/compgeniuses/n8n:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "n8n", "image_name": "n8n", "image_url": "ghcr.io/compgeniuses/n8n:latest"}

## Configure

Let's assume that the paperless instance is named `n8n1`.

Launch `configure-module`, by setting the following parameters:

- `PAPERLESS_ADMIN_PASSWORD`: Define the Default password Set to = P@perle5$
- `lets_encrypt`: Set LEtsecnrypt to True or False, Default is FALSE
- `http2https`: set redirect to True or False, Default is True
- `host`: the traefik host url for the project

- ...

Example:

    api-cli run module/n8n1/configure-module --data '{"host": "n8n.domain.com"}'

    or if modifying another value: 

    api-cli run module/n8n5/configure-module --data '{"host": "n8n.domain.com","n8n_name": "Myn8n NGX"}'

    api-cli run module/n8n1/configure-module --data '{
        "host": "n8n.rocky9-pve2.org",
        "lets_encrypt": false,
        "http2https": true,
    }'


The above command will:
- start and configure the n8n instance
- (describe configuration process)
- ...

Additional Parameters are Described here:
https://docs.paperless-ngx.com/configuration/#hosting-security
WHile they have not been Implemented, if you require more parameters to be defined, kindly free to raise an issue, and define why and how that parameter should be implemented for use

Send a test HTTP request to the ns8-paperless-ngx backend service:

    curl http://127.0.0.1/n8n/

## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the
`configure-module` action input: they are discovered by looking at some
Redis keys.  To ensure the module is always up-to-date with the
centralized [smarthost
setup](https://nethserver.github.io/ns8-core/core/smarthost/) every time
kickstart starts, the command `bin/discover-smarthost` runs and refreshes
the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when ns8-paperless-ngx is already
running, the event handler `events/smarthost-changed/10reload_services`
restarts the main module service.

See also the `systemd/user/paperless-server.service` file.

This setting discovery is just an example to understand how the module is
expected to work: it can be rewritten or discarded completely.

## Uninstall

To uninstall the instance:

    remove-module --no-preserve n8n1

## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/ns8-paperless-ngx:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project

## To Do
[Optional Services:](https://docs.paperless-ngx.com/configuration/#optional-services)
- Understand and Implement [Apache Tika](https://tika.apache.org/) to your repository
- Understand and Implement Docker  [gotenberg](https://gotenberg.dev/) to your repository

Paperless can make use of Tika and Gotenberg for parsing and converting "Office" documents (such as ".doc", ".xlsx" and ".odt"). Tika and Gotenberg are also needed to allow parsing of E-Mails (.eml).