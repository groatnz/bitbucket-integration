# Additional Notes and Tips


# Enable SSH to bitbucket.org.
(macOS)

Add this to the ~/.ssh/config  file
Where groatnz is the organisation or workspace name and you have ssh-keygen a pair called groatnz too. 

````
Host bitbucket.org
  User groatnz
  IdentityFile ~/.ssh/groatnz
  IdentitiesOnly yes 

````

and then change the git clone request to use your user not git.
i.e 
git clone groatnz@bitbucket.org:groatnz/bitbucket-integration.git 
