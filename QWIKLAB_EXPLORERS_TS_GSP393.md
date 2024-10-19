# Implement CI/CD Pipelines on Google Cloud: Challenge Lab || [GSP393](https://www.cloudskillsboost.google/focuses/52826?parent=catalog) ||

# # Like, comment, share & Don't forget to subscribe [Qwiklab_Explorers_ts](https://youtube.com/@titashshil?si=RgamNu1dc9jVIbJN) 👍😄🤝

### Run the following Commands in CloudShell

```
export ZONE=
```
```
export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
export REGION="${ZONE%-*}"
gcloud config set compute/region $REGION

gcloud services enable \
container.googleapis.com \
clouddeploy.googleapis.com \
artifactregistry.googleapis.com \
cloudbuild.googleapis.com \
clouddeploy.googleapis.com

sleep 20

gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$(gcloud projects describe $PROJECT_ID \
--format="value(projectNumber)")-compute@developer.gserviceaccount.com \
--role="roles/clouddeploy.jobRunner"

gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$(gcloud projects describe $PROJECT_ID \
--format="value(projectNumber)")-compute@developer.gserviceaccount.com \
--role="roles/container.developer"

gcloud artifacts repositories create cicd-challenge \
--description="Image registry for tutorial web app" \
--repository-format=docker \
--location=$REGION

gcloud container clusters create cd-staging --node-locations=$ZONE --num-nodes=1 --async
gcloud container clusters create cd-production --node-locations=$ZONE --num-nodes=1 --async

cd ~/
git clone https://github.com/GoogleCloudPlatform/cloud-deploy-tutorials.git
cd cloud-deploy-tutorials
git checkout c3cae80 --quiet
cd tutorials/base

envsubst < clouddeploy-config/skaffold.yaml.template > web/skaffold.yaml
cat web/skaffold.yaml

cd web
skaffold build --interactive=false \
--default-repo $REGION-docker.pkg.dev/$DEVSHELL_PROJECT_ID/cicd-challenge \
--file-output artifacts.json
cd ..

cp clouddeploy-config/delivery-pipeline.yaml.template clouddeploy-config/delivery-pipeline.yaml
sed -i "s/prod/cd-production/" clouddeploy-config/target-cd-production.yaml


for CONTEXT in ${CONTEXTS[@]}
do
    envsubst < clouddeploy-config/target-$CONTEXT.yaml.template > clouddeploy-config/target-$CONTEXT.yaml
    gcloud beta deploy apply --file clouddeploy-config/target-$CONTEXT.yaml
done

gcloud beta deploy releases create web-app-001 \
--delivery-pipeline web-app \
--build-artifacts web/artifacts.json \
--source web/

gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001

# Wait for the rollout to complete
while true; do
  # Check the rollout status
  status=$(gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001 --format="value(state)" | head -n 1)
  
  # If the status is SUCCESS, break out of the loop
  if [ "$status" == "SUCCEEDED" ]; then
    break
  fi
  
  # Wait for a short duration before checking agai
  echo "it's creating now, please wait..."

  sleep 10
done


gcloud beta deploy releases promote \
--delivery-pipeline web-app \
--release web-app-001 \
--quiet

###


# Wait for the rollout to complete
while true; do
  # Check the rollout status
  status=$(gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001 --format="value(state)" | head -n 1)
  
  # If the status is SUCCESS, break out of the loop
  if [ "$status" == "PENDING_APPROVAL" ]; then
    break
  fi
  
  # Wait for a short duration before checking again
  echo "it's creating now, please wait..."
  sleep 10
done


gcloud beta deploy rollouts approve web-app-001-to-cd-production-0001 \
--delivery-pipeline web-app \
--release web-app-001 \
--quiet


# Wait for the rollout to complete
while true; do
  # Check the rollout status
  status=$(gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001 --format="value(state)" | head -n 1)
  
  # If the status is SUCCESS, break out of the loop
  if [ "$status" == "SUCCEEDED" ]; then
    break
  fi
  
  # Wait for a short duration before checking again
  echo "it's creating now, please wait..."
  sleep 10
done

gcloud services enable cloudbuild.googleapis.com

cd ~/
git clone https://github.com/GoogleCloudPlatform/cloud-deploy-tutorials.git
cd cloud-deploy-tutorials
git checkout c3cae80 --quiet
cd tutorials/base

envsubst < clouddeploy-config/skaffold.yaml.template > web/skaffold.yaml
cat web/skaffold.yaml

cd web
skaffold build --interactive=false \
--default-repo $REGION-docker.pkg.dev/$DEVSHELL_PROJECT_ID/cicd-challenge \
--file-output artifacts.json
cd ..

gcloud beta deploy releases create web-app-002 \
--delivery-pipeline web-app \
--build-artifacts web/artifacts.json \
--source web/


# Wait for the rollout to complete
while true; do
  # Check the rollout status
  status=$(gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-002 --format="value(state)" | head -n 1)
  
  # If the status is SUCCESS, break out of the loop
  if [ "$status" == "SUCCEEDED" ]; then
    break
  fi
  
  # Wait for a short duration before checking agai
  echo "it's creating now, please wait..."
  sleep 10
done

gcloud deploy targets rollback cd-staging \
   --delivery-pipeline=web-app \
   --quiet
```

# Congratulations ..!!🎉  You completed the lab shortly..😃💯

# *Well done..!* 👏

# Thank you for visiting.... :) 🗯️

# [Qwiklab_Explorers_ts](https://youtube.com/@titashshil?si=RgamNu1dc9jVIbJN)
