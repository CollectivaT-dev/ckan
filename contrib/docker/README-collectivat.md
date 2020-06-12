# Installation instructions for local development

0) on a different folder of your machine clone https://github.com/CollectivaT-dev/datapusher, checkout branch master_collectivat (git checkout master_collectivat) and build datapusher image (docker build -t datapusher .)

1) cd contrib/docker

2) "cp .env.template .env" and set env variables if necessary (note that CKAN_MAX_UPLOAD_SIZE_MB is not present in the template file)

3) docker-compose -f docker-compose-development.yml build

4) "docker-compose -f docker-compose-development.yml up -d". The first time is better to run just the database and solr which need longer initialization.
The ckan container will fail to connect until initialization is complete. So it has to be restarted it a few times.

5) check you can see home page on http://localhost:5000/

6) docker exec ckan /usr/local/bin/ckan-paster --plugin=ckan datastore set-permissions -c /etc/ckan/production.ini | docker exec -i db psql -U ckan

7) docker cp ckan:/etc/ckan/production.ini ./development.ini

8) Edit develpment.ini locally:
 - add datastore datapusher to ckan.plugins
 - enable ckan.datapusher.formats (other settings for datastore and datapusher already hardcoded in several files)

9) uncomment volume bind "- ./development.ini:/etc/ckan/production.ini" in docker-compose-development.yml 

10) docker-compose -f docker-compose-development.yml restart ckan

11) check http://localhost:5000/api/3/action/datastore_search?resource_id=_table_metadata returns content

12) docker exec -it ckan /usr/local/bin/ckan-paster --plugin=ckan sysadmin -c /etc/ckan/production.ini add <name_admin_user>

13) access platform and try to upload a dataset. Check that datapusher works as well.

14) in the parent directory of the ckan folder there should be now a folder called "ckan_extensions" (created when you run docker-compose, see ckan volumes). cd to this folder and clone the theme extension:

git clone https://github.com/CollectivaT-dev/ckanext-collectivat-theme.git

15) open a bash in the running ckan container install the extension:

docker-compose -f docker-compose-development.yml exec ckan bash

cd /usr/lib/ckan/default/src/extensions/ckanext-collectivat_theme

python setup.py develop

16) add collectivat_theme to ckan.plugins in development.ini and restart ckan container

## GUNICORN + NginX in development

If you want to test the gunicorn+Nginx configuration in development, you have to:

1) uncomment command /usr/lib/ckan/venv/bin/gunicorn --paste /etc/ckan/production.ini in docker-compose-development.yml;

2) uncomment nginx container block in docker-compose-development.yml;

3) in development.ini, replace [server:main] block as in development_nginx.example_ini

4) restart ckan and start nginx container. Now the platform should be accessible both at localhost:5000 and at localhost.

# Installation instructions for production

0) on a different folder of your machine clone https://github.com/CollectivaT-dev/datapusher, checkout branch master_collectivat (git checkout master_collectivat) and build datapusher image (docker build -t datapusher .)

1) cd contrib/docker

2) "cp .env.template .env" and set env variables if necessary (note that CKAN_MAX_UPLOAD_SIZE_MB and CKAN_PUBLIC_DOMAIN are not present in the template file)

3) cp development_nginx.example_ini production.ini

4) docker-compose -f docker-compose-production.yml build

5) "docker-compose -f docker-compose-production.yml up -d". The first time is better to run just the database and solr which need longer initialization.
The ckan container will fail to connect until initialization is complete. So it has to be restarted it a few times.

6) update values of beaker.session.secret and app_instance_uuid in production.ini. To get new values, you can use these commands (note that both a new value and the old one are returned by each command):

docker-compose -f docker-compose-production.yml exec ckan ckan-paster make-config --no-interactive ckan /etc/ckan/production.ini | grep beaker.session.secret

docker-compose -f docker-compose-production.yml exec ckan ckan-paster make-config --no-interactive ckan /etc/ckan/production.ini | grep app_instance_uuid 

7) docker exec ckan /usr/local/bin/ckan-paster --plugin=ckan datastore set-permissions -c /etc/ckan/production.ini | docker exec -i db psql -U ckan

8) docker-compose -f docker-compose-production.yml stop
    docker-compose -f docker-compose-production.yml up -d 

9) docker exec -it ckan /usr/local/bin/ckan-paster --plugin=ckan sysadmin -c /etc/ckan/production.ini add <name_admin_user>

10) add ckan server block to local nginx container (see example configuration in nginx/nginx.conf). Then restart nginx and check platform is accessible from outside.

11) access platform and try to upload a dataset. Check that datapusher works as well.

# Data import / export

Datasets, users, groups and organizations can be exported to/import from a json or zip file using the python package ckanapi. To do this from within the ckan container, follow these steps:

1) docker-compose exec ckan bash

2) cd /usr/lib/ckan/venv  (you have to be in a directory where user ckan has write permissions)

3) source bin/activate

4) pip install ckanapi (if not installed yet)

5) To export use:

   ckanapi dump datasets --all -O datasets.jsonl.gz -z -p 4 -c /etc/ckan/production.ini 
   ckanapi dump users --all -O users.jsonl.gz -z -p 4 -c /etc/ckan/production.ini 
   ckanapi dump organizations --all -O organizations.jsonl.gz -z -p 4 -c /etc/ckan/production.ini 

To import use, e.g:

   ckanapi load datasets -I datasets.jsonl.gz -z -p 3 -c /etc/ckan/production.ini

If you want to export data from another ckan server, use e.g.:

    ckanapi dump datasets --all -O datasets.jsonl.gz -z -p 4 -r https://dadess.cat/

NOTE: before loading datasets, remember to load the organizations to which they belong. Also, resource files should be already in the resources directory.