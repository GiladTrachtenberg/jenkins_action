Hello everyone! This is another personal-favorite of mine, as it has to do with a Jenkins MBP in AWS spanning across 4 potential repositories, as well as deployment to Jfrog's Artifactory. 

For convenience's sake, I've included all files and instructions in this one repository, so that you'd be able to experiment with the various branches yourselves.

Here is how it works:
1. I setup a CI environment in AWS, first and foremost; using an EC2 server as a base for Jenkins.
   - We have 4 repositories:
     - John
     - Bob
     - Alice
     - Spectate
   - I setup a Jfrog-Artifactory. 
2. I made sure all of these repositories work with the artifactory.
3. I created an MBP that performs a CI (and CD according to the branch name):
John:
- 'Feature' branches ("feature/...") passes build and unit test for every commit (mvn package). 
- 'Master' always performs the CI build stage, e2e tests and deployment to artifactory.
- Release branches ("release/...") attempt a deployment to Artifactory (see below).
Bob: same logic
Alice:
- Only 'Release' branches go through CI stages.
Spectate:
- It has only the main branch. every commit performs build, e2e tests and publish to artifactory.

Release
-------
- Versions `x.y.z` come from the 'Release' branch `release/x.y`
  - Fixed version in pom to `x.y.z`
  - Make sure that other dependencies are on `x.y` 
  - Build and unit test 
  - John & Bob perform E2E tests. Alice doesn't.
  - If all goes well:
    - Deploy to artifactory

Here are some of the plugins you'll need to add to your Jenkins server:
1) Install all suggested plugins upon server init.
2) All Maven plugins, including their MBP plugin.

Have fun and keep on coding :) 