# docker-study

Learning Docker by containerizing my own course projects.

| Step | Repo | What it teaches | Status |
|---|---|---|---|
| 0 | (this repo) | install, client vs daemon, image vs container, kernel sharing | done 2026-10-14 |
| 1 | cassandra-exploration-activity | running a ready image, ports, named volumes, healthchecks | |
| 2 | --kafka-wikipedia-pipeline | first Dockerfile, layer caching, `localhost` inside a container | |
| 3 | --kafka-wikipedia-pipeline | compose, networks, Kafka's two listeners, `--scale` | |
| 4 | spark-kafka-activity | JVM in a container, memory limits, image size, caching jars | |
| 5 | css182-realtime-pipeline | full stack: 7 services, init container, compose vs the `css182` script | |
| 6 | any | multi-stage builds, pushing to a registry, CI | |

Notes per step are in [NOTES.md](NOTES.md). The Dockerfiles for each app live in that app's repo on a
local `docker-study` branch (not pushed), because a Dockerfile builds from the code next to it.
