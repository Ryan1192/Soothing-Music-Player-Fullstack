# Soothing-Music-Player-Fullstack

A **Soothing-Music-Player-Fullstack** website with essential functionality, such as user sign up, login, and a player interface. This application is implemented using a `django` backend and a `react.js` frontend.

This project already includes some initial user data in the database for experimentation.

# Setup Guide

## Prerequisites

First, you'll need these common development tools:

- `pip` - a package manager for `python3`, utilized by Django.
- `node.js` - a JavaScript runtime used by many web projects (including the frontend here).
- `yarn` (or `npm`) - a package manager for `node.js`.

Install them with these straightforward commands:

**MacOS**

```zsh
brew install python3 node yarn
# This also installs pip
```

**Linux**

```bash
sudo apt update
sudo apt install python3-pip
sudo apt install nodejs
sudo apt install yarn
```

**Django prerequisites** are detailed in `django_server/requirements.txt`.
Install them using pip
(this step is advised to be done inside a [Python virtual environment](https://docs.python.org/3/tutorial/venv.html)):

```bash
pip install -r ./django_server/requirements.txt
# This installs required packages like django, djangorestframework-jwt, uWSGI, django-cors-headers
```

**Frontend prerequisites** are outlined in `frontend/package.json`.
Install them using yarn (or npm):

```bash
cd frontend
yarn install
# This installs required packages like bootstrap, react, redux
```

## Development Environment

Configure the appropriate flags for development:

**`./django_server/django_server/settings.py`**

```python
...
DEBUG = True
...
```

Run the backend and frontend servers independently:

```bash
python ./django_server/manage.py runserver
# This starts the Django server (backend)
```

```bash
cd ./frontend
yarn start
# This starts the React application server (frontend)
```

## Production Environment (for a Linux server)

First, build the frontend and incorporate it into the backend's static files:

```bash
cd frontend && yarn build
cd .. && python django_server/manage.py collectstatic
```

You will need to get the IP address or Fully Qualified Domain Name (FQDN) for the machine to access external endpoints. It's recommended to follow this [Nginx installation tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04).

Install and start Nginx:

```bash
sudo apt install nginx
sudo systemctl start nginx
```

Adjust the `./django_server/uwsgi.ini` file with the project's absolute path:
**`./django_server/uwsgi.ini`**

```conf
... # for example "/home/username/Soothing-Music-Player-Fullstack/django_server"
chdir={PROJECT_PATH}/Soothing-Music-Player-Fullstack/django_server
wsgi-file={PROJECT_PATH}/django_server/django_server/wsgi.py
...
```

At this stage, we must inform `nginx` about which `uwsgi_params` to use.
To accomplish this, the `nginx.conf` file needs to be modified (you can find more information about Nginx + uWSGI + Django setup [here](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)).

**`/etc/nginx/conf.d/music_player.conf`**

```conf
upstream django {
  server unix:///var/run/music_player_app.sock; # for a file socket
}

# server configuration
server {
  # the port your site will be accessible on
  listen       80;
  # the domain name it will respond to
  server_name {SERVER_EXTERNAL_IP}; # replace with your machine's IP address or FQDN, e.g., example.com
  charset      utf-8;

  # maximum upload size
  client_max_body_size 75M;   # adjust as needed

  location /static {
    alias {PROJECT_PATH}/Soothing-Music-Player-Fullstack/django_server/static; # your Django project's static files - modify as required
  }

  # Finally, direct all non-media requests to the Django server.
  location / {
    uwsgi_pass  django;
    include     {PROJECT_PATH}/django_server/uwsgi_params; # the uwsgi_params file you configured
  }
}
```

Alternatively, you can modify `./music_player.conf` and then move it to the Nginx configurations directory:

```bash
vim ./music_player.conf # edit the .conf file as indicated above
sudo mv ./music_player.conf /etc/nginx/conf.d/music_player.conf
```

Finally, restart Nginx:

```bash
sudo systemctl restart nginx # Corrected from ngincx to nginx
```

At this point, you should be able to access the website by visiting your server's external IP address.

To halt Nginx:

```bash
sudo systemctl stop nginx
```
