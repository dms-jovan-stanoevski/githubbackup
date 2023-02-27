# githubbackup

A Docker image for backing up Github repositories.

## Usage

```yaml
name: Backup

on:
  workflow_dispatch:
  schedule:
    - cron: '0 0 * * *'

permissions:
  id-token: write
  contents: read

      # Needs secrets:
      # BACKUP_GITHUB_PAT
      # BACKUP_GITHUB_OWNER
      # BACKUP_AWS_ROLE
      # BACKUP_AWS_REGION
      # BACKUP_BUCKET_NAME

jobs:
  Backup:
    runs-on: ubuntu-latest
    container: ghcr.io/dms-jovan-stanoevski/githubbackup:main
    name: Backup

    steps:
      - name: Assume role
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: ${{ secrets.BACKUP_AWS_ROLE }}
          aws-region: ${{ secrets.BACKUP_AWS_REGION }}

      - name: Display assumed role
        run: aws sts get-caller-identity

      - name: Backup
        shell: bash
        env:
          BACKUP_GITHUB_PAT: ${{ secrets.BACKUP_GITHUB_PAT }}
          BACKUP_GITHUB_OWNER: ${{ secrets.BACKUP_GITHUB_OWNER }}
          BACKUP_AWS_REGION: ${{ secrets.BACKUP_AWS_REGION }}
          BACKUP_BUCKET_NAME: ${{ secrets.BACKUP_BUCKET_NAME }}
        run: backup-organisation-code
```

## Tasks

### build

```sh
docker build --progress=plain -t githubbackup:latest .
```

### shell

```sh
docker run -it --rm --entrypoint "/bin/bash" githubbackup:latest
```

### run-locally

```sh
docker run -e BACKUP_GITHUB_PAT -e BACKUP_AWS_ROLE -e BACKUP_AWS_REGION -e BACKUP_BUCKET_NAME githubbackup:latest backup-organisation-code
```
## AWS CodeCommit Instructions:

clone your AWSCodeCommit repository with the mirror flag, 

git clone --mirror codecommit::{region}://{repo} {repo}

then we are going into the folder and add our GitHub remote path with the token and mirror tag with fetch at the end of it we are fetching from the origin (CodeCommit) and push 
to GitHub repository after it.

cd /into-clonned-aws-codecommit-repo-local-folder
git remote add --mirror=fetch github https://<token>@github.com/{username}/{repo}

And afterwards execute 
git fetch origin
git push github --all

Each time you made PR towards AWS CodeCommit Repo and want to sync that change back to GitHub Repo repeat last two stated commands , and all changes that were committed towards 
AWS CodeCommit Backup Repo will reflect and transfer the same exact code/file change towards same GitHub repo.

Simulation Test 01
