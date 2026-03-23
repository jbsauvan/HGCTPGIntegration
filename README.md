This repository documents conventions and procedures related to the HGCAL TPG simulation code management and integration. It includes at the moment the following information:
- The branches structure and naming conventions
- The list and details of the steps to integrate new developments
- Checklists for the different integration steps

This repository may host, in the future, scripts in order to streamline the integration process.

## Branches structure and conventions
Three types of branches are used:
- Development branches
- Release branches
- Integration branches

In addition, specific versions of the code are tagged on the release branches.

### Development branches
The development branch is the entry point of all new developments. Pull Requests (PR) are made on this branch, and this is where the review and integration process start.

Development branches are based on CMSSW pre-releases and meant to be used primarily by developers. New development branches are created when new CMSSW pre-releases are available. At a given point in time there is only one development branch being used (the one corresponding to the most recent CMSSW pre-release), which means that on-going developments need to be ported to newly created development branches (see the following [Section](#moving-to-more-recent-development-and-release-branches) for information on this porting). **The development branch being used at a given moment is defined as the `default` branch in Github**.

Development branches should follow to some extent the evolution of the CMSSW pre-releases, although the creation of a new development branch is in general not necessary for every CMSSW pre-release. 

Development branch names follow the pattern `hgc-tpg-devel-$CMSSW_VERSION` where `$CMSSW_VERSION` is the CMSSW version name, for instance `hgc-tpg-devel-CMSSW_16_1_0_pre2. :warning: The PR validation scripts extract the release version from the branch name, so this pattern must be respected for the validation to work.

### Release branches
Release branches are based on stable CMSSW releases and meant to be used by users for performance studies. Similarly as for development branches, new release branches are created to follow new CMSSW stable releases. But contrary to development branches, there can be more than one release branch being used at a given time. It can happen if an older CMSSW version is required to run on some set of samples (e.g. because of deprecated geometries being used in those samples), and a newer CMSSW version is used to run on more recent samples.

Release branch names follow the pattern `hgc-tpg-$CMSSW_VERSION` where `$CMSSW_VERSION` is the CMSSW version name, for instance `hgc-tpg-CMSSW_16_0_1. :warning: The PR validation scripts extract the release version from the branch name, so this pattern must be respected for the validation to work.


### Integration branches
Integration branches are branches used for integration to the central CMSSW repository ([`cms-sw/cmssw`](https://github.com/cms-sw/cmssw)). Such branches are created to integrate a set of developments to central CMSSW, and are typically based on the latest CMSSW Integration Build (IB).

No naming convention is strictly needed for such branches, but the pattern `hgc-tpg-integration-DATE`, where `DATE` is the date of the branch creation , has been used so far, e.g. `hgc-tpg-integration-240112`. 


### Tags / Releases
For every piece of developments integrated in the release branch, a tag / release is created. The version numbers are of the form `X.Y.Z`:
- `Z` is used for bug fixes, code cleaning and small updates that don't impact trigger primitives 
- `Y` is used for larger updates, and more generally for any update that impact trigger primitives
- `X` is used for major infrastructure changes

The tag name should contain the version number and an identifier of the CMSSW release on which it is based. So far tag names have been following the pattern `vX.Y.Z_ID`, where `ID` is a shortened identifier of the CMSSW release (for instance `1252p1` for `CMSSW_12_5_2_patch1`). 

## Overview of the life cycle of new developments
Developments are tracked in the Github project [`HGCAL TPG Simulation Developments`](https://github.com/orgs/hgc-tpg/projects/1) within `hgc-tpg`. The following workflow statuses have been defined:
- `Inbox` for open issues and developments that haven't started yet
- `Stalled` for developments that have started but are not actively worked on at the moment
- `In Progress` for developments being worked on. There can be a PR being reviewed associated to these developments.
- `Ready for Integration` for developments that have been finalised and integrated in `hgc-tpg`, and are ready to be integrated in the central CMSSW repository
- `Integration in Progress` for developments that have been PRed to the central CMSSW repository and are being reviewed there
- `Integrated` for developments that have been integrated in the central CMSSW. 

To be noted that issues that have been closed and not updated in the last 2 weeks are automatically archived in the project. See the [Project Workflow page](https://github.com/orgs/hgc-tpg/projects/1/workflows/) for more details on automated workflows.

Developments follow the following cycle:
- They are first PRed to the latest development branch. They are reviewed and validation scripts are triggered.
- One reviewed and validated they are integrated in the latest development branch
- They are backported to the release branch and a new version / tag is created
- Once a set of developments is judged to be mature enough, this set of developments (which can correspond to one or several PRs in `hgc-tpg`) is rebased to an integration branch and PRed to the central CMSSW repository
- Finally they follow the central CMSSW review process

## Issues and Labels
Github issues are used for:
- Reporting issues, suggesting code improvements or new features
- Tracking central CMSSW integration in progress
- Tracking changes which have been made in HGCAL TPG packages directly in the central CMSSW repository (in general for CMS-wide code maintenance), and that could eventually collide with changes made in `hgc-tpg` and not integrated yet in central CMSSW.

Several Github labels are defined to distinguish issues (and PRs). Among these, some are specific to our workflow: 
- `cmssw integration`, used for issues tracking code integrations in central CMSSW
- `backport`, used for PRs backporting new code to the release branches
- `upstream change`, used for issues tracking changes in our codebase that have been made upstream in central CMSSW

## Integration of new developments
Before being integrated, new developments must be PRed to the latest development branch. No direct pushes are allowed, which is enforced through branch protection. New PRs are automatically entered in the Github project `HGCAL TPG Simulation Developments`, through project workflows.

For every new PR and PR update, a Jenkins CI job is triggered and runs validation scripts (defined in [`hgc-tpg/HGCTPGValidation`](https://github.com/hgc-tpg/HGCTPGValidation)) which :
- Compile the proposed code
- Run code quality checks (based on `scram code-checks` and `scram code-format`)
- Run the HGCAL TPG simulation default sequence, and possibly alternative custom sequences, based on the proposed code
- Also runs the default (and possibly custom) sequences based on the target development branch (which serves as reference to be compared with the proposed code)
- Compare a set of histogrammed quantities between the proposed code and the reference code, which can help spotting unexpected changes. This comparison is published in the HGCAL TPG Validation [webpage](https://validation.llrhgcal.in2p3.fr/prod/). 

In addition to the automated validation scripts, it is recommended to have a review of the code before integration, although that is not enforced. The first guiding principle for the review should be to make sure that the intent of any piece of code is clear and can be understood by a future developer. The intent should ideally be expressed in the code itself (minimally with e.g. clear variable and function names and no magic numbers), and eventually in comments if necessary. Once the intent of the code is clear it should be checked that it is doing what it is intended to do.

More complete guidelines can be found in:
- [CMSSW guidelines](https://cms-sw.github.io/cms_coding_rules.html)
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)

Once the proposed code is judged to be mergeable, it can be integrated in the target development branch. In order to have a clear and linear commit history, as well as to simplify the porting of code, all the commits from a PR are **squashed into a single commit** onto the development branch (which is done using `Squash and merge` in Github).  
When the PR is merged, it is automatically flagged as `Ready for integration` in the Github project `HGCAL TPG Simulation Developments` by the project workflow.

## Backporting and releasing
New development integrated in the development branch are backported to the latest release branches. This is done through backport PRs onto the release branches. Having backport PRs is useful in order run the validation with the release branch and check that there is no incompatibility between the new code and what is in the release branch.

Backport PRs are conventionally identified with a flag in the PR title, e.g. `[Backport CMSSW_16_0_1]`, and with the PR label `backport`. Backport PRs don't need to be included in the Github project `HGCAL TPG Simulation Developments` as the original PRs are already tracked there.

Once the backport PR is validated it can be integrated into the release branch, which is done with **rebasing in order to keep the commit history linear** (using `Rebase and merge` in Github). Then a new version tag is created on the release branch and the user installation recipe is updated in the Twiki page.  

A checklist for backporting and releasing is available in [`checklists/backport.md`](checklists/backport.md).

## Moving to more recent development and release branches
Our developments need to follow evolutions in CMSSW, which means that our development branch needs to be based on a recent CMSSW pre-release as much as possible. It implies regularly porting developments existing in a development branch based on a given pre-release to a more recent pre-release. 

To do so, a new development branch based on the most recent pre-release is created and the set of developments / commits which have not been integrated yet in this pre-release are picked and rebased there. :warning: This requires some care in order to avoid picking commits already integrated between the old pre-release and the new pre-release.

Once a new development branch has been created, the `default` branch in Github is changed to this new development branch and the developer installation recipe is updated in the Twiki page.

A checklist for moving to a new development branch is available in [`checklists/new-dev-branch.md`](checklists/new-dev-branch.md).
The same process applies when migrating to a new release branch.

## Integration in central CMSSW
Integration in central CMSSW is done when a set of developments is judged to be mature enough. This is done from an `hgc-tpg` integration branch following the standard CMSSW integration process. An integration branch is built from the latest CMSSW IB and the commits to be integrated are **rebased** on it. 

**Important note** :warning: : data files (in the `data/` directory) should not be integrated into CMSSW, they should go in [`cms-data/L1Trigger-L1THGCal`](https://github.com/cms-data/L1Trigger-L1THGCal). Therefore any new data file should be removed from the git history before the PR to central CMSSW is made. Data files should not only be git removed, but their removal commit should also be squashed with the commit where they appeared, in order to make them disappear from git history. These data files should be PRed to `cms-data/L1Trigger-L1THGCal` in parallel with the PR made to CMSSW.

The `cms-sw` integration PR description should ideally link to all the related `hgc-tpg` PRs in order to keep track of the individual internal reviews. It should also ideally link to related and relevant presentations in meetings, if any. Once an integration PR has been created, a dedicated `cmssw integration` issue is created in `hgc-tpg` with a link to the integration PR, so that it can easily be tracked. Links to the associated `hgc-tpg` internal PR can also be added to the issue description for easier internal references.

**Important note** :warning: : if data files have been PRed to `cms-data/L1Trigger-L1THGCal`, it should be mentioned in the CMSSW PR message that it depends on external data, with a link to the `cms-data` PR, so that tests can be run using these external data.

In the central review process, changes can be requested. In that case, these changes can eventually be backported to our development branch before the development branch is migrated to a pre-release where these changes are included.

A checklist for integrating development in central CMSSW is available in [`checklists/integration.md`](checklists/integration.md).
