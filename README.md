# docker-dnsmasq

This is a [dnsmasq][dnsmasq] Docker image forked from [andyshinn][andyshinn]. It is only 5 MB in size and provides an `entrypoint` to the `dnsmasq` binary. Which is suitable for running as a service in Docker swarm mode.

## Usage

The latest image uses dnsmasq 2.78 based on Alpine 3.7.

### Entrypoint

The entrypoint is

```
 dnsmasq -k --log-facility - -u root
```

The option `-k` keeps dnsmasq running in the foreground enabling docker health checks. The second option `--log-facility -` routes logging to stdout. To avoid having to set linux capabilities, `dnsmasq` is told to run as `root` instead of `nobody` by setting the parameter `-u root`. 

### Deploy as a service

```
docker service create --name dnsmasq \
	--publish 53:53/tcp --publish 53:53/udp \
	ntim/dnsmasq:latest
```

### Configuration

Per default, dnsmasq looks for configuration files in the `/etc/dnsmasq.d/` directory. The following configuration file overwrites DNS requests to `example.com` and its subdomains (useful for SNI routing) to the ip `192.168.99.99`

```
cat <<EOF > dnsmasq_example.conf
address=/example.com/192.168.99.99
EOF
```

Add docker configuration

```
docker config create dnsmasq_example.conf example.conf
```

and deploy the service with 

```
docker service create --name dnsmasq \
	--publish 53:53/tcp --publish 53:53/udp \
	--config source=dnsmasq_example.conf,target=/etc/dnsmasq.d/example.conf \
	ntim/dnsmasq:latest
```

mounting the configuration.

[dnsmasq]: http://www.thekelleys.org.uk/dnsmasq/doc.html
[andyshinn]: https://github.com/andyshinn/docker-dnsmasq