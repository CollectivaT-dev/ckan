# Installation instructions for local development

0) on a different folder of your machine clone https://github.com/CollectivaT-dev/datapusher and build datapusher image: "docker build -t datapusher ."

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

