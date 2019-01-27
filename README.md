# Git-related guidelines

The following guidelines are a description of the workflows that are mandatory
for the maintenance of any git repository of the `pyeve` organization.

Though it is still a *draft*.


## General considerations

- The `git` history shall preserve as much information as possible, e.g. about
  merge events, that can be explored with
  `git log --graph --decorate --all --oneline`
- "Now is better than never." is imperative regarding changelogs and authors
  lists and especially with respect to backport fixes to maintenance branches.
- The changelog shall be wrapped at 80 characters for readability on common 
  devices.


## Variable names for the examples

The following placeholders are used below:

- `<pr_id>` is the numerical identifier of a pull request
- `<remote>` is the name of the authorative remote repository on `github.com`
  - usually `upstream` or `origin`


## Merging pull requests into the master branch

1. There is a consent among the reviewers to proceed.
   - One shall of course review own proposed changes, but obviously one's voice
     doesn't count in the review process.
2. Make sure to work on the up-to-date `master` branch:
   `git checkout master && git pull <remote> master`
3. Fetch the desired pull request's changes:
   `git fetch <remote> pull/<pr_id>/head:pull_<pr_id>`
4. Check it out:
   `git checkout pull_<pr_id>`
5. Align it with the state of the `master` branch:
   `git rebase master`
6. Check that there is a descriptive note including references to relevant
   issues in the "Unreleased" section of the changelog and that all involved
   contributors are mentioned in the authors list. Mind to commit adjustments.
7. Push the current state to trigger jobs on the CI:
   `git push <remote> pull_<pr_id>`.
   Depending on the changes and available interpreters, local tests may be
   sufficient or even better.
8. Actually merge and publish the changes:
   `git checkout master && git merge --no-ff pull_<pr_id> && git push <remote> master`
9. If applicable, proceed with "Backporting fixes" below. If so, you may repeat
   this flow with other pull requests before.


## Publishing a major or minor release

1. All related issues and pull requests in the issue tracker are closed.
2. Make sure to work on the up-to-date `master` branch:
   `git checkout master && git pull <remote> master`
3. Update the "Unreleased" section in the changelog and the version in
   `setup.py`.
4. Commit and publish these changes on `github.com`:
   `git commit -a -m "Bumps to version <major>.<minor>" && git push <remote> master`
5. Create a git tag and publish it:
   `git tag <major>.<minor> && git push <remote> <major>.<minor>`
6. Publish the new version on the PyPI:
   `python setup.py sdist`
7. Add a new "Unreleased" section in the changelog, commit and publish this on `master`.
8. Create a new branch for the minor release and publish it:
   `git checkout -b <major>.<minor>.x && git push --set-upstream <remote> <major>.<minor>.x`


## Backporting fixes

1. You have an ordered list of commit hashes (`<hash>`) that need to be
   backported for a minor version.
2. Work on an up-to-date branch for that minor version:
   `git checkout <major>.<minor>.x && git pull <remote> <major>.<minor>.x`
3. Apply the commits:
   `git cherry-pick <hash>...`
  - If a commit is not available locally, fetch the corresponding pull request's objects:
    `git fetch <remote> pull/<pr id>/head`
4. Run tests and update the changelog.
5. Publish the backported changes:
   `git push <remote> <major>.<minor>.x`


## Publishing a micro / bug fix release

1. All related issues and pull requests in the issue tracker are closed.
2. Work on an up-to-date branch for the related minor version:
   `git checkout <major>.<minor>.x && git pull <remote> <major>.<minor>.x`
3. Update the "Unreleased" section in the changelog and the version in
  `setup.py`.
4. Commit and publish these changes on `github.com`:
  `git commit -a -m "Bumps to version <major>.<minor>" && git push <remote> <major>.<minor>.x`
5. Create a git tag and publish it:
   `git tag <major>.<minor>.<micro> && git push <remote> <major>.<minor>.<micro>`
6. Publish the new version on the PyPI:
   `python setup.py sdist`
7. Copy the changelog section of this release to the changelog in the `master`
   branch, commit and publish the result to `<remote>`.
