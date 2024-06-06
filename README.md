
# Basic Library App

## Stack

* Python
* Flask
* Google Cloud Storage
* Google Cloud Firestore NoSQL Database
* Secret Manager to store sensitive application data.
* OAuth 2.0 to add user login to an application.
* Cloud Translation API to detect the language of text and translate text.
* Artifact Registry
* Cloud Run
* Cloud Logging

## Description
A Python application that manages a list of books. Supports adding, editing, and deleting books. Collecting data like author, title, and description.

## gcloud CLI commands used
### Create firestore database
gcloud firestore databases create --location=us-east4
### Create storage bucket
gcloud storage buckets create gs://${GOOGLE_CLOUD_PROJECT}-covers --location=us-east4 --no-public-access-prevention --uniform-bucket-level-access

### Enable secret manager and configure secret
gcloud services enable secretmanager.googleapis.com
gcloud secrets create bookshelf-client-secrets --data-file=$HOME/client_secret.json
tr -dc A-Za-z0-9 </dev/urandom | head -c 20 | gcloud secrets create flask-secret-key --data-file=-

### Make bucket publicly available
gcloud storage buckets add-iam-policy-binding gs://${GOOGLE_CLOUD_PROJECT}-covers --member=allUsers --role=roles/storage.legacyObjectReader

### Create Docker image in Artifact Registry
gcloud artifacts repositories create app-repo \
  --repository-format=docker \
  --location=us-east4

### Build app with Cloud Build
gcloud builds submit \
  --pack image=us-east4-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/app-repo/bookshelf \
  ~/bookshelf

### Deploy Bookshelf app
gcloud run deploy bookshelf \
  --image=us-east4-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/app-repo/bookshelf \
  --region=us-east4 --allow-unauthenticated \
  --update-env-vars=GOOGLE_CLOUD_PROJECT=${GOOGLE_CLOUD_PROJECT}


### Create service account and grant roles secret manager, cloud translate, datastore, and cloud storage
gcloud iam service-accounts create bookshelf-run-sa
gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
  --member="serviceAccount:bookshelf-run-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
  --member="serviceAccount:bookshelf-run-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
  --role="roles/cloudtranslate.user"
gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
  --member="serviceAccount:bookshelf-run-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
  --role="roles/datastore.user"
gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
  --member="serviceAccount:bookshelf-run-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
  --role="roles/storage.objectUser"
