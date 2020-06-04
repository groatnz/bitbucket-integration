# Migrating GitHub Repositories to Bitbucket
This page describes two ways to migrate a repository from GitHub to Bitbucket
1. Using the Bitbucket Import
2. By changing the remote origin

## Bitbucket Import
If you want to completely move a repository from GitHub to BitBucket and don't need to go back then the easiest way is to use the Bitbucket Create Repository -> Import option.

This generates a complete clean copy of the repository in Bitbucket.  It does not have upstream pointing at Github.

* Remove any existing checkouts and clone the repository again.
* Update any tools or scripts that reference the github repo

* check everything builds
* rename the repo in github to decouple it from previous usage if everything still works then delete the repo.

### Checking you have the right upstream repo

`git remote -v`
````
origin	git@bitbucket.org:plantandfood/ckanext-pfr.git (fetch)
origin	git@bitbucket.org:plantandfood/ckanext-pfr.git (push)
````


## Upload to new repo from a checkout.

* checkout the repository.
* cd repo
* `git remote rm origin` to remote existing origin
* create new repo on bitbucket
* `git remote add origin git@bitbucket.org:plantandfood/{repo}.git`
* `git push --set-upstream origin master`


