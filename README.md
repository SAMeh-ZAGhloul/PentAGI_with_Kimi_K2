# PentAGI Local Notes

## Official Repository

This local setup is based on the original PentAGI project:

- GitHub: https://github.com/vxcontrol/pentagi
- Project site: https://pentagi.com

The notes below document the working Docker setup and local fixes used in this copy.

## Correct Installation Steps

This is the working setup for running PentAGI with Moonshot / Kimi in Docker.

### 1. Configure Kimi in `.env`

Set Kimi to use the direct Moonshot API and leave the provider suffix empty in [`.env`](/Users/user/Downloads/pentagi/.env):

```env
KIMI_API_KEY=your_key_here
KIMI_SERVER_URL=https://api.moonshot.ai/v1
KIMI_PROVIDER=
```

Important:
Do not set `KIMI_PROVIDER=moonshot` for this setup. That can make PentAGI request the wrong model name.

### 2. Use the local Moonshot provider file

In [`docker-compose.yml`](/Users/user/Downloads/pentagi/docker-compose.yml), mount the local Moonshot config into the container:

```yaml
- ./example.moonshot.provider.yml:/opt/pentagi/conf/moonshot.provider.yml
```

### 3. Use a supported Kimi model in the provider config

In [`example.moonshot.provider.yml`](/Users/user/Downloads/pentagi/example.moonshot.provider.yml), use `kimi-k2-0905-preview` for these roles:

- `simple`
- `simple_json`
- `installer`
- `pentester`

That is the model setup that worked successfully here.

### 4. Avoid host `DEBUG` variable conflicts

In [`docker-compose.yml`](/Users/user/Downloads/pentagi/docker-compose.yml), use:

```yaml
- DEBUG=${PENTAGI_DEBUG:-false}
```

instead of:

```yaml
- DEBUG=${DEBUG:-false}
```

This avoids conflicts if your shell or IDE already exports `DEBUG=release` or another non-boolean value.

### 5. Recreate the container

After changing config mounts, recreate the PentAGI container:

```bash
docker compose up -d --force-recreate pentagi
```

Using only `docker compose restart pentagi` may not apply the new bind mount configuration.

### 6. Create a new flow

After the container comes back up:

- open PentAGI
- create a brand new flow
- do not reuse older failed flows

## Why These Steps Matter

The main failure before the fix was:

```text
API returned unexpected status code: 404: Not found the model moonshot/kimi-k2-turbo-preview or Permission denied
```

This happened because PentAGI was effectively using the wrong model path and an unsuitable bundled Moonshot config for this setup.

## Verification

The setup was confirmed working when:

- `KIMI_PROVIDER` inside the running container was empty
- `/opt/pentagi/conf/moonshot.provider.yml` inside the container used `kimi-k2-0905-preview`
- PentAGI started successfully after `docker compose up -d --force-recreate pentagi`
- a new flow was created successfully

## Remaining Note

You may still see this startup warning:

```text
failed to create embedder 'openai'
```

That is separate from the Kimi installation fix, but it may affect embeddings or memory-related features later.
