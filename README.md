# CACHE CLOUD BUILD

## 1- Crear bucket

Debes crear un bucket para la cache inicialmente se llama 'your-cache-bucket'

## 2- Desargar Repo

Este repo en cloudShell o en local tienes que estar conectado al proyecto de GCP

## 3- Correr el comando

Esto es para que se creen los artefactos que vamos a necesitar para cachear

```sh
gcloud builds submit --config cloudbuild.yaml .
```

## 4- Config cloudbuild.yml

Tus primeros steps de cloudbuild.yaml deberian comenzar asi

```yaml
steps:
  - name: "gcr.io/$PROJECT_ID/restore_cache:latest"
    id: "Restoring NPM modules cache"
    args:
      - --bucket=gs://$_CACHE_BUCKET
      - --key=npm-build-cache-$( checksum package.json )-$( checksum package-lock.json )
      - --key_fallback=npm-build-cache-
  - name: "gcr.io/cloud-builders/npm"
    id: "Run Install"
    args:
      # - ci
      - install
      - --prefer-offline
      - --cache=.npm
    waitFor: ["Restoring NPM modules cache"]
  - name: "gcr.io/$PROJECT_ID/save_cache:latest"
    id: "Saving .npm cache"
    args:
      - --bucket=gs://$_CACHE_BUCKET
      - --key=npm-build-cache-$( checksum package.json )-$( checksum package-lock.json )
      - --path=.npm
      - --no-clobber
    waitFor: ["Run Install"]

images: ["gcr.io/${PROJECT_ID}/cmpc-dogs-nest-api:latest"]
options:
  machineType: N1_HIGHCPU_8
substitutions:
  _CACHE_BUCKET: your-cache-bucket
timeout: 900s
```
