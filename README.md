# concourse

## Install 
### Concourse
curl -Lo concourse https://github.com/concourse/concourse/releases/download/v3.3.4/concourse_darwin_amd64 && chmod +x concourse && mv concourse /usr/local/bin

### Fly
curl -Lo fly https://github.com/concourse/concourse/releases/download/v3.3.4/fly_darwin_amd64 && chmod +x fly && mv fly /usr/local/bin/

### Postgres 
brew install postgres

## Verification 

concourse --version

fly --version

pg_config --version 

psql --version

## Setup
Init db : initdb /usr/local/var/postgres

Start db server : pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start

Config db : createdb atc; createdb concourse; createuser concourse --pwprompt;

config rsa key : ssh-keygen -t rsa -f host_key -N '' && ssh-keygen -t rsa -f worker_key -N '' && ssh-keygen -t rsa -f session_signing_key -N '' & cp worker_key.pub authorized_worker_keys

## Start 

concourse web \
  --basic-auth-username concourse \
  --basic-auth-password the_password \
  --session-signing-key session_signing_key \
  --tsa-host-key host_key \
  --tsa-authorized-keys authorized_worker_keys
  --bind-port=9999
  
  Open browser : http://localhost:9999
  
 ## Test
 Create file hello.yml
 
`jobs:
 - name: hello-world
  plan:
  - task: say-hello
    config:
      platform: darwin
      run:
        path: echo
        args: ["Hello, world!"] `  
        
  Create target  : fly -t lite login -c http://127.0.0.1:9999
  
  Add the hello pipeline : fly -t lite set-pipeline -p hello-world -c hello.yml
  
  Run : 
  sudo concourse worker \
  --work-dir /opt/concourse/worker \
  --tsa-host 127.0.0.1 \
  --tsa-public-key host_key.pub \
  --tsa-worker-private-key worker_key
        
