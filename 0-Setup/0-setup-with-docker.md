Keep everything in one project folder and bind-mount the data directory into it. Moving to another machine is then just copying that folder. The official Hermes Docker docs say the container keeps all its state in `/opt/data`. That corrects my earlier `docker run` example, which used `/root/.hermes`. Use `/opt/data`.

## 1. Folder and compose file

```bash
mkdir -p ~/hermes/data && cd ~/hermes
```

`docker-compose.yml`:

```yaml
services:
  hermes:
    image: nousresearch/hermes-agent:latest   # pin a specific tag once it works
    container_name: hermes
    restart: unless-stopped
    command: gateway run
    ports:
      - "127.0.0.1:8642:8642"   # gateway API, local only
    volumes:
      - ./data:/opt/data        # config, sessions, memory, skills, logs
```

I bound the port to `127.0.0.1` so the API isn't exposed publicly. Remove that prefix only if you need outside access, and put auth or a reverse proxy in front.

## 2. First-time setup

```bash
docker compose run --rm hermes setup
```

In the wizard, choose the custom endpoint and enter `https://backend.sovereigneg.com/v1`, your key, and `glm-5.3`. Everything is saved into `./data`. You can also edit `data/config.yaml` with the `model:` block from earlier.

## 3. Run and check

```bash
docker compose up -d
docker compose logs -f
docker compose exec hermes hermes doctor
```

## 4. Upgrade without losing data

```bash
docker compose pull
docker compose up -d
```

Your data lives in `./data`, so recreating the container doesn't touch it.

## 5. Migrate to a new machine

On the old machine:

```bash
cd ~/hermes
docker compose down            # stop first so files are consistent
cd ~ && sudo tar czf hermes-backup.tar.gz hermes
scp hermes-backup.tar.gz user@new-server:~
```

On the new machine:

```bash
sudo tar xzf hermes-backup.tar.gz
cd hermes
docker compose up -d
```

Use `sudo` for `tar` so file ownership and permissions are preserved. That matters because the container may write files as a different user.

## Tips

- **Backups:** run the `tar` command on a cron job. A scheduled copy of `./data` is your whole backup.
- **Use a bind mount, not a named volume.** A bind mount is a plain folder you can see, copy, and `tar`.
- **Pin the image tag** after it works, so a migration or rebuild doesn't pull a surprise new version. You can pin `latest` to a specific version tag from Docker Hub.
- **Multiple agents:** run another copy of the stack with a different folder, for example `~/hermes-work/data`. The docs describe this as separate, independent agents.

## How To connect to it and give command inside container 


1. Step-by-Step Connection (Recommended)
Step 1: Find your container's Name or ID
Before connecting, you need to know which container you want to target. Run this command to list your active containers
```bash
docker ps
```
Look at the CONTAINER ID or NAMES column to copy the target identifier.

Step 2: Connect to the interactive shell
Run the following command, replacing <container_name_or_id> with your actual data:

```bash
docker exec -it <container_name_or_id> /bin/bash

```
• -i (interactive): Keeps the standard input (stdin) open so you can type commands.
• -t (tty): Allocates a pseudo-terminal, rendering a command prompt like a regular terminal.
Step 3: Fallback if bash isn't installed
Minimalist Docker images (like Alpine Linux) do not come with bash built-in. If the previous command throws an error, use sh or ash instead:

```bash
docker exec -it <container_name_or_id> sh

```
how i update container

Run these from the folder that holds your docker-compose.yml (e.g. ~/hermes):

```bash
docker compose pull
docker compose up -d
```
pull downloads the newest image, and up -d 
recreates the container with it. Your data is safe because it lives in ./data, outside the container.

Then check that it came up fine:

```bash
docker compose logs --tail 50
docker compose exec hermes hermes doctor
```
Optionally, clean up old images afterwards:

```bash
docker image prune
```

Before a big update, back up the data first (stop the container, then sudo tar czf hermes-backup.tar.gz data). Then you can roll back if something breaks.

If you pinned a version tag (e.g. image: nousresearch/hermes-agent:1.2.3), pull won’t fetch anything newer. Edit the tag in docker-compose.yml to the new version first, then run the same two commands.

To roll back, set the image back to the previous tag and run docker compose up -d again. This is the main reason to pin versions instead of using latest.

For automatic updates, you can add Watchtower, but I’d skip it for an agent you rely on. Manual updates let you read the release notes and back up first.


