---
title: "Complete Guide: Migrating a Repository from SVN to Git"
date: "2016-01-23T19:00:00Z"
summary: "A step-by-step guide to migrating a project from Subversion (SVN) to Git, preserving commit history and authors."
tags: ["git", "svn", "migration", "devops"]
translationKey: "2016-01-23-migrate-svn-to-git"
---

Migrating a project from Subversion (SVN) to Git is a key step in modernizing your development workflow. Git offers major advantages such as a more flexible branching model, offline work capabilities, and better performance.

This guide details the steps for a clean and complete migration.

### Prerequisites

Ensure that `git` and its `git-svn` extension are installed on your system.

```bash
# On Debian/Ubuntu
sudo apt-get update && sudo apt-get install git-svn
```

### Step 1: Prepare the Author Mapping

SVN only records the username for each commit. Git, however, expects a full name and an email address. We will create a mapping file for this.

Run this command at the root of your SVN project to extract the list of authors:

```bash
svn log --xml | grep author | sort -u | perl -pe 's/.*>(.*?)<.*/$1 = /' > users.txt
```

Open the `users.txt` file and complete it to match the format `SVN Username = Firstname Lastname <email@example.com>`:

```ini
# users.txt file
oxomichael = Michael Oxo <michael@example.com>
anotheruser = Another User <another@example.com>
```

### Step 2: Clone the SVN Repository with `git-svn`

Now, we will clone the SVN repository into a new local Git repository. This command will fetch each SVN revision and turn it into a Git commit, using the `users.txt` file to correctly assign the authors.

*   `--stdlayout`: Indicates that your SVN repository has a standard layout (`trunk`, `branches`, `tags`).
*   `--no-metadata`: Prevents adding SVN metadata to the Git commit messages.
*   `--authors-file`: Specifies the author mapping file.

```bash
git svn clone --stdlayout --no-metadata --authors-file=users.txt http://[PATH_TO_SVN_PROJECT] [LOCAL_PROJECT_NAME]
```
This operation can be very time-consuming for repositories with a large history.

### Step 3: Clean Up and Convert References

The `git-svn` tool creates remote branches and tags that are not native Git branches and tags. We need to convert them.

Navigate into the newly created project folder (`cd [LOCAL_PROJECT_NAME]`) and run the following commands.

**1. Convert SVN tags to Git tags:**

```bash
git for-each-ref refs/remotes/origin/tags | cut -d / -f 5- | grep -v @ | while read tagname; do git tag "$tagname" "refs/remotes/origin/tags/$tagname"; done
```

**2. Convert remote SVN branches to local Git branches:**

```bash
git for-each-ref refs/remotes/origin | cut -d / -f 4- | grep -v 'trunk' | while read branchname; do git branch "$branchname" "refs/remotes/origin/$branchname"; done
```

**3. Clean up old references:**

Now that the branches and tags are converted, we can remove the references created by `git-svn`.

```bash
git branch -r -d origin/trunk
git branch -r -d $(git branch -r | grep "origin/tags")
git branch -r -d $(git branch -r | grep "origin/[^t]") # Deletes remaining branches except trunk
```

### Step 4: Link the Local Git Repository to a Remote

It's time to connect your local repository to a new remote Git repository (on GitHub, GitLab, etc.).

```bash
git remote add origin git@[PATH_TO_GIT_PROJECT]
```

### Step 5: Push the Project to the New Git Repository

Send your entire history, branches, and tags to the new Git server.

```bash
# Push all branches
git push origin --all

# Push all tags
git push origin --tags
```

Your project is now fully migrated to Git! You can archive the SVN repository.