# Sync GitHub Repo with BitBucket

This project demonstrates maintaining code in Github and having it automatically update into BitBucket
The copy in BB is a Fork that pulls from upstream and updates on a github action when the upstream changes

Clone and Sync between private accounts on bitbucket and GitHub.

Example test
- Organization on Github - groat-nz
- Repository groat-nz/enterprise-demo

This is a Private Repository - only owner has access

1. Connect accounts in BB to Github. 

Setup BitBucket so that it has access to the Github repository - inc private ones.

See: Personal Settings | Connected Accounts.
click connect to Github,  authorise BB to access the GH repostory.

## Fork the github repo into BB.

Note: Don't use import as this creates a new repo that is disconnected from the orginal

Go to Bitbucket and create a new repository (its better to have an empty repo)
then checkout and enter.

    git clone https://groat_nz@bitbucket.org/groat_nz/enterprise-demo.git
    # or git clone git@bitbucket.org:abc/myforkedrepo.git

    Cloning into 'enterprise-demo'...
    Password for 'https://groat_nz@bitbucket.org': 
    warning: You appear to have cloned an empty repository.

    cd enterprise-demo

Now add Github repo as a new remote in Bitbucket called "sync"
    git remote add sync https://github.com/groat-nz/enterprise-demo.git

Verify what are the remotes currently being setup. This following command should show "fetch" and "push" for two remotes i.e. "origin" and "sync"
    git remote -v

Now do a pull from the "master" branch in the "sync" remote 
    git pull sync master

Setup a local branch called "github" track the "sync" remote's "master" branch
    git branch --track github sync/master

Now push the local "master" branch to the "origin" remote in Bitbucket.
    git push -u origin master
    
To pull changes from the original :

    git pull sync master  --allow-unrelated-histories


and then push back to Bitbucket

    git push 


# Now we make this a github action. 
