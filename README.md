# ![](https://raw.githubusercontent.com/yudai/gotty/master/resources/favicon.png) DockerTTY - Terminal over web for remote docker container

DockerTTY is a simple command line tool that provides terminal over web for remote docker container.

DockerTTY is based on the work of [GoTTY](https://github.com/yudai/gotty), but with following features:
- Provides terminal over web for remote docker container
- Allow to run command in the container like `docker exec -it ${containerId} sh`
- Do not leave legacy processes in container

# Installation

If you have a Go language environment, you can install DockerTTY with the `go get` command.

```sh
$ go get github.com/supereagle/dockertty
```

# Usage

```
Usage: dockertty [options] <command> [<arguments...>]
```

Running `dockertty` will start a web server. Open the URL with the specified container id on your web browser, DockerTTY will run `docker exec -it ${containerId} sh`, then you can run command on your terminal as if you are running them in the container.

By default, DockerTTY starts a web server at port 8080, and permits clients to write to the TTY.

## Options

```
--address, -a                                                IP address to listen [$GOTTY_ADDRESS]
--port, -p "8080"                                            Port number to listen [$GOTTY_PORT]
--permit-write, -w                                           Permit clients to write to the TTY (BE CAREFUL) [$GOTTY_PERMIT_WRITE]
--credential, -c                                             Credential for Basic Authentication (ex: user:pass, default disabled) [$GOTTY_CREDENTIAL]
--random-url, -r                                             Add a random string to the URL [$GOTTY_RANDOM_URL]
--random-url-length "8"                                      Random URL length [$GOTTY_RANDOM_URL_LENGTH]
--tls, -t                                                    Enable TLS/SSL [$GOTTY_TLS]
--tls-crt "~/.gotty.crt"                                     TLS/SSL certificate file path [$GOTTY_TLS_CRT]
--tls-key "~/.gotty.key"                                     TLS/SSL key file path [$GOTTY_TLS_KEY]
--tls-ca-crt "~/.gotty.ca.crt"                               TLS/SSL CA certificate file for client certifications [$GOTTY_TLS_CA_CRT]
--index                                                      Custom index.html file [$GOTTY_INDEX]
--title-format "GoTTY - {{ .Command }} ({{ .Hostname }})"    Title format of browser window [$GOTTY_TITLE_FORMAT]
--reconnect                                                  Enable reconnection [$GOTTY_RECONNECT]
--reconnect-time "10"                                        Time to reconnect [$GOTTY_RECONNECT_TIME]
--timeout "0"                                                Timeout seconds for waiting a client (0 to disable) [$GOTTY_TIMEOUT]
--max-connection "0"                                         Set the maximum number of simultaneous connections (0 to disable)
--once                                                       Accept only one client and exit on disconnection [$GOTTY_ONCE]
--permit-arguments                                           Permit clients to send command line arguments in URL (e.g. http://example.com:8080/?arg=AAA&arg=BBB) [$GOTTY_PERMIT_ARGUMENTS]
--close-signal "1"                                           Signal sent to the command process when gotty close it (default: SIGHUP) [$GOTTY_CLOSE_SIGNAL]
--config "~/.gotty"                                          Config file path [$GOTTY_CONFIG]
--version, -v                                                print the version
```

> For more detailed usage, please refer to [GoTTY Usage](https://github.com/yudai/gotty#usage).

## Advanced Usage

To login and exec command in containers in [Kubernetes](https://kubernetes.io/) cluster, DockerTTY can be deployed as an agent on every Kubernetes nodes through [Daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    app: dockertty-agent
  name: dockertty-agent
spec:
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: dockertty-agent
    spec:
      restartPolicy: Always
      hostNetwork: true
      containers:
      - image: ${DOCKERTTY_AGENT_IMAGE}
        name: dockertty-agent
        resources:
          limits:
            cpu: 800m
            memory: 1Gi
          requests:
            cpu: 400m
            memory: 500Mi
        ports:
          - name: agent-port
            hostPort: 8080
            containerPort: 8080
        volumeMounts:
        - mountPath: /var/run/docker.sock
          name: dockersock
        - mountPath: /usr/bin/docker
          name: dockerclient
        - mountPath: /usr/lib64/libltdl.so.7
          name: libltdl
      volumes:
      - hostPath:
          path: /var/run/docker.sock
        name: dockersock
      - hostPath:
          path: /usr/bin/docker
        name: dockerclient
      - hostPath:
          path: /usr/lib64/libltdl.so.7.3.0
        name: libltdl
```

# Clients

## Web Browser

Open the URL like `http://localhost:8080?containerId=c28` in web browser.

## GoTTY Client

[GoTTY Client](https://github.com/moul/gotty-client) is a terminal client for GoTTY. It can also be used as a terminal client for DockerTTY.

```sh
$ ./gotty-client http://localhost:8080?containerId=c28
```

## Architecture

GoTTY uses [hterm](https://groups.google.com/a/chromium.org/forum/#!forum/chromium-hterm) to run a JavaScript based terminal on web browsers. GoTTY itself provides a websocket server that simply relays output from the TTY to clients and receives input from clients and forwards it to the TTY. This hterm + websocket idea is inspired by [Wetty](https://github.com/krishnasrinivas/wetty).

# License

The MIT License
