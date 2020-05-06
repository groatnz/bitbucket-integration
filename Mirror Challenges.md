# Challenges in managing mirrored repositories between BitBucket and GitHub
_Andrew Watkins_ _May 2019_

# Summary
We (PFR) want access to git repositories that are low usage for writing but open to many within the company for reading. GitHub enterprise licence charges the same for all forms of access as does Bitbucket. Bitbucket licences are significantly cheaper, but many existing users prefer GitHub. With 1200 staff and 400+ likely users of git repositories this represents a significant long term cost.

As git is designed as a distributed source code management system the activity of copying from one repository to another is relatively simple. Problems arise however, when the various systems involve privacy and security. 

This article reviews some of the challenges in using both systems together with BitBucket being used as a cheaper widespread publishing tool and GitHub being used more for development. 



## Why do we have this problem? 
Because the pattern of science projects is to create some analysis code and data that is applied to a specific experiment or piece of research.  One person may write content for a period and then the repository is frozen. Many others may want to read, copy and run the data + code in the repository while not necessarily needing to push updates to it. 

## Can't the repository be made public then? 
No - it may contain PFR IP and while open internally across the company it should not be globally public. 

## Can the repository be public just in the organisation?
No - not using the GitHub cloud service. Yes if we installed GitHub Enterprise Server which is an internally hosted solution but this would lose privacy between different science groups.

# first solution: Basic Synchronisation.
Accounts on BitBucket are much cheaper than on GitHub ($3 vs $21) so we can afford to put a larger number of people onto BitBucket. However, some people can't or won't move from GitHub so we proposed to provide a method that will allow a repository on Github to be copied or mirrored into BitBucket each time it is updated. 

This fix involves a GitHub Action - a script triggered on push events that checks out the repository, changes the upstream remote to that of a matching BitBucket repository and then pushes the updated files to BitBucket. see .github/workflows/sync-bitbucket.yml 

We need to provide to the action the destination repo to synch into. This can be hard coded into the action .yml file, placed in a secret in the GH Repo, or ideally implied from the source repo - i.e if the source and destination names are matched. 

# Challenges
## Permissions
Each repository on Bitbucket belongs to an owner which is a person or organisation. If the repository is private then it can only by pushed to by someone who has the required access credentials.  This will be either a username and password or an SSH key

However we can't hard code these credentials into the Action as the action may be copied into many new repositories. we would prefer it to have zero configuration.  

### The username:password fix
We can use GitHub secrets to store the destination and credentials. If the remote repo can be accessed using https with a username and password we can save the entire destination URL in the secret. 
e.g 
`BITBUCKET_SYNC_URL=https://username:password@bitbucket.org/workspace/repository.git`

This means that each owner of a repository on GH will need to setup Bitbucket with a new blank repository, then add to the GH repo secrets the remote URL. 

This is the approach taken by sync-bitbucket.yml

### The SSH fix
SSH key pairs allow git users to avoid having to type username/password at the command line. After generating a public/private key pair the private key is kept in the local ~/.ssh folder while the public key is associated with the person's account on BitBucket (and GitHub too for that matter).

In order for the sync action to operate we need to provide the private key to the action so that it can be placed in the container's .ssh file ( or git command line) 
This is the approach taken by sync-bitbucket-ssh.yml  We now need a differt secret to be added to the repo. 
`BITBUCKET_SSH_KEY` - the private key starting with -----BEGIN RSA PRIVATE KEY-----

But this has the advantage that that destination URL is simpler and does not contain credentials. Hence, can be placed directly in the script.
````
env:
  BB_REPO: git@bitbucket.org:groatnz/bitbucket-integration.git
````
## The Single Sign on SSO Problem
When single sign on is enabled for Bitbucket (or GH) it stops being possible to use the simple https form of git request i.e. https://username:password@bitbucket.org as the username routes through the PFR AD for authentication and (I don't think) that the password can be carried through the request - will test this.
This means that bot/machine accounts and command line access becomes much more dependent on SSH keys and limits us to the SSH version of the solution.

## Scaling and Usability
The above solutions work ok but we run into difficulties in making a system that scales to a larger number of inexperienced users. 

PFR Staff would like to be able to simply click a new repo button, or clone/fork an existing repo into their workspace and at most copy in the action script. 
However they will also have to:
1. create a new empty repo on BB 
2. copy their bitbucket username/password or SSH key into a secret in the repo.
3. which may in turn require them to generate a suitable key pair.

This method is therefore difficult for the casual user who just wants to turn on mirroring of the Github repo. It would effectively make creating new repositories a help desk request.

If we are to migrate a significant number of existing repositories (~300) we would need to automate this process.

## Possible solutions

### One credential to rule them all
Create a special account on BB that is the GitHub user. This account has one ssh keypair where the public key is in BB and the private key can be injected by an API script into each GH repository. 

- a problem with this is that with a little editing we can craft a GH repo that can read all the repos on BB including those that should be private to other groups. 
- which implies we should not have a super account that can access all BB repositories. 


### Everyone gets a key
The link between the GH and BB private repositories must be connected by some shared secret that belongs to the owner of the repository and is different for each repository. That is either the username/password of the owner, or an SSH key generated for the owner - but is different for each repo. 

To simplify setup we would need an automation script that does the following for a given GH repo - all via GH and BB APIs:
* remotely creates the bb repo
* creates a key pair, saves it somewhere safe
* places the private key in a GH secret
* places the public key into the BB repo as an access key
  
This need could be met by Deployment keys.

#### Deploy keys - Github.
You can launch projects from a GitHub repository to your server by using a deploy key, which is an SSH key that grants access to a single repository. GitHub attaches the public part of the key directly to your repository instead of a personal user account, and the private part of the key remains on your server - this allows the server to pull data from the repository. 
https://developer.github.com/v3/guides/managing-deploy-keys/

#### Deployment keys - Bitbucket
So we need something like deploy keys on BitBucket
https://bitbucket.org/blog/deployment-keys
Deployment keys allow users to clone/pull source from a repository over SSH using Git and Hg without having to use up one of their plan limit users. Deployment keys are similar to adding SSH keys to your account, but they are done on a per-repository basis. You can add the same deployment key to multiple repositories.

However, on BB we can only use deploy keys for reading the repo. 

- so currently it would be easier to have a specific key that can pull from BitBucket and Push to Github, but this is the wrong way around for our requirements. 


## Do the services support the required automation.
###  Bitbucket API
https://developer.atlassian.com/
https://developer.atlassian.com/cloud/bitbucket/getting-started/
- example app sets up auth and access for calling API 
* api to create bb repo - Not found
* api to set user ssh - Not found

Although BB has a developer API it is mainly focussed on writing companion apps that are mainly read only on the BB data. e.g. webhooks that report changes into other systems. 

### GitHub API
https://developer.github.com/v3/
* can create/update/delete new files in the repo - yes
* create repo using a template - yes https://developer.github.com/v3/repos/#create-a-repository-using-a-template ( preview period 
* create repo - https://developer.github.com/v3/repos/#create-a-repository-for-the-authenticated-user) (in preview) 

* create a new secret for the action - No


