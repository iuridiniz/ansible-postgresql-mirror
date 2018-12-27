# Get start

## install into a virtualenv (ubuntu 18.04)

```
sudo apt install python3-virtualenv

python3 -mvirtualenv -ppython3 ./env
./env/bin/pip install -r python-requirements.txt

./env/bin/ansible --version

```

## Or just use direnv (ubuntu 18.04)

```

sudo apt-get install -y direnv python3-virtualenv

# put this on you ~/.bashrc ~/.zshrc
eval "$(direnv hook $0)"

cp envrc.sample .envrc

direnv allow .

pip install -r python-requirements.txt

ansible --version

```

# Development

## Create a test environment with 2 ubuntu machines using docker (ubuntu 18.04)

### Install and configure the environment

```
# install docker from https://docs.docker.com/install/linux/docker-ce/ubuntu/
sudo apt-key adv --fetch-keys https://download.docker.com/linux/ubuntu/gpg

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   bionic \
   stable"

apt-get install -y docker-ce

# add yourself to group docker
sudo usermod -a -G docker $(whoami)

```

### start two machines

```
# go to group docker (if you didn't logout)
newgrp docker

# create a keys
mkdir ssh
ssh-keygen -N '' -f ./ssh/id_rsa
cp ./ssh/id_rsa.pub ./ssh/authorized_keys
chmod 700 ./ssh
chmod 600 ./ssh/*

# create machines
docker run -d -P --name ssh1 -v $(pwd)/ssh:/tmp/sshkeys:ro --rm rastasheep/ubuntu-sshd
docker run -d -P --name ssh2 -v $(pwd)/ssh:/tmp/sshkeys:ro --rm rastasheep/ubuntu-sshd

# copy the keys
docker exec ssh1 sh -c 'cp -a --no-preserve=ownership /tmp/sshkeys/* /root/.ssh/'
docker exec ssh2 sh -c 'cp -a --no-preserve=ownership /tmp/sshkeys/* /root/.ssh/'

# generate a dev.ini
echo "[evolux]" > dev.ini
docker inspect -f "{{.NetworkSettings.IPAddress}}" ssh1 ssh2 >> dev.ini
cat << END >>dev.ini

[evolux:vars]
ansible_ssh_private_key_file=./ssh/id_rsa
ansible_user=root
END

# install python on machines
ansible -i dev.ini -m raw -a "sh -c 'apt-get update && apt-get -y install python'" all

# run ping
ansible -i dev.ini -m ping all

```

# Usage

just do:
```
ansible-playbook postgresql.yml -v -i <HOSTFILE> --extra-vars "replicant_passwd=REPLICANT_PASSWORD"
```
