# Connecting to Cloud SQL - SQL Server

This is a sample application that inserts and reads votes for two options (tabs and spaces) in a Cloud SQL database. The application demonstrates the reommended method of connecting to Cloud SQL from a Java application using the [Cloud SQL Java Connector](https://github.com/GoogleCloudPlatform/cloud-sql-jdbc-socket-factory)

## Before you begin

1. If you haven't already, set up a Java Development Environment (including google-cloud-sdk and 
maven utilities) by following the [java setup guide](https://cloud.google.com/java/docs/setup) and 
[create a project](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project).

1. Create a 2nd Gen Cloud SQL Instance by following these 
[instructions](https://cloud.google.com/sql/docs/sqlserver/create-instance). Note the connection string,
database user, and database password that you create.

1. Create a database for your application by following these 
[instructions](https://cloud.google.com/sql/docs/sqlserver/create-manage-databases). Note the database
name. 

1. Create a service account with the 'Cloud SQL Client' permissions by following these 
[instructions](https://cloud.google.com/sql/docs/sqlserver/connect-external-app#4_if_required_by_your_authentication_method_create_a_service_account).
Download a JSON key to use to authenticate your connection. 

1. Use the information noted in the previous steps:
```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service/account/key.json
export CLOUD_SQL_CONNECTION_NAME='<MY-PROJECT>:<INSTANCE-REGION>:<INSTANCE-NAME>'
export DB_USER='my-db-user'
export DB_PASS='my-db-pass'
export DB_NAME='my_db'
```
Note: Saving credentials in environment variables is convenient, but not secure - consider a more
secure solution such as [Cloud KMS](https://cloud.google.com/kms/) or [Secret Manager](https://cloud.google.com/secret-manager/) to help keep secrets safe.

## Deploying locally

To run this application locally, run the following command inside the project folder:

```bash
mvn jetty:run
```

Navigate towards `http://127.0.0.1:8080` to verify your application is running correctly.

## Google App Engine Standard

To run on GAE-Standard, create an AppEngine project by following the setup for these 
[instructions](https://cloud.google.com/appengine/docs/standard/java/quickstart#before-you-begin) 
and verify that 
[appengine-maven-plugin](https://cloud.google.com/java/docs/setup#optional_install_maven_or_gradle_plugin_for_app_engine)
 has been added in your build section as a plugin.


### Development Server

The following command will run the application locally in the the GAE-development server:
```bash
mvn clean package appengine:run
```

Note: if the GAE development server fails to start, check that you are using a supported version of Java. Supported versions are Java 8 and Java 11.

### Deploy to Google Cloud

First, update `src/main/webapp/WEB-INF/appengine-web.xml` with the correct values to pass the 
environment variables into the runtime.

Next, the following command will deploy the application to your Google Cloud project:
```bash
mvn clean package appengine:deploy
```

### Deploy to Cloud Run

See the [Cloud Run documentation](https://cloud.google.com/run/docs/configuring/connect-cloudsql)
for more details on connecting a Cloud Run service to Cloud SQL.

1. Build the container image using [Jib](https://cloud.google.com/java/getting-started/jib):

  ```sh
mvn clean package com.google.cloud.tools:jib-maven-plugin:2.8.0:build \
 -Dimage=gcr.io/[YOUR_PROJECT_ID]/run-sqlserver -DskipTests
  ```

2. Deploy the service to Cloud Run:

  ```sh
  gcloud run deploy run-sqlserver \
    --image gcr.io/[YOUR_PROJECT_ID]/run-sqlserver \
    --platform managed \
    --allow-unauthenticated \
    --region [REGION] \
    --update-env-vars CLOUD_SQL_CONNECTION_NAME=[INSTANCE_CONNECTION_NAME] \
    --update-env-vars DB_USER=[MY_DB_USER] \
    --update-env-vars DB_PASS=[MY_DB_PASS] \
    --update-env-vars DB_NAME=[MY_DB]
  ```

  Replace environment variables with the correct values for your Cloud SQL
  instance configuration.

  Take note of the URL output at the end of the deployment process.

  It is recommended to use the [Secret Manager integration](https://cloud.google.com/run/docs/configuring/secrets) for Cloud Run instead
  of using environment variables for the SQL configuration. The service injects the SQL credentials from
  Secret Manager at runtime via an environment variable.

  Create secrets via the command line:
  ```sh
  echo -n "my-awesome-project:us-central1:my-cloud-sql-instance" | \
      gcloud secrets versions add CLOUD_SQL_CONNECTION_NAME_SECRET --data-file=-
  ```

  Deploy the service to Cloud Run specifying the env var name and secret name:
  ```sh
  gcloud beta run deploy SERVICE --image gcr.io/[YOUR_PROJECT_ID]/run-sql \
      --add-cloudsql-instances [CLOUD_SQL_CONNECTION_NAME] \
      --update-secrets CLOUD_SQL_CONNECTION_NAME=[CLOUD_SQL_CONNECTION_NAME_SECRET]:latest,\
        DB_USER=[DB_USER_SECRET]:latest, \
        DB_PASS=[DB_PASS_SECRET]:latest, \
        DB_NAME=[DB_NAME_SECRET]:latest
  ```

3. Navigate your browser to the URL noted in step 2.

  For more details about using Cloud Run see http://cloud.run.
  Review other [Java on Cloud Run samples](../../../run/).

### Cleanup
To avoid incurring any charges, navigate to your project's [App Engine settings](https://console.cloud.google.com/appengine/settings) and click `Disable Application`. Also [delete your Cloud SQL Instance](https://cloud.google.com/sql/docs/mysql/delete-instance) if you no longer need it.
