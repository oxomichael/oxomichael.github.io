---
title: "Complete Guide: Migrating an SVN Repository to Git"
date: "2016-01-23T19:00:00Z"
summary: "A step-by-step guide to migrating a project from Subversion (SVN) to Git, preserving commit history and authors."
tags: ["git", "svn", "migration", "devops"]
translationKey: "2016-01-23-migrate-svn-to-git"
---

Migrating a project from Subversion (SVN) to Git is a key step in modernizing development processes. Git offers major advantages such as a more flexible branching model, offline work capabilities, and better performance.

This guide details the steps for a clean and complete migration.

### Prerequisites

Ensure that `git` and its `git-svn` extension are installed on your system.

```bash
# On Debian/Ubuntu
sudo apt-get update && sudo apt-get install git-svn
```

### Step 1: Prepare the Author Mapping

SVN only records the username for each commit. Git, however, expects a full name and an email address. We will create a mapping file.

Run this command at the root of your SVN project to extract the list of authors:

```bash
svn log --quiet | grep -E "r[0-9]+ \| .* \|" | cut -d'|' -f2 | sed 's/ //g' | sort -u > users.txt
```

Open the `users.txt` file and complete it to match the format `SVN Username = First Name Last Name <email@example.com>`:

```ini
# users.txt file
oxomichael = Michael Oxo <michael@example.com>
anotheruser = Another User <another@example.com>
```

### Step 2: Clone the SVN Repository with `git-svn`

Now, we will clone the SVN repository into a new local Git repository. This command will fetch each SVN revision and transform it into a Git commit, using the `users.txt` file to correctly attribute the authors.

*   `--stdlayout`: Indicates that your SVN repository has a standard layout (`trunk`, `branches`, `tags`).
*   `--no-metadata`: Prevents adding SVN metadata to Git commit messages.
*   `--authors-file`: Specifies the author mapping file.

```bash
git svn clone --stdlayout --no-metadata --authors-file=users.txt http://[SVN_PROJECT_PATH] [LOCAL_PROJECT_NAME]
```
This operation can be very time-consuming for repositories with a large history.

### Step 3: Clean Up and Convert References

The `git-svn` tool creates remote branches and tags that are not native Git branches and tags. We need to convert them.

Navigate to the newly created project folder (`cd [LOCAL_PROJECT_NAME]`) and run the following commands.

**1. Convert remote SVN branches to local Git branches:**

```bash
git for-each-ref refs/remotes/origin | grep -v 'trunk' | cut -d / -f 4- | while read branchname; do git branch "$branchname" "refs/remotes/origin/$branchname"; done
```

**2. Convert SVN tags to Git tags:**

```bash
git for-each-ref refs/remotes/origin/tags | cut -d / -f 5- | while read tagname; do git tag "$tagname" "refs/remotes/origin/tags/$tagname"; done
```

**3. Rename the `trunk` branch to `main`:**

The `trunk` branch in SVN is the equivalent of the main branch in Git. Let's rename it to `main` to follow modern conventions.

```bash
git branch -m trunk main
```

### Step 4: Link the Local Git Repository to a Remote Repository

It's time to connect your local repository to a new remote Git repository (on GitHub, GitLab, etc.).

```bash
git remote add origin git@[GIT_PROJECT_PATH]
```

### Step 5: Push the Project to the New Git Repository

Send your entire history, branches, and tags to the new Git server.

```bash
# Push the main branch and all other branches
git push origin --all

# Push all tags
git push origin --tags
```

Your project is now fully migrated to Git! You can archive the SVN repository.
