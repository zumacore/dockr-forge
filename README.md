# zuma/dockr-forge

`dockr-forge` is a simple package to install [Traefik Proxy](https://traefik.io/traefik) on a standalone [Laravel Forge](https://forge.laravel.com) Worker server in order to route domains and/or sub-domains to Docker containers.

1. Create a new Worker Server at whichever cloud service you wish to use.

2. Install Docker and Docker Compose by either SSH or better yet create a Forge Recipe that runs as root that contains the following command:

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
UBUNTU_VERSION=$(lsb_release -cs)
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $UBUNTU_VERSION stable"
apt-cache policy docker-ce
apt install -y docker-ce
systemctl status docker
usermod -aG docker forge
COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
sh -c "curl -L https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
chmod +x /usr/local/bin/docker-compose
```

3. Ensure you have a fully-qualified domain pointed to the IP of this server.

> IMPORTANT: This is crucial to have a valid propagated domain set up or Let's Encrypt won't dynamically create certificates for your containers.

4. Create a new site named `traefik` on your new server.

5. In the Application tab, install the `zumacore/dockr-forge` repository from Github using the `main` branch and uncheck `Install Composer Dependencies` as there are none.

6. SSH into your server and run the following commands in order:

```bash
# First ensure that docker and docker-compose are installed:
$ docker -v
$ docker-compose -v
```

> If you don't see the versions displayed go back to Forge a run the Recipe on the server and try again.

```bash
# Create a docker network for your container to run on:
$ docker network ls | grep reverse_proxy || docker network create reverse_proxy

# cd into the traefik site directory
$ cd traefik

# rename the env file
$ mv .env.example .env

# rename to docker-compose.prod.yml file
$ mv docker-compose.prod.yml docker-compose.yml

# run the auth script to create a username and strong password
# replace {username} and {password} with your actual username and password
$ ./run auth {username} {password}
```

Next return to Forge and go to the Environment tab and update and save the `env` variables.

```bash
DASHBOARD_HOST=traefik.your-registered-domain.tld
CHALLENGE_EMAIL=you@your-email-address.tld
```

Next go to the Deploy tab and update and save the deploy script:

```bash
cd /home/forge/traefik

git pull origin $FORGE_SITE_BRANCH

mv docker-compose.prod.yml docker-compose.yml

docker-compose up -d --remove-orphans
```

Next we should be ready to deploy, so go ahead and hit the `Deploy` button.

With any luck, you should see the containers being created in the deploy panel.

Back in your terminal run:

```bash
$ docker ps
```

And you should see your containers running.

Next go to the url you set in the `env` as `DASHBOARD_HOST` and you should be prompted for your credentials, then see the Traefik dashboard once you're logged in.

Best of luck!

Thanks to [Joao Patricio](https://github.com/ijpatricio) and [this great article](https://blog.jpat.dev/how-to-deploy-docker-applications-with-laravel-forge) along with the [original repo](https://github.com/ijpatricio/traefik-for-forge) used to create `zuma/dockr-forge`
