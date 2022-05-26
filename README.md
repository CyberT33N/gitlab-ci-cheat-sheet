# gitlab-cheat-sheet
Cheat Sheet for gitlab











<br><br>
<br><br>
_________________________________________________________

<br><br>
<br><br>

# Docker
- https://docs.gitlab.com/ee/install/docker.html
- https://hub.docker.com/r/gitlab/gitlab-ee/
```bash
sudo docker pull gitlab/gitlab-ee
```


<br><br>
<br><br>


## docker-compose.yml
```yaml
version: '3.6'
networks:
  default:
    external:
      name: localdev
services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: 'gitlab.local.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.local.com'
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'
```

<br><br>

.env
```bash
GITLAB_HOME=/srv/gitlab
```

<br><br>

Next run:
```bash
sudo -E docker-compose up
```

<br><br>

Get your container name:
```bash
sudo docker ps
```

<br><br>


Inspect container to get ip adress where you can visit your local gitlab
```bash
sudo docker inspect containerNameHere
# Will stand anywhere at the buttom
# "IPAddress": "xx.x.x.x"
```
  - Next add to your /etc/hosts
  ```bash
  sudo gedit /etc/hosts
  12.34.56.78 gitlab.local.com
  ```


<br><br>

Get your Gitlab password:
```
sudo docker exec -it gitlabContainerNameHere bash
cd etc/gitlab
cat initial_root_password
```
  - You username is root






<br><br>

If needed you can reset your password of the root with:
```bash
sudo docker exec -it gitlab-ce bash
gitlab-rake "gitlab:password:reset[root]"
```





<br><br>

Next add your SSH Key on Gitlab by clicking on your Avatar > Settings > SSH Keys. Create a sample project and try if you can clone it via ssh
```bash
git clone git@gitlab.local.com:websites/porn-scrap.git

# If it ask "Are you sure you want to continue connecting (yes/no/[fingerprint])?" Use fingerprint 
# In my case I got an error after this "fatal: Could not read from remote repository.". But after this I tried again to git clone and it works.

```










<br><br>
<br><br>
_________________________________________________________

<br><br>
<br><br>

# Disable LFS
- Settings > General > Visibility, project features, permissions > Expand > Git Large File Storage (LFS)















<br><br>
<br><br>
_________________________________________________________

<br><br>
<br><br>

# gitlab-ci.yml
```yaml
stages:
  - test

scrap:
  stage: test
  script:
    - npm i && npm run test
```

<br><br>
In order to use Pipelines you must install gitlab-runner



<br><br>
<br><br>

## gitlab-runner


<br><br>
<br><br>

#### docker-compose local
```.env
GITLAB_HOME=/srv/gitlab
GITLAB_ROOT_PASSWORD=root
```


```yaml
# docker-compose.yml
networks:
  default:
    external:
      name: localdev

services:
  gitlab:
    extends:
      file: services/gitlab/service.yml
      service: web

  gitlab-runner:
    depends_on:
      - gitlab
    extends:
      file: services/gitlab-runner/service.yml
      service: gitlab-runner
```


```yaml
# services/gitlab/service.yml
version: '3.6'

services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: 'gitlab.local.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.local.com'
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'

```

```yaml
# services/gitlab-runner/service.yml
version: '3.6'

services:
  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    restart: unless-stopped
    volumes:
      - ./config/gitlab-runner:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
```

<br><br>

then run this command to configure the runner:
```bash
sudo docker-compose up -d
sudo docker-compose exec gitlab-runner gitlab-runner register
```

It will ask you for details about the GitLab instance you want to attach to.
You will find this information at https://<your-gitlab-domain>/admin/runners. This example is for my GitLab instance:
```
Runtime platform                                    arch=amd64 os=linux pid=38 revision=943fc252 version=13.7.0
Running in system-mode.

Enter the GitLab instance URL (for example, https://gitlab.com/):
https://gitlab.local.com/
Enter the registration token:
Loo2lahf9Shoogheiyae
Enter a description for the runner:
[148a53203df8]: My-Runner
Enter tags for the runner (comma-separated):

Registering runner... succeeded                     runner=oc-oKWMH
Enter an executor: custom, docker-ssh, shell, virtualbox, docker-ssh+machine, docker, parallels, ssh, docker+machine, kubernetes:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

<br><br>
If you specify tags then you must include them. To ignore tags and force run the runner you can use:
- Project > Settings > CI/CD > Edit Button of your runner > Check "Indicates whether this runner can pick jobs without tags"
  
  
<br><br>
Now, restart the runner that is running with the old config (i.e. with no gitlab instance being attached):
```
docker-compose down
```
