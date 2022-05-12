# Github Actions Build Deploy With Staging and Production Environment Secret

Do you feel that the github documentation is horrible? 
Unable to use the environments secrets? 
Fear not, I already did the hardwork bruteforcing all possibilities.

TLDR:
You need **2 files** to use dynamic environment variables! This is not mentioned in ANY github documentation, it's not mentioned either how to use Environment Secrets at all

First create your environment:

![image](https://user-images.githubusercontent.com/15200079/168071603-4dc8540a-ae65-4153-b987-21eb6a7365cd.png)


The name of the environment should be the name of the branch you'll use for that env:

![image](https://user-images.githubusercontent.com/15200079/168072238-fbb527a4-5868-4293-a830-0fd0901076d2.png)

Here you can specify your envs to a specific branch:

![image](https://user-images.githubusercontent.com/15200079/168072341-06ce4581-daab-44c1-93a0-9656b2dfe16d.png)

![image](https://user-images.githubusercontent.com/15200079/168072394-5afce27c-2913-4b6b-b7df-d07da41d642a.png)

Now you can create your Environment Secrets!

![image](https://user-images.githubusercontent.com/15200079/168072447-4dd7340b-76f2-41d3-b50e-7178d7cd5d03.png)

![image](https://user-images.githubusercontent.com/15200079/168072654-d9c1f5dd-7f7b-466d-bc2e-c3d54b7de7ff.png)

After creating you should see something like this:

![image](https://user-images.githubusercontent.com/15200079/168072727-ececb7f7-68ab-43f8-8033-86c9200db7c5.png)

Now you can create multiple environments! Examples: **staging**, **qa**, **development**, **feat/\***, etc

## Workflow using environment secrets

You need 2 files for that (why Github?). First create `.github/workflows/main.yml` here you'll define your desired branches and variables

```yaml
on:
  push:
    branches:
    - main
    - staging
    - development

jobs:  
  deploy:
    uses: ./.github/workflows/deploy.yml
    with:
      environment: ${{ github.ref_name }}
    secrets:
      AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
```

Second create your deploy workflow (or something else) `.github/workflows/deploy.yml` here you'll define your deploy script, and this script will use the environments defined in main.yml. I'm using a simple s3 aws deploy

```yaml
name: Deployment


on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      AWS_S3_BUCKET:
        required: true
      AWS_ACCESS_KEY_ID:
        required: true
      AWS_SECRET_ACCESS_KEY:
        required: true
      AWS_REGION:
        required: true


jobs:
  deploy:
    name: Deploy
    environment: ${{ github.ref_name }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - uses: jakejarvis/s3-sync-action@master
        name: Deploy to S3
        env:
          AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ secrets.AWS_REGION }}
        with:
          args: --acl public-read --follow-symlinks --delete
```

That's how you use Environment Secrets in Github Actions! There is nothing about that in the github actions docs

# TODO

[ ] How to use only one file? Seems dumb using 2 files for environment selection
[ ] How to use a different environment name for the branch and the environment?
