This project leverages docker-compose to stand up a self contained version of all Convergence Services to create a full Convergence environment. This project can be used as a local development environment or the basis of a production deployment using Docker Compose.

# Support

[Convergence Labs](https://convergencelabs.com) provides several different channels for support:

- For assistance with large deployments or for deploying on systems like Kubernetes, use the [Convergence Community Forum](https://forum.convergence.io) or [contact us](https://convergence.io/contact-sales/) directly.
- Chat with us on the [Convergence Public Slack](https://slack.convergence.io).
- To report problems in this repository, use the [Convergence Management Project](https://github.com/convergencelabs/convergence-project).

# Hostname Configuration
If running in Linux, using Docker for Mac, or Docker for Windows, it is likely that your docker hostname will be "localhost". If using Docker Toolbox, or if you have installed docker in a Linux VM, it is possible that your docker hostname will be something else (for example an IP Address).

By default the `.env` file contains the following:

```
DOCKER_HOSTNAME=localhost
```

If you docker host is not localhost, update this setting to point to the right host.

You will also need to edit the default nginx config file `config/nginx/default.conf` to set the `server_name` to the proper hostname.  Just do a find and replace of `localhost` to your hostname.

## Browsing and Service URLs
Please note that this installation contains a self signed SSL Certificate that will need to be accepted. Also note that the hostname may not be localhost (as discussed above).

* Dev Launchpad URL: `https://<hostname>/`
* REST API Endpoint: `https://<hostname>/rest/v1`
* Web Socket Endpoint: `https://<hostname>/realtime/`

# Persistent Data
When the environment starts up, it will create a `data` directory where all persistent data from the services will be stored. You can tear down and recreate the environment and data will still persist in the data directory. If you would like to start from scratch simply delete the data directory.

# Kibana and Logging
The project provides log aggregation through Elasticsearch, FluentD, and Kibana. When the system is first initialized, Kibana will not be configured to visualize logs from any particular Elasticsearch index. To configure logging follow these steps:

* Browse to https://localhost/kibana/
* Click the `Discover` link in the side panel. This will bring you to the "Create index pattern" dialog.
* Enter `logstash-*` in the `Index pattern` field.
* Click `Next step`.
* Select `timestamp` from the `Time Filter field name` dropdown.
* Click `Create index pattern`.
* Click `Discover` from the side panel again.

At this point you should see log entries populated and you can begin to explore the logs. Consult the [Kibana](https://www.elastic.co/products/kibana) documentation for information on working with Kibana.

# Additional configuration

Within the `config` directory you'll see a few different sets of configurations for the various containers (see below).  You'll probably be most interested in the `convergence` and `nginx` directories.  

To use your own SSL certs, edit the `config/nginx/default.conf` file appropriately and drop your certs in the `config/nginx/ssl` directory. 

You can configure the default namespace, domain, and administration user (to log into the administration console) in `config/convergence/convergence-server.conf`. 

## Security

You probably only want to provide public access to the API endpoints listed above, and lock down everything else.  

# Docker Usage

## Services / Containers
* `server-orientdb`
* `server-cluster-seed`
* `server-node`
* `admin-console`
* `elasticsearch`
* `kibana`
* `fluentd`
* `proxy`: an nginx reverse proxy for the whole Convergence environment

## Start Full Deployment
To start up the entire environment issue the following command
```
docker-compose up
```

To start the environment the background run:

```
docker-compose up -d
```


## Start Individual Containers
To start up only a single container use:
```
docker-compose up <container-name>
```

If you change a container, docker up will undeploy the current one and re-deploy the new one if there are any changes.

## Stop Individual Containers
To start up only a single container use:
```
docker-compose stop <container-name>
```

## Tear down the environment
To tear down the environment use:
```
docker-compose down
```

# SSL with Let's Encrypt

We have provided a convenient script for automatically installing and renewing SSL certificates with the free Let's Encrypt service.  To use this:

1. Uncomment the `certbot` container config in the `docker-compose.yml` file.
1. Modify the default configurations at the top of the `init-certs.sh` script. We recommend leaving `staging=0` until a certificate is successfully generated to avoid hitting letsenrypt request limits while testing.
1. Run the `init-certs.sh` script.
1. If a certificate was successfully generated, edit the `init-certs.sh` and set `staging=0`. 
1. Edit the `config/nginx/default.conf` `ssl_certificate` settings as noted in the file's comments.
1. Run the script again.  This will generate a certificate that browsers will accept.


# Troubleshooting

> The server-node container just keeps rebooting.  I can't tell what's happening. The logs just say "WARNING: no logs are available with the 'fluentd' log driver"

By default, all the logs are piped to FluentD.  Once the system starts the combination of ElasticSearch and Kibana provides a very nice log viewer, but it's not so useful if things are failing before it even starts up.  Thus, for startup debugging you can just comment out the customized logging in the appropriate section of the `docker-compose.yml`:

```
  server-node:
  ...
#   logging:
#     driver: fluentd
#     options:
#       fluentd-address: "localhost:24224"
#       fluentd-async-connect: "true"
#       tag: "docker.server-node"
```

If all else fails, post your problem at https://forum.convergence.io and the team will help you out.

> I'm getting `access denied` errors

We have seen periodic issues where files and directories are owned by `root` as opposed to the default user.  Just run `sudo chown -R <user> <directory>` as appropriate in the `config` or `data` subdirectories. 