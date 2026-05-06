### Install the working area
```bash
cmsrel <NEW_RELEASE_NAME>
cd <NEW_RELEASE_NAME>/src
cmsenv
```
Replacing `<NEW_RELEASE_NAME>` with the name of the new CMSSW release on which the development branch will be rebased.

### If nothing has been integrated in the new release compared to the old one
-  One can simply do a `rebase-topic`
```bash
git cms-rebase-topic hgc-tpg:hgc-tpg-devel-<OLD_RELEASE_NAME>
git checkout -b hgc-tpg-devel-$CMSSW_VERSION
```
Replacing `<OLD_RELEASE_NAME>` with the name of the old CMSSW release on which the development branch is currently based.

### Otherwise the commits need to be picked manually
- Initialize the repository
```bash
git cms-init
git cms-addpkg L1Trigger/L1THGCal
git cms-addpkg L1Trigger/L1THGCalUtilities
git cms-addpkg DataFormats/L1THGCal
```
- Retrieve current development branch and intialize new one
```bash
git remote add hgctpg git@github.com:hgc-tpg/cmssw.git
git fetch hgctpg
git checkout -b hgc-tpg-devel-$CMSSW_VERSION
```
- Pick and rebase commits that have not been integrated yet in this `$CMSSW_VERSION`
```bash
# pick last commit of the release (M)
git log --oneline -n 5
# pick commits to be rebased (X..Y)
git log --oneline hgctpg/OLD_DEVEL_BRANCH_NAME
# Interactive rebase to select a subset of the commits 
git rebase -i --onto M <commit before X> Y
# Now we are on detached HEAD, come back to devel branch
git rebase HEAD hgc-tpg-devel-$CMSSW_VERSION
```

### Test and finalize
- Check compilation
```bash
git cms-checkdeps -a -A
scram b clean; scram b -j10
```
- Run test config
```bash
cd L1Trigger/L1THGCalUtilities/test
cmsRun testHGCalL1T_RelValV16_cfg.py
cmsRun testHGCalL1T_multialgo_V16_cfg.py
```
- Push the new development branch and make it the default branch
```bash
git remote add hgctpg git@github.com:hgc-tpg/cmssw.git # if not done already
git push hgctpg hgc-tpg-devel-$CMSSW_VERSION
```

