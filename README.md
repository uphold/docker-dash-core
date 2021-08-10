# uphold/docker-dash-core

A Dash Core docker image.

[![uphold/dash-core][docker-pulls-image]][docker-hub-url] [![uphold/dash-core][docker-stars-image]][docker-hub-url] [![uphold/dash-core][docker-size-image]][docker-hub-url] [![uphold/dash-core][docker-layers-image]][docker-hub-url]

## Tags

- `0.17.0.3`, `0.17`, `latest` ([0.17/Dockerfile](https://github.com/uphold/docker-dash-core/blob/master/0.17/Dockerfile))
- `0.16.0.1`, `0.16` ([0.16/Dockerfile](https://github.com/uphold/docker-dash-core/blob/master/0.16/Dockerfile))
- `0.15.0.0`, `0.15`  ([0.15/Dockerfile](https://github.com/uphold/docker-dash-core/blob/master/0.15/Dockerfile))
- `0.14.0.5-alpine`, `0.14-alpine`, `alpine` ([0.14/alpine/Dockerfile](https://github.com/uphold/docker-dash-core/blob/master/0.14/alpine/Dockerfile))
- `0.14.0.5`, `0.14`  ([0.14/Dockerfile](https://github.com/uphold/docker-dash-core/blob/master/0.14/Dockerfile))
- `0.12.2.3-alpine`, `0.12-alpine` ([0.12/alpine/Dockerfile](https://github.com/uphold/docker-dash-core/blob/master/0.12/alpine/Dockerfile))
- `0.12.2.3`, `0.12`  ([0.12/Dockerfile](https://github.com/uphold/docker-dash-core/blob/master/0.12/Dockerfile))

## What is Dash?
_from [dashwiki](https://github.com/dashpay/dash/wiki)_

Dash: A Privacy-Centric Crypto-Currency https://www.dash.org

## Usage

### How to use this image

This image contains the main binaries from the Dash Core project - `dashd`, `dash-cli` and `dash-tx`. It behaves like a binary, so you can pass any arguments to the image and they will be forwarded to the `dashd` binary:

```sh
$ docker run --rm -it uphold/dash-core \
  -printtoconsole \
  -regtest=1 \
  -rpcallowip=172.17.0.0/16 \
  -rpcpassword=bar \
  -rpcuser=foo
```

By default, `dashd` will run as user `dash` for security reasons and with its default data dir (`~/.dashcore`). If you'd like to customize where `dash-core` stores its data, you must use the `DASH_DATA` environment variable. The directory will be automatically created with the correct permissions for the `dash` user and `dash-core` is automatically configured to use it.

```sh
$ docker run --env DASH_DATA=/var/lib/dash --rm -it uphold/dash-core \
  -printtoconsole \
  -regtest=1
```

You can also mount a directory it in a volume under `/home/dash/.dashcore` in case you want to access it on the host:

```sh
$ docker run -v ${PWD}/data:/home/dash/.dashcore -it --rm uphold/dash-core \
  -printtoconsole \
  -regtest=1
```

You can optionally create a service using `docker-compose`:

```yml
dash-core:
  image: uphold/dash-core
  command:
    -printtoconsole
    -regtest=1
```

### Using RPC to interact with the daemon

There are two communications methods to interact with a running Dash Core daemon.

The first one is using a cookie-based local authentication. It doesn't require any special authentication information as running a process locally under the same user that was used to launch the Dash Core daemon allows it to read the cookie file previously generated by the daemon for clients. The downside of this method is that it requires local machine access.

The second option is making a remote procedure call using a username and password combination. This has the advantage of not requiring local machine access, but in order to keep your credentials safe you should use the newer `rpcauth` authentication mechanism.

#### Using cookie-based local authentication

Start by launch the Dash Core daemon:

```sh
❯ docker run --rm --name dash-server -it uphold/dash-core \
  -printtoconsole \
  -regtest=1
```

Then, inside the running `dash-server` container, locally execute the query to the daemon using `dash-cli`:

```sh
❯ docker exec --user dash dash-server dash-cli -regtest getmininginfo

{
  "blocks": 0,
  "currentblocksize": 0,
  "currentblockweight": 0,
  "currentblocktx": 0,
  "difficulty": 4.656542373906925e-10,
  "errors": "",
  "networkhashps": 0,
  "pooledtx": 0,
  "chain": "regtest"
}
```

In the background, `dash-cli` read the information automatically from `/home/dash/.dashcore/regtest/.cookie`. In production, the path would not contain the regtest part.

#### Using rpcauth for remote authentication

Before setting up remote authentication, you will need to generate the `rpcauth` line that will hold the credentials for the Dash Core daemon.
You can either do this yourself by constructing the line with the format `<user>:<salt>$<hash>` or use the official `rpcuser.py` script to generate this line for you, including a random password that is printed to the console.

Example:

```sh
❯ curl -sSL https://raw.githubusercontent.com/dashpay/dash/master/share/rpcuser/rpcuser.py | python - foo

String to be appended to bitcoin.conf:
rpcauth=foo:796d3d89ded5b826c7a4bf2ca8fe465$4cb1618e1552b414941783822b087b2df8c2b8bb1fa3dc441d9fa8f32d43e054
Your password:
Yec3WkzEpXGNFRQgTCsdKYp8HO11Z6DaoOY8BvV4YhE=
```

Note that for each run, even if the username remains the same, the output will be always different as a new salt and password are generated.

Now that you have your credentials, you need to start the Dash Core daemon with the `-rpcauth` option. Alternatively, you could append the line to a `dash.conf` file and mount it on the container.

Let's opt for the Docker way:

```sh
❯ docker run --rm --name dash-server -it uphold/dash-core \
  -printtoconsole \
  -regtest=1 \
  -rpcallowip=172.17.0.0/16 \
  -rpcauth='foo:796d3d89ded5b826c7a4bf2ca8fe465$4cb1618e1552b414941783822b087b2df8c2b8bb1fa3dc441d9fa8f32d43e054'
```

Two important notes:

1. Some shells require escaping the rpcauth line (e.g. zsh), as shown above.
2. It is now perfectly fine to pass the rpcauth line as a command line argument. Unlike `-rpcpassword`, the content is hashed so even if the arguments would be exposed, they would not allow the attacker to get the actual password.

You can now connect via `dash-cli` or any other [compatible client](https://github.com/uphold/dash-core). You will still have to define a username and password when connecting to the Dash Core RPC server.

To avoid any confusion about whether or not a remote call is being made, let's spin up another container to execute `dash-cli` and connect it via the Docker network using the password generated above:

```sh
❯ docker run --link dash-server --rm uphold/dash-core dash-cli -rpcconnect=dash-server -regtest -rpcuser=foo -rpcpassword='Yec3WkzEpXGNFRQgTCsdKYp8HO11Z6DaoOY8BvV4YhE=' getmininginfo

{
  "blocks": 0,
  "currentblocksize": 0,
  "currentblockweight": 0,
  "currentblocktx": 0,
  "difficulty": 4.656542373906925e-10,
  "errors": "",
  "networkhashps": 0,
  "pooledtx": 0,
  "chain": "regtest"
}
```

Done!


## Image variants

The `uphold/dash-core` image comes in multiple flavors:

### `uphold/dash-core:latest`

Points to the latest release available of Dash Core. Occasionally pre-release versions will be included.

### `uphold/dash-core:<version>`

Based on Alpine Linux with Berkeley DB 4.8 (cross-compatible build), targets a specific version branch or release of Dash Core.

## Supported Docker versions

This image is officially supported on Docker version 1.12, with support for older versions provided on a best-effort basis.

## License

[License information](https://github.com/dashpay/dash/blob/master/COPYING) for the software contained in this image.

[License information](https://github.com/uphold/docker-dash-core/blob/master/LICENSE) for the [uphold/dash-core][docker-hub-url] docker project.

[docker-hub-url]: https://hub.docker.com/r/uphold/dash-core
[docker-layers-image]: https://img.shields.io/imagelayers/layers/uphold/dash-core/latest.svg?style=flat-square
[docker-pulls-image]: https://img.shields.io/docker/pulls/uphold/dash-core.svg?style=flat-square
[docker-size-image]: https://img.shields.io/imagelayers/image-size/uphold/dash-core/latest.svg?style=flat-square
[docker-stars-image]: https://img.shields.io/docker/stars/uphold/dash-core.svg?style=flat-square
