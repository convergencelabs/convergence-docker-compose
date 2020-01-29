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

# Additional configuration

Within the `config` directory you'll see a few different sets of configurations for the various containers (see below).  You'll probably be most interested in the `convergence` and `nginx` directories.  

To use your own SSL certs, edit the `config/nginx/default.conf` file appropriately and drop your certs in the `config/nginx/ssl` directory. 

You can configure the default namespace, domain, and administration user (to log into the administration console) in `config/convergence/convergence-server.conf`. 

## System requirements

If you're spinning up all the containers defined in the `docker-compose.yml` in this repository on a single cloud compute instance, make sure this instance has AT LEAST 4GB RAM (and preferably 6GB+) available.  Otherwise you will see errors in the server and/or orientdb.

## Security

You probably only want to provide public access to the API endpoints listed above, and lock down everything else.  

# Docker Usage

## Services / Containers
* `orientdb`: Provides the core database for Convergence
* `cluster-seed`: The seed node of the Convergence cluster
* `server`: The Convergence Server
* `admin-console`: The Administration Console user interface
* `proxy`: An nginx reverse proxy for the whole Convergence environment

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

# Troubleshooting
If all else fails, post your problem at https://forum.convergence.io and the team will help you out.

> I'm getting `access denied` errors

We have seen periodic issues where files and directories are owned by `root` as opposed to the default user.  Just run `sudo chown -R <user> <directory>` as appropriate in the `config` or `data` subdirectories. 
