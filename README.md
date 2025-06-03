<div align="center">
<div><img src=".github/images/cws.png" width="312" alt="cws2"></div>

**cws2**, or **conworkshop 2**, is a website to bring conlangers together.

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/m5ka/cws2/test.yaml?label=tests)
![GitHub contributors](https://img.shields.io/github/contributors/m5ka/cws2)
![Python version: >= 3.12](https://img.shields.io/badge/python-%3E%3D%203.12-blue?logo=python&logoColor=white)
[![Django version 5.1](https://img.shields.io/badge/django-5.1-green?logo=django)](https://docs.djangoproject.com/en/5.1/)
[![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)
</div>

## 🐻 About
This project is the [Django](https://www.djangoproject.com/) successor to the original PHP ConWorkShop. It aims to be a modern, stable and more future-proof version of the site whilst maintaining the spirit of ConWorkShop's vibrant past and community.

----

## 📖 Table of Contents

* [🏄 Setup](#-setup)
  * [0. Requirements](#0-requirements)
  * [1. Clone Repository](#1-clone-repository)
  * [2. Create Virtual Environment](#2-create-virtual-environment)
  * [3. Install Dependencies](#3-install-dependencies)
  * [4. Configuring](#4-configuring)
  * [5. Run Server](#5-run-server)
* [📌 Deploying](#-deploying)
  * [🏢 Server](#-server)
  * [🖼 Assets](#-assets)
* [🤖 Development](#-development)
  * [🎨 Compiling Assets](#-compiling-assets)
  * [🧪 Testing](#-testing)
  * [🧩 Migrations](#-migrations)
  * [🍓 Code Style](#-code-style)
* [🤝 Contributing](#-contributing)
* [📚 License](#-license)
* [🌳 Acknowledgements](#-acknowledgements)

## 🏄 Setup
### 0. Requirements
* [Python](https://www.python.org/downloads/) (3.12 or later)
* [Poetry](https://python-poetry.org/docs/)
* [PostgreSQL](https://www.postgresql.org/download/)

This application runs on both Linux/WSL and Windows. However, Linux would be used in production.

Depending on your system, Python is installed as either `python`, `python3`, or `py`. The instructions below assumes `python3` on Ubuntu or Debian.

### 1. Clone Repository
Clone this repository using `git clone`, and go to the newly created directory:
```bash
git clone https://github.com/m5ka/cws2.git
cd cws2
```
Alternatively, download this repository as a ZIP from GitHub by pressing the **<> Code** button and unzip the downloaded file.

### 2. Create Virtual Environment
Using a [virtual environment](https://docs.python.org/3/tutorial/venv.html) is recommended to keep your Python libraries clean. We recommend using [pyenv](https://github.com/pyenv/pyenv) alongside [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv).
```bash
pyenv install 3.12                      # If you already have Python 3.12 installed in pyenv, you can skip this step
pyenv virtualenv 3.12 cws2
pyenv activate cws2
```

Alternatively, using Python's builtin `venv` directly:
```bash
python3 -m venv .venv
source .venv/bin/activate               # Powershell: .venv\Scripts\Activate.ps1
```

### 3. Install Dependencies
Install all packages through Poetry that you'll need for cws2, as well as for local development and testing:
```bash
poetry install --with dev,test
```

**Note (Linux)**: You might need to install packages `python3-dev` and `libpq-dev`, required by Python library `psycopg2` to connect to PostgreSQL from Python:
```bash
sudo apt update
sudo apt install python3-dev libpq-dev
```

### 4. Configuring
**Note (Windows)**: These instructions use `make`. Alternatively, you can enter the commented (#) commands, replacing `python3` accordingly. Windows users can run `make` using [w64devkit](https://github.com/skeeto/w64devkit/releases).

Set up your `.env` file for application setup:
```bash
make env                                # cp .env.example .env
```
Edit this file to update your configuration using your editor of choice, such as VS Code or (Neo)vim.

Make sure to fill in `DATABASE_URL` as it is used to connect to your database. The format is `postgres://user:password@host:port/database`. Also ensure that the specified PostgreSQL user has the Create DB permission, or you might get an error during tests.

Once you've finished configuring your environment, you can migrate the database using:
```bash
make migrate                            # python3 manage.py migrate
```

### 5. Run Server
Finally you should be able to run the server. Woohoo! 🎉
```bash
make serve                              # python3 manage.py runserver
```

**Note**: You might need to compile SASS to CSS first for styles to load properly:
```bash
# python3 manage.py sass cws2/static/scss/base.scss cws2/static/css/base.css -g -t compressed
make sass
```

**WARNING**: `runserver` is not suitable for production! See [deploying](#-deploying) for instructions.

You can also make the server LAN-accessible:
```bash
make servelan                           # python3 manage.py runserver 0.0.0.0:8000
```

**WARNING (WSL)**: You might have issues connecting to the local network from another device due to Windows Firewall.

When serving LAN, make sure to add your server's local IP to `ALLOWED_HOSTS` in `.env` to allow other devices to connect to the server. You can check your local IP using `hostname -I` (Linux) or `ipconfig` under `IPv4 Address . . . 192.168.*.*` (Windows).

## 📌 Deploying
### 🏢 Server
For production, ensure `SECRET_KEY` and `ALLOWED_HOSTS` are set correctly! These are very important for security. `DEBUG` should also be set to false.

If you're happy with everything and are ready to launch an instance of cws2 into production, you can do this via Gunicorn. With all the project dependencies already set up, you can launch the server with the following command. (Adjust certain parameters to you and your server's needs!)

```bash
gunicorn --access-logfile - --workers 3 cws2.config.wsgi:application
```

You can use the `--bind` parameter to bind the server to a specific socket, which can be useful for certain server management setups e.g controlling it with systemd.

### 🖼 Assets
When running the production server, you will have to serve static assets yourself! This is most easily done with a server like Nginx, which can also be used to proxy requests to the Gunicorn server/socket.

In development, the Django server automatically fetches the static files of any dependencies but the production server won't know where these are so you'll need to manually collect up all the static files of any dependencies into your project environment. Luckily there's a management command for this.

```bash
make static                             # python3 manage.py collectstatic
```

## 🤖 Development
### 🎨 Compiling Assets
Style assets are written in SASS and compiled on the server, so when developing locally you need to make sure you compile these assets before you see any change on your development copy. You can do that with:
```bash
# python3 manage.py sass cws2/static/scss/base.scss cws2/static/css/base.css -g -t compressed
make sass
```

Alternatively, to compile assets and keep watching for new changes, you can run:
```bash
# python3 manage.py sass cws2/static/scss/base.scss cws2/static/css/base.css -g -t compressed --watch
make watch
```

### 🧪 Testing
We use [pytest](https://docs.pytest.org/en/7.2.x/) for testing. You can run tests with the following command:
```bash
make test                               # pytest
```

### 🧩 Migrations
If you make any changes to models, you'll need to write migrations so that users of the app can keep their database up to date with these changes.

Migrations are written as regular Python scripts in the `cws2/migrations` directory, and you can read about how to write them [here](https://docs.djangoproject.com/en/4.1/topics/migrations/).

There's a script for auto-generating migrations based on changes you make to models which you can run using the command below:
```bash
make migrations                         # python3 manage.py makemigrations
```

Note that for more complex changes such as modifying data in the database, you'll have to write the migration yourself. You can generate a blank migration file using the following command.

```bash
python manage.py makemigrations cws2 --empty -n my_migration
```

### 🍓 Code Style
We care about code style, so all code should be compliant with [ruff](https://docs.astral.sh/ruff/)'s formatter and linter.

You can check whether your code is compliant with the above using the following command.
```bash
make lint                               # ruff check .; ruff format --check .
```

You can also automatically format your code with the following command.
```bash
make format                             # ruff format .
```

Code editors often have tools to automate this, such as in [VS Code](https://dev.to/adamlombard/how-to-use-the-black-python-code-formatter-in-vscode-3lo0).

## 🤝 Contributing
We'd love for you to contribute! Read [CONTRIBUTING.md](CONTRIBUTING.md) for some guidelines, and keep in mind our style guidelines. Also, make sure you follow our [code of conduct](CODE_OF_CONDUCT.md) when contributing on GitHub.

## 📚 License
cws2 is licensed under a [BSD 2-Clause](https://opensource.org/licenses/BSD-2-Clause) license - see [LICENSE](LICENSE) to read it in full.

## 🌳 Acknowledgements
Thanks to Jay (hashi) for all your hard work on CWS's first version. We love you. 💘
