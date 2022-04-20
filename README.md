# Synapse-docker

This is an attempt at making a very simple to deploy [synapse server]() using docker compose. 
It is intended for my personal usage because I didn't find something that satisfied me.
This is deeply inspired by [this](https://gist.github.com/matusnovak/37109e60abe79f4b59fc9fbda10896da) though.

## Prerequisites

### Domain name

You will need a domain name that you can buy at any place (like [Gandi](https://www.gandi.net/fr) for example).

When you have you domain name, you will need to create 2 sub-domains (that we will call `matrix.yourdomain.com` and `synapse.yourdomain.com`). This is usually done in the DNS section of the registrat you bought your domain name from, so look at their documentation if you need help (basically it consists to link each subdomain with an IP address, probably the same one in this setup)

### A Public IP address

You can use your own IP address at home (as long as it is fixed), or buy a VPS that will have one, there are many providers out there.

### Install docker and docker compose

Refer to [docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/) documentation to install them on your system.

## Create a sample config

```
docker run -it --rm \
    -v $(pwd)/data/matrix/synapse:/data \
    -e SYNAPSE_SERVER_NAME=matrix.example.com \
    -e SYNAPSE_REPORT_STATS=yes \
    -e UID=1000 \
    -e GID=1000 \
    matrixdotorg/synapse:latest generate
```

## Edit homeserver.yaml

At the very least, you should change:
* `server_name: matrix.yourdomain.com`
* `public_baseurl: synapse.yourdomain.com`
* in `database` section, comment out the `sqlite` part and activate the part that looks like this (you're likely to have to modify `user`, `password` and `host`):
```
database:
 name: psycopg2
 txn_limit: 10000
 args:
   user: synapse
   password: secretpassword
   database: synapse
   host: db
   port: 5432
   cp_min: 5
   cp_max: 10
```
* activate the commented out part in `redis`

## Edit Nginx files

* edit `data/matrix/nginx/matrix.conf` with your actual domain name
* same in `data/matrix/nginx/www/.well-known/matrix/server`

## Edit traefik file

In `data/traefik/traefik.yml`, add an email address you can be reached on in the `acme` section. 

## Edit the .env file

`.env` file contains some environment variables that are necessary for your docker containers. An example is provided, you probably only should change `POSTGRES_PASSWORD` (or not, just make sure that it matches the password in `homeserver.yaml`).

## Configure port redirection

This depends a lot on your setup, whether you're at home or your rent a VPS. 

If you run Synapse on a computer at home you'll likely need to log in your home router (see the documentation of your ISP). Then you'll have to look for `port redirection` and tell the router to redirect external port 443 to the port 443 of the machine that runs Synapse. 

If you don't know the local IP address of your computer, you can run `ip address` (on Linux), or look for your device in the interface of your router. 

Do the same redirection for port 80.

## (optional) Configure firewall

Depending on the actual situation of your network, you might want to run a firewall as an extra security as your machine is now facing the internet.

On Linux, look for `ufw`, it's probably installed by default or you can install it easily with your package manager. 

`sudo ufw enable`
`sudo ufw allow http https`

If you're on a computer at home, it's probably overkill as you're already hiding behind a router and hopefully you only redirect ports 80 and 443. 

## Start the docker compose 

`docker compose up -d`

You can keep an eye on the log with `docker compose logs -f`. Once you're sure everything is fine, Ctrl+c to quit.

You can turn off the server anytime using `docker compose down`

## Test that your server is reachable

Go to `https://federationtester.matrix.org/` and type in the domain name of your server (for example, `matrix.yourdomain.com`).

Everything should be green.

## Create your account

Since we deactivated registration, we need to create our user (that will also have admin rights) manually.

While the docker compose is running:
* `docker compose exec synapse /bin/bash`
* you should now been logged in a terminal that is actually your synapse container
* `register_new_matrix_user -c data/homeserver.yaml http://localhost:8008`
* Give you user a name, a password, and don't forger to say `yes` when asked if he has admin rights

You should have a confirmation message if the user was created.

Now quit the container with `exit`.

## Download Elements and log in

You should easily find it in your favorite app store

When you open it for the first time, it will asks if you want to create a new account. Since you just created one, say that you want to log in with an existing user. 

Type username as `@username:matrix.yourdomain.com`

The password is the one you defined previously.

You should now be logged in your own instance.
