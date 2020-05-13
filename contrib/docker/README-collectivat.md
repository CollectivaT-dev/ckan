Installation instructions for local development

1) cd contrib/docker

2) "cp .env.template .env" and set env variables if necessary

3) docker-compose -f docker-compose-development.yml build

4) "docker-compose -f docker-compose-development.yml up -d". The first time is better to run just the database and solr which need longer initialization.
The ckan container will fail to connect until initialization is complete. So it has to be restart it a few times.

5) check you can see home page on http://localhost:5000/

6) docker exec ckan /usr/local/bin/ckan-paster --plugin=ckan datastore set-permissions -c /etc/ckan/production.ini | docker exec -i db psql -U ckan

7) docker cp ckan:/etc/ckan/production.ini .

8) Edit production.ini locally:
 - add datastore datapusher to ckan.plugins
 - enable ckan.datapusher.formats (other settings for datastore and datapusher already hardcoded in several files)

 9) docker cp production.ini ckan:/etc/ckan/production.ini 

 10) docker-compose -f docker-compose-development.yml restart ckan

 11) check http://localhost:5000/api/3/action/datastore_search?resource_id=_table_metadata returns content

 12) docker exec -it ckan /usr/local/bin/ckan-paster --plugin=ckan sysadmin -c /etc/ckan/production.ini add <name_admin_user>

 
