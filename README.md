# Icinga2 / Nagios - CHECK_DOCKERAPI

Scripts for monitoring docker containers using the docker API.

Dependencies: curl, jq and awk

## check_dockerapi

This script connects to the docker API and retrieves json from the URL `/container/json?all=true`. It then iterates through all the states and returns a count of running and not running containers. It also returns all of the names of the containers along with the state and how long since it was created.

It supports connecting to the API using a client certificate, which is the suggested minimum level of authentication required  for the exposed API.

Details of the API calls can be found here: [https://docs.docker.com/engine/api/v1.40/](https://docs.docker.com/engine/api/v1.40/)

How to go about protecting the API using client certificates can be found here: [https://docs.docker.com/engine/security/https/](https://docs.docker.com/engine/security/https/)

```shell
$ ./check_dockerapi -h

check_dockerapi Revision 1.0.0 - Checks Docker Container Status from Docker API for Icinga2

Usage: check_dockerapi

  -h           Show this page

  -H | --host  Host name / IP of Docker API server (required)

  -P | --port  Port Number of Docker API Service

  --https      Use HTTPS requires cert, key and cacert set

    --cert       Client certificate (pem)

    --key        Client key (pem)

    --cacert     CA cert (pem)

  --include    String of container names to include

  --exclude    String of container names to exclude

Usage: check_dockerapi --help
```

Example:

Ensure it runs as the user nagios and has certificates that the user can read.

```shell
sudo -u nagios ./check_dockerapi -H myhost -P 2376 --https --cert /var/lib/nagios/icinga2-cert.pem --key /var/lib/nagios/icinga2-key.pem --cacert /var/lib/nagios/ca.pem
```

### custom-commands.conf

Add into icinga2 config:

```text
object CheckCommand "check_dockerapi" {
  command = [ PluginDir + "/check_dockerapi" ]
  arguments = {
    "-H" = "$dockerapi_host$"
    "-P" = "$dockerapi_port$"
    "--include" = "$dockerapi_include$"
    "--exclude" = "$dockerapi_exclude$"
    "--https" = {
      set_if = "$dockerapi_https$"
    }
    "--cert" = "$dockerapi_cert$"
    "--key" = "$dockerapi_key$"
    "--cacert" = "$dockerapi_cacert$"
  }
}
```

## check_dockerapi_stats

Like it's parent this connects to the docker API, but this time returns performance metrics from a specific container.

```shell
$ ./check_dockerapi_stats -h                                                                                                                                                               [±master ●]

check_dockerapi_stats Revision 1.0.0 - Checks Docker Container Status from Docker API for Icinga2

Usage: check_dockerapi_stats

  -h               Show this page

  -H | --host      Host name / IP of Docker API server (required)

  -P | --port      Port Number of Docker API Service

  --https      Use HTTPS requires cert, key and cacert set

    --cert       Client certificate (pem)

    --key        Client key (pem)

    --cacert     CA cert (pem)

  -C | --container Name of container to inspect

Usage: check_dockerapi_stats --help
```

Example:

```shell
sudo -u nagios ./check_dockerapi_stats -H myhost -P 2376 --https --cert /var/lib/nagios/icinga2-cert.pem --key /var/lib/nagios/icinga2-key.pem --cacert /var/lib/nagios/ca.pem --container mycontainer_db_1
```

## Caveats

If you use HTTPS, you need to ensure that the client certificate can be verified against the server certificate. This means you __MUST__ ensure that the `SubjectAlternativeName` of the server certificate matches the `-H` host name parameter you are using either by IP address or name, but the certificate must have the same name included in it's SAN.
