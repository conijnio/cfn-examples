# API Gateway with OAuth2

This example includes an example that allows you to call the API Gateway using OAuth2 scopes.

## Deploy the solution

* Copy the `example.env` to `.env`, see: `cp example.env .env`.
* Deploy the solution, see: `make deploy`
* Update the `.env` file with the `CLIENT_ID` and `CLIENT_SECRET` from the console.

## Perform API Call

First, you need to request the `access_token`:

```shell
make read-token
```

Now, you are able to call the API using:

```shell
make hello
```

You can now try and post to the API: 

```shell
make post-hello
```

Switch the scopes to read/write:

```shell
make read-write-token
```

Perform the API call again:

```shell
make post-hello
```