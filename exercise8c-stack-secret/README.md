First we nee to create the secrets, otherwise the deployment will fail: `echo "jordi123" | docker secret create psql-pw -`

Start swarm: `docker stack deploy -c docker-compose.yml drupal`

Once is tested we can drop the swarm service:

```
docker service rm drupal_drupal
docker service rm drupal_postgres
docker service rm psql
```
