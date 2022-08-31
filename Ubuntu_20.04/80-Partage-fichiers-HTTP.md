Partage de fichiers HTTP
==========

http with caddy
```
$ apt install -y debian-keyring debian-archive-keyring apt-transport-https
$ curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
$ curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
```

```
$ apt update
$ apt install caddy
```
```
$ mkdir /var/www
$ mkdir /var/www/docs
$ mkdir /var/www/images

$ vi /etc/caddy/Caddyfile

:80 {
        # Set this path to your site's directory.
        root * /var/www

        # Enable the static file server.
        file_server
        log {
        output file /var/log/caddy/serv.local.lan-access.log {
                format console
                roll_size 10mb
                roll_keep 20
                roll_keep_for 720h
                }
        }
}

$ systemctl restart caddy

```


### Ajout de l'affichage des fichiers markdown

```
        # from https://github.com/dbohdan/caddy-markdown-site
        templates

        @media {
                path /favicon.ico
                path /media/*
        }
        @templates {
                path /templates/*
                not path /templates/*.css /templates/*.js
        }
        @markdown {
                path_regexp \.md$
        }
        @markdown_exists {
                file {path}.md
        }

        handle @media {
                file_server
        }

        handle @templates {
                error 403
        }
        handle @markdown {
                rewrite * /templates/index.html
        }
        handle @markdown_exists {
                map {path} {caddy_markdown_site.append_to_path} {
                        default extension
                }
                rewrite * /templates/index.html
        }

        handle_errors {
                file_server
                templates

                @markdown_index_exists_404 {
                        file {path}/index.md
                        expression `{http.error.status_code} == 404`
                }

                handle @markdown_index_exists_404 {
                        map {path} {caddy_markdown_site.append_to_path} {
                                default index
                        }
                        file_server {
                                status 200
                        }
                        rewrite * /templates/index.html
                }
                handle {
                        rewrite * /templates/error.html
                }
        }

```

```
$ mkdir /var/www/templates/
$ cd /var/www/templates/

$ wget https://raw.githubusercontent.com/dbohdan/caddy-markdown-site/main/templates/axist.min.css
$ wget https://raw.githubusercontent.com/dbohdan/caddy-markdown-site/main/templates/error.html
$ wget https://raw.githubusercontent.com/dbohdan/caddy-markdown-site/main/templates/footer.html
$ wget https://raw.githubusercontent.com/dbohdan/caddy-markdown-site/main/templates/head.html
$ wget https://raw.githubusercontent.com/dbohdan/caddy-markdown-site/main/templates/header.html
$ wget https://raw.githubusercontent.com/dbohdan/caddy-markdown-site/main/templates/index.html

```

```
# remplacer dans axist.min.css
code{white-space:nowrap
#par
code{white-space:wrap
```
