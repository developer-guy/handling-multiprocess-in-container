# handling-multiprocess-in-container
Handling Multi-Process in container using Supervisord init-system

# Test without init system
Let's display the content of the Dockerfile to see what is inside of the Dockerfile
```Dockerfile
FROM alpine
ENTRYPOINT ["sleep","120"]
```
Build docker image with the following command
```bash
$ docker image build -t without-init-system:latest -f without-init-system/ without-init-system
```
Run the docker image built above
```bash
# Don't forget to add --init flag
$ docker container run --rm --init --name without-init without-init-system:latest
```
> The --init flag inserts a tiny init-process into the container as the main process, and handles reaping of all processes when the container exits. Handling such processes this way is superior to using a full-fledged init process such as sysvinit , upstart , or systemd to handle process lifecycle within your container.

In the second terminal connect to the container to see what processes are working inside of the container
```bash
$ docker container exec -ti without-init sh

# As soon as you connect to the container, you should run "pstree -p" command to see processes that are working in the container
/ # pstree -p
init(1)---sleep(6)
```

# Test with init sytem - Handle multiple sleep process at once
Let's display the content of the Dockerfile to see what is inside of the Dockerfile
```Dockerfile
FROM alpine
RUN apk update && apk add --no-cache supervisor
RUN mkdir -p /etc/supervisor.d
COPY supervisord.conf /etc/supervisord.conf
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
```
Build docker image with the following command
```bash
$ docker image build -t with-init-system:latest -f with-init-system/ with-init-system
```
Run the docker image built above
```bash
# Don't forget to add --init flag
$ docker container run --rm --init --name with-init with-init-system:latest
```
In the second terminal connect to the container to see what processes are working inside of the container
```bash
$ docker container exec -ti without-init sh

# As soon as you connect to the container, you should run "pstree -p" command to see processes that are working in the container
/ # pstree -p
init(1)---supervisord(7)-+-sleep(9)
                         `-sleep(10)
```
To test it kill the one of the sleep process
```bash
/ # kill -9 
/ # pstree -p
init(1)---supervisord(7)-+-sleep(10)
```
