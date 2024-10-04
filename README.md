# Repo to demonstrate docker-compose apparent bug

I have two `docker-compose.yml` files, each in a different folder, representing
a frontend and backend service. Each binds its `./src` folder to `/app/src` in
its respective container, apparently.

So we expect the following binding.

```
backend/src/backend.txt   -> /app/src/backend.txt
frontend/src/frontend.txt -> /app/src/frontend.txt
```

... which is what we see if we run

```bash
docker compose \
  -p docker-volume-bug-backend \
  -f backend/docker-compose.yml \
  up;

docker compose \
  -p docker-volume-bug-frontend \
  -f frontend/docker-compose.yml \
  up;

docker compose -p docker-volume-bug-backend down;
docker compose -p docker-volume-bug-frontend down;
```

But when we run these as a single command:


But when we run:

```bash
docker compose \
  -p docker-volume-bug \
  -f backend/docker-compose.yml \
  -f frontend/docker-compose.yml \
  up;

docker compose -p docker-volume-bug down;
```

We see that both containers have `backend/src/` mounted at `/app/src`:

```
backend-1   | /app/src/backend.txt
frontend-1  | /app/src/backend.txt
```

There is a workaround:

```bash
env DOCKER_VOLUME_BUG_FRONTEND_PATH=../frontend/src \
  docker compose \
  -p docker-volume-bug \
  -f backend/docker-compose.yml \
  -f frontend/docker-compose.yml \
  up;
docker compose -p docker-volume-bug down;
```

Weirdly, however, it doesn't work when `DOCKER_VOLUME_BUG_BACKEND_PATH` is set.
