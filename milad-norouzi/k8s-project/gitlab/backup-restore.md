# Backup

For GitLab, you can create a backup using the following command:

```bash
docker exec -it CONTAINER_NAME gitlab-backup create
```

Then you should copy the `/data/gitlab/config/gitlab-secrets.json` and `/data/gitlab/data/gitlab.rb` manually.

**Take care to copy it manually**

The backup will be stored in the `backup` directory of the container. If you persist the data using a volume, you can access the backup files on your host machine.

In our case, the backup files are stored in the `/data/gitlab/data/backups` directory on the host machine, which is mapped to the container's `/var/opt/gitlab/backups` directory.

# Restore

For restoring a GitLab backup, It's better to exec on container and run the restore command directly.

Copy the backup file to the container's backup directory.

```bash
cp backup.tar /data/gitlab/data/backups/
```

Then, exec into the GitLab container and run the following command:

```bash
docker exec -it CONTAINER_NAME /bin/bash

chown git:git /var/opt/gitlab/backups/backup.tar.gz

# Stop the processes that are connected to the database
docker exec -it <name of container> gitlab-ctl stop puma
docker exec -it <name of container> gitlab-ctl stop sidekiq

# Verify that the processes are all down before continuing
docker exec -it <name of container> gitlab-ctl status

```

Now, you can restore the backup using the following command:

```bash

# Don't need to specify the prifix and gitlab name.
docker exec -it <name of container> gitlab-backup restore BACKUP=11493107454_2018_04_25_10.6.4-ce

# OR
docker exec -it <name of container> /bin/bash
gitlab-backup restore BACKUP=11493107454_2018_04_25_10.6.4-ce
```

After the restore, you MUST copy the `gitlab-secrets.json` and `gitlab.rb` files back to their respective locations:

```bash
cp gitlab-secrets.json /data/gitlab/config/gitlab-secrets.json
cp gitlab.rb /data/gitlab/config/gitlab.rb
```

Finally, restart the GitLab services:

```bash
docker exec -it <name of container> gitlab-ctl start

# OR

docker restart <name of container>

# Check GitLab
docker exec -it <name of container> gitlab-rake gitlab:check SANITIZE=true
```
