# Migrating GitLab from a VM or physical server to a Docker container

Migrating GitLab from a VM or physical server to a Docker container involves several steps. The process generally consists of creating a backup of your existing GitLab instance, setting up a new Docker container with the desired version of GitLab, and then restoring the backup into the new container. 

knowing that the two GitLab versions must be similar. The old server and the new container.

## Prepare the existing GitLab instance:

Ensure that all repositories, databases, and attachments are up to date and synchronized. and block any external access to the server.

## Create a backup 
Generate a backup of your GitLab instance, which includes the Git repositories, databases, and configuration files. You can use the gitlab-rake command to create a backup:

```bash
sudo gitlab-rake gitlab:backup:create
```

The backup file will be stored in the default backup directory (usually /var/opt/gitlab/backups/).

## Set up the Docker container

Install Docker on the new server or VM where you want to run GitLab in a container. Pull the GitLab Docker image for the desired version.

```bash
docker pull gitlab/gitlab-ce:12.2.4
```

Run a Docker compose file that contains all the Directoires that must be mounted to the Docker container and the settings that must be passed to the container.



```bash
cat <<EOF>> docker-compose.yaml

version: '3.7'
services:
  gitlab-ce-12-2-4:
    image: 'gitlab/gitlab-ce:12.2.4-ce.0'
    restart: always
    container_name: gitlab-ce-12-2-4
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://example.com'
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
    volumes:
      - ./config:/etc/gitlab
      - ./logs:/var/log/gitlab
      - ./data:/var/opt/gitlab
EOF
```

```bash
docker compose up -d
```

The two environment variables nginx['listen_port'] and nginx['listen_https'] = false. I added them because I'm behind a reverse proxy. 


## Restore the backup:

Copy the backup file (e.g., 1578943210_2023_12_12_12.2.4_gitlab_backup.tar) to the host machine where the GitLab container is running and copy it inside the docker container or copy the backup file on any volume on the host that's mounted inside the docker container.

Access the running GitLab container using the docker exec command:

```bash
docker exec -it gitlab-ce-12-2-4 bash
```
Inside the container, run the restore command:
```bash
gitlab-rake gitlab:backup:restore BACKUP=1578943210_2023_12_12_12.2.4_gitlab_backup.tar
```

After the restore process completes, access your new GitLab instance by navigating to the appropriate URL in your web browser. Log in and ensure that all your repositories, users, settings, and data have been successfully migrated.
