# Use Ansible and Ansible Semaphore UI

## Install ansible and semaphore

```bash
sudo apt install ansible
sudo snap install semaphore
```

## Setup semaphore

```bash
sudo snap stop semaphore
sudo semaphore user add --admin \
--login john \
--name=John \
--email=john1996@gmail.com \
--password=12345
sudo snap start semaphore
```

Go to `http://<IP or HOSTNAME>:3000`, eventually use traefik or nginx to proxy with SSL

## Configure and test Semaphore

- Create a repo on Github, follow the instructions to add an ssh key to authenticate to github
- Add Keys to Semaphore: one with the private key used to authenticate to github, others with usee/pass or key authentication to the machines to manage
- Create a default environment, can be empty (put {} on both the spaces)
- Create a repository, using the ssh url of your repo and the github key previously created
- Create one or more inventories, at least one per authentication key, selecting the relative authentication key
- Create one test template per inventory, pointing to a ping playbook in your repo and run it to test connections.
