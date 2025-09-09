# django_production
Got it 👍 You want to migrate your Django project to a new server.
Here’s a step-by-step guide that works on Linux (Ubuntu/Debian, CentOS, etc.):

1️⃣ Copy Your Project to the New Server

On your old server/local machine:

scp -r /opt/myproject user@NEW_SERVER_IP:/opt/


(or use git if your project is version controlled)

2️⃣ Install Dependencies on New Server

Make sure Python, pip, and virtualenv are installed:

sudo apt update
sudo apt install python3 python3-pip python3-venv -y


Create virtual environment:

cd /opt/myproject
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

3️⃣ Setup Database

If you’re using PostgreSQL:

sudo apt install postgresql postgresql-contrib -y
sudo -u postgres psql


Inside psql:

CREATE DATABASE myprojectdb;
CREATE USER myprojectuser WITH PASSWORD 'your_password';
ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE myprojectuser SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE myprojectdb TO myprojectuser;
\q


Update your settings.py:

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myprojectdb',
        'USER': 'myprojectuser',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

4️⃣ Run Migrations
source venv/bin/activate
python manage.py makemigrations
python manage.py migrate

5️⃣ Collect Static Files
python manage.py collectstatic

6️⃣ Create Superuser
python manage.py createsuperuser

7️⃣ Run Django Server (test)
python manage.py runserver 0.0.0.0:8000

8️⃣ Setup Production Server (Gunicorn + Nginx)

Install Gunicorn & Nginx:

pip install gunicorn
sudo apt install nginx -y


Create systemd service /etc/systemd/system/myproject.service:

    [Unit]
    Description=Gunicorn instance to serve myproject
    After=network.target
    
    [Service]
    User=www-data
    Group=www-data
    WorkingDirectory=/opt/myproject
    ExecStart=/opt/myproject/venv/bin/gunicorn --workers 3 --bind unix:/opt/myproject/myproject.sock                   config.wsgi:application
    
    [Install]
    WantedBy=multi-user.target


Enable & start:

sudo systemctl start myproject
sudo systemctl enable myproject

9️⃣ Configure Nginx

Create /etc/nginx/sites-available/myproject:

    server {
        listen 80;
        server_name myserver.com;
    
        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /opt/myproject;
        }
    
        location / {
            include proxy_params;
            proxy_pass http://unix:/opt/myproject/myproject.sock;
        }
    }


Enable site:

sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx

🔟 (Optional) Enable HTTPS
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d myserver.com


✅ Now your Django project is live on the new server!

Do you want me to also write the steps to copy your existing database from the old server (dump/restore) so you keep your users/data?
