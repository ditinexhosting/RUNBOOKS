# CD react in VPS server with github action

#### Prerequisite
- You need to have a private key and public key of your SSH access where public key is already added in `~/.ssh/authorized_keys`. i.e The SSH key have access to login into the server and access the code. 
- Create a new ssh key inside the server `ssh-keygen`
- Copy the content of public key created at *~/.ssh/id_ed25519.pub*
- Go to [Github profile settings](https://github.com/settings/keys)
- Add the public key there under *SSH keys*


#### Create CD pipeline
- In the front-end project create file `.github/workflows/staging.yml` and paste the below deployment code : 
```
# This is a basic workflow to help you get started with Actions

name: CI/CD FRONTEND STAGING

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [staging]
  pull_request:
    branches: [none]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  deploy:
    name: 'Deploy to frontend STAGING'
    runs-on: ubuntu-latest
    steps:
      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh/
          echo "$SSH_PRIVATE_KEY" > ~/.ssh/server.key
          chmod 600 ~/.ssh/server.key
          cat >>~/.ssh/config <<END
          Host server
            HostName $SSH_HOST
            User $SSH_USER
            IdentityFile ~/.ssh/server.key
            StrictHostKeyChecking no
            Port $PORT
          END
        env:
          SSH_USER: ${{ secrets.SSH_STAGING_USER }}
          SSH_PRIVATE_KEY: ${{ secrets.SSH_STAGING_PRIVATE_KEY }}
          SSH_HOST: ${{ secrets.SSH_STAGING_HOST }}
          PORT: ${{ secrets.SSH_STAGING_PORT }}

      - name: Deploying into server
        run: |
          ssh server 'cd /home/ubuntu/mice.ditinex.com/MICE-frontend && git reset --hard && git pull origin staging && npm install && npm run build && rm -rf build && mv dist build && pm2 restart mice.ditinex.com-FE --time && pm2 flush mice.ditinex.com-FE'
      - run: echo "🎉 Deployed successfully."
```
- Modify the names and command as per your need. 
- Go to settings of the repository and then go to "Secret and variables" > "actions".
- Add the secrets used in above code i.e SSH_STAGING_USER,SSH_STAGING_PRIVATE_KEY,SSH_STAGING_HOST,SSH_STAGING_PORT
- Add them under "Repository secrets"
 
