# Setting up services
Once we have everything ready, now the exciting moment: we can start adding services to our server! Since we have Docker running, we can easily start most of the services we want with Docker. In this chapter, I'm going to provide the docker-compose files needed to start certain services. Apart from that, you will also need to set up the path for the service as mentioned in the [last chapter](/build_a_personal_website_domain_https_and_reverse_proxy).

## Docsify
This tech note is running on [Docsify](https://docsify.js.org/). It is a very lightweight framework that allows you to build your website from just a few `.md` files and one `index.html` file.

### Gin server
Technically speaking, to run Docsify, you just need to serve that `index.html` file, with Nginx. However, here I serve the file with a simple [Gin](https://github.com/gin-gonic/gin) application, with just a `main.go` file:

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()

	r.Static("", "./var/www")

	if err := r.Run(":8080"); err != nil {
		panic(err)
	}
}
```

This serves the static files under `./var/www`. We are going to mount our Docsify `index.html` here. Then, run command to build that into a binary file:

```bash
GOOS=linux GOARCH=amd64 go build -o build/linux-amd64/bin/docsify main.go
```

This builds the binary file to `build/linux-amd64/bin/docsify`.

### Build Docker image
After that, we are going to build it into a Docker image. For that, we create a `Dockerfile` in that same directory:

```dockerfile
# Dockerfile

FROM scratch

WORKDIR /docsify

COPY ./build/linux-amd64 /docsify

EXPOSE 8080

CMD ["./bin/docsify"]
```

Then, build that with a tag `docsify-server` by running:

```bash
sudo docker build -t docsify:server .
```

This builds our Gin server into a Docker image.

### Start container with docker-compose
And then we can configure a container with that image:

```yaml
# docker-compose.yml
services:
  docsify-server:
    image: docsify-server:latest
    restart: unless-stopped
    volumes:
      - ./www/:/docsify/var/www/:ro
    ports:
      - "8083:8080"
```

This mounts `./www` to the directory we are serving, `/docsify/var/www`, and maps container's port `8080`, which is Gin server's port, to host's port `8083`. Then, run `sudo docker-compose up -d` to start the container.

### Docsify content
You can refer to [Docsify](https://docsify.js.org/) for a detailed tutorial on how to set up Docsify.

Note that if you want to use a [GitHub](https://github.com) repo for the website's content, remember to set the `basePath` param to `https://raw.githubusercontent.com/[your_repo]/[branch_name]/[root_path]`.

## Emby
I want to set up a private media repo that supports CarPlay, TV app, etc. [Emby](https://emby.media) is a good media repo that support all I need.

To set up Emby, you just need a simple docker-compose file:

```yaml
# docker-compose.yml
version: "2.3"
services:
  emby:
    image: emby/embyserver
    container_name: embyserver
    volumes:
      - ./programdata:/config # Configuration directory
      - ./music:/mnt/music # Media directory
      - ./movies:/mnt/movies # Media directory
    ports:
      - "8096:8096" # HTTP port
    restart: on-failure
```

In this docker-compose file, I mounted folders for music and movies. You can mount more if you need more media types. Start the container with `sudo docker-compose up -d`.

After that, access Emby, and finish the setups on the webpage. Then, you can upload your media files with `scp` command.

## FileBrowser
[FileBrowser](https://filebrowser.org) is a simple cloud drive that allows you to use your server as a cloud storage. The setup is also simple, just a docker-compose file:

```yaml
# docker-compose.yml
services:
  drive:
    image: filebrowser/filebrowser
    container_name: drive
    restart: unless-stopped
    volumes:
      - ./data:/srv
      - ./db/filebrowser.db:/database/database.db
      - ./config/.filebrowser.json:/.filebrowser.json
    ports:
      - 8082:80
```

Then, start the container with `sudo docker-compose up -d`, and you shall be able to use it.
