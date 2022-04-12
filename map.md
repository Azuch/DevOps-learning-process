1. File need to create:
    - .gitlab-ci.yml
    - Dockerfile
    - .gitignore

2. Folder need to create: This is for beanstalk to auth the registry and deploy image. You can read at: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/awseb-dg.pdf#GettingStarted

    - template
    - template/Dockerrun.aws.json
    - template/auth.json -> use to auth the registry of Gitlab, it is private

3. the process is:

- build a web

- push it to gitlab

- Create variable, deploy_token, IAM, environment: production and staging

- build it:
    + Artifact the root folder

- Build an image:
    + Login to registry of the project
    + create a dockerfile
    + Build an image with two tags: latest and version
    + Push all image to registry

- Test image by a service: $CI_REGISTRY_IMAGE:$APP_VERSION, alias: website

- Deploy to staging: it is an staging app

- Deploy to production:
    + Create the template folder: dockerrun.aws.json, auth.json

    + create dockerrun.aws.json, auth.json with:
        - yum install -y gettext
        - envsubst < templates/Dockerrun.aws.json > Dockerrun.aws.json
        - envsubst < templates/auth.json > auth.json
    
    + copy dockerrun.aws.json and auth.json to s3 of beanstalk

    + Deploy beanstalk:
        - aws elasticbeanstalk create-application-version --application-name "$APP_NAME" --version-label $APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=Dockerrun.aws.json
        - aws elasticbeanstalk update-environment --application-name "$APP_NAME" --version-label $APP_VERSION --environment-name $APP_ENV_NAME
        - aws elasticbeanstalk wait environment-updated --application-name $APP_NAME --version-label $APP_VERSION --environment-name $APP_ENV_NAME
        - curl $CI_ENVIRONMENT_URL/version.html | grep $APP_VERSION
        