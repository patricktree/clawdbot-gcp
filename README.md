# Clawdbot on GCP (Google Cloud Platform) via Terraform <!-- omit in toc -->

- [Prerequisites](#prerequisites)
  - [Telegram bot and user ID for using clawdbot](#telegram-bot-and-user-id-for-using-clawdbot)
  - [Tools for setting up the VM on GCP](#tools-for-setting-up-the-vm-on-gcp)
- [Create \& Configure GCP Project](#create--configure-gcp-project)
- [Create GCP Infrastructure](#create-gcp-infrastructure)
- [TODO](#todo)

## Prerequisites

### Telegram bot and user ID for using clawdbot

1. **Create a Telegram Bot:**
   1. Open Telegram.
   2. Contact `@botfather`, send text message `/start`.
   3. Send text message `/newbot`.
   4. Name `clawdbot`
   5. Some username (e.g. `john_doe_clawdbot`).
   6. Remember the returned token.

2. **Get your Telegram User ID:**
   1. Open Telegram.
   2. Contact `@userinfobot`, send text message `/start`.
   3. Remember the returned ID for your user (e.g. `345123789`).

### Tools for setting up the VM on GCP

1. **Install the `gcloud` CLI:** See [cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install).
2. **Install Terraform:** See [learn.hashicorp.com/tutorials/terraform/install-cli#install-terraform](https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/install-cli).

## Create & Configure GCP Project

1. **Use Terraform to create and configure a GCP project:** Run the steps outlined in [the README of `./01-terraform-gcp-project`](./01-terraform-gcp-project/README.md).
2. **Configure `gcloud` CLI the created GCP project:**

   ```sh
   gcloud init
   # - when asked to choose a project, make sure to choose the new project
   # - when asked for default Compute Region and Zone, choose "[15] europe-west4-a"

   gcloud config get-value project
   # should print the correct GCP project ID

   gcloud auth application-default login
   # Terraform will now use your account to access GCP
   ```

## Create GCP Infrastructure

1. **Use Terraform to create the GCP infrastructure:** Run the steps outlined in [the README of `./02-terraform-gcp-infrastructure`](./02-terraform-gcp-infrastructure/README.md).

## TODO

```sh
gcloud compute ssh clawdbot

sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER
newgrp docker

git clone https://github.com/patricktree/clawdbot.git"
cd ./clawdbot
cat <<'EOF' > .env
CLAWDBOT_HOME_VOLUME="clawdbot_home"
CLAWDBOT_DOCKER_APT_PACKAGES="build-essential curl file git"
TAILSCALE_AUTHKEY="replace-me"
EOF
./docker-setup.sh

docker compose -f /home/pkerschbaum/clawdbot/docker-compose.yml run --rm clawdbot-cli configure
# configure gateway: bind LAN, tailscale exposure Off, copy token at the end

# Open host port to VM port for Gateway
gcloud compute ssh clawdbot -- -N -L 18789:localhost:18789

# Write the bot via Telegram, it sends you back a code. Then:
docker compose -f /home/pkerschbaum/clawdbot/docker-compose.yml run --rm clawdbot-cli pairing approve telegram <code>
```
