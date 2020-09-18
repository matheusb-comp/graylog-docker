# Graylog docker stack

## Docker secrets

Graylog need some [configurations](https://docs.graylog.org/en/3.3/pages/getting_started/configure.html)
for security, and this stack expect them to be set using [docker secrets](https://docs.docker.com/engine/swarm/secrets/).

- `password_secret`

Secret used for password encryption and salting.
The docs suggest generating a random password with `pwgen -N 1 -s 96`.

``` sh
# Create the secret named "graylog_password_secret"
$ printf 'RandomPassword' | \
  docker secret create "graylog_password_secret" -
```

- `root_password_sha2`

SHA-2 hash of the master password used for the initial login.

``` sh
# Generate the password SHA-2 hash
$ printf 'password' | sha256sum | awk '{ print $1 }'
5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8

# Create the secret named "graylog_root_password_sha2"
$ printf '5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8' | \
  docker secret create "graylog_root_password_sha2" -
```

## Host configuration

ElasticSearch since version 5.0 executes strict bootstrap checks when running
in [production mode](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/system-config.html#dev-vs-prod).
Since the docker image has the configuration `network.host` set to `0.0.0.0`,
unless `discovery.type` is set to `single-node`, some host configurations are
required.

### Virtual memory

From the [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/vm-max-map-count.html):

> Elasticsearch uses a mmapfs directory by default to store its indices.
The default operating system limits on mmap counts is likely to be too low,
which may result in out of memory exceptions.

To increase the limit:

``` sh
sudo sysctl -w vm.max_map_count=262144
```

Be aware of the [implications](https://www.suse.com/support/kb/doc/?id=000016692)
of increasing `vm.max_map_count` in the host machine.
