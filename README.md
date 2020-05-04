# Synchronising a GitHub Repository into Bitbucket.

This project demonstrates maintaining code in Github and having it automatically update into BitBucket. This is entirely driven from the GitHub end and only requires a github action and a secret containing the target repo and access credentials. 

This will work for public and private repositories. For private repos you will need to modify the Github secret BITBUCKET_SYNC_URL to include the bitbucket access credentials. Or modify the action so that it has SSH access to BB. 

This is a one way synch. If you modify the BB version either the action will fail due to conflicts or the next update will replace your changes.  Hence you should configure the Bitbucket repo to be read only except for the Github account doing the updates. 

The action script syncs all branches on push.  You might want to modify the script to only synch on push to master. 

## Steps for an existing Github repository

1. Checkout the repo, create a .github/workflows folder and copy the sync-bitbucket.yml file there.
2. On Bitbucket create a new empty repository in the project and workspace you want to use. You should name the repo the same as on github but its not absolutely necessary. Get the URL for the new repo. e.g. `https://<username>:<password>@bitbucket.org/pfr/example-repo.git`. If the repo is public you will not require the username and password.
3. For private repos, on Github in the Settings tab, add a secret with name BITBUCKET_SYNC_URL and value being the URL. This is used to set the sync remote for the checkout to the bitbucket repo. 
    `git remote add sync ${{ secrets.BITBUCKET_SYNC_URL }}` 
1. For public repos - you can do the same or use a literal string in the action.
    `git remote add sync https://bitbucket.org/pfr/example-repo.git`
2. Commit the changes. The action will run and the results will show in the actions tab on the repository. 
3. Verify the Bitbucket repo - it should now contain a full copy of the github repo including history and branches. 
4. Now any time you push to github the bitbucket repo will also be updated. 





