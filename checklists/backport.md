- Install the working area
```bash
# first export the correct SCRAM_ARCH
cmsrel CMSSW_16_0_1
cd CMSSW_16_0_1/src
cmsenv
```
- Checkout the release branch
```bash
git cms-checkout-topic hgc-tpg:hgc-tpg-$CMSSW_VERSION
```
- Create a new branch from the release branch
```bash
git checkout -b BRANCH_NAME
```
- Add `hgc-tpg` remote and fetch the development branch
```bash
git remote add hgctpg git@github.com:hgc-tpg/cmssw.git
git fetch hgctpg hgc-tpg-devel-CMSSW_16_1_0_pre2
```
- Cherry-pick the commit to be backported
```bash
git cherry-pick COMMIT_HASH
```
- Push the new branch
```bash
git push hgctpg BRANCH_NAME
```
- Make a Pull Request from this new branch to the release branch
- Once the PR is merged, pull the updated release branch locally
```bash
git checkout hgc-tpg-$CMSSW_VERSION
git pull --rebase hgctpg hgc-tpg-$CMSSW_VERSION
```
- Create a version tag and push it
```bash
git tag -a TAG_NAME -m "TAG_MESSAGE"
git push hgctpg TAG_NAME
```
