## ansible-config-mgt

Dare.io PBL Repository exclusive to project 11


In this project we automated the tasks that were manually carried out in projects 7 to 10 using asnible configuration management.

#### Error 1: 

ERROR: Couldn't find any revision to build. Verify the repository and branch configuration for this job.
Finished: FAILURE

#### Solution:

Created a new branch called "master" and went to settings/branches then changed default branch to master from main. This now corresponds to the branch specifier on Jenkins
I could have also changed the specifier in Jenkins to */main to match the default branch in GIT.
