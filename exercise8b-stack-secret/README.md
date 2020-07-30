How to create a secret:

- Create file with the content of the secret. Lets create a file called *psql_user.txt*

- Add the secret to Docker Swarm. `docker secret create psql_user psql_user.txt`

- Create password from input Cmd Line `echo "myDBpassWORD" | docker secret create psql_pass -`

Both methods have some security issues:

- File stored in hard disk

- Password stored in the bash history, so someone with root access can get the password

We can see the secrets with `docker secret ls`. Not the content of the secret, only the secret name


```
 docker service create --name psql --secret psql_user --secret psql_pass -e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass -e POSTGRES_USER_FILE=/run/secrets/psql_user postgres.


docker service ps psql
```

If we check the content of the container, wil find the secrets in the location `/run/secrets/` obviously this is a security issue.
