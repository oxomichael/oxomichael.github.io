---
title: "Guide: Migrating a Git Repository to Another Server"
date: "2016-02-04T20:59:00Z"
summary: "A simple and effective guide to migrating a complete Git repository, including all branches and tags, from one server to another."
tags: ["git", "migration", "devops"]
translationKey: "2016-02-04-migrate-git-to-another-git"
---

Migrating a Git repository from one server to another is a common operation, whether it's to switch platforms (from GitHub to GitLab, for example) or to consolidate projects. The goal is to transfer the entire history, including all branches and tags.

The `mirror` method is the most reliable and comprehensive for this task.

### Step 1: Clone the Repository in Mirror Mode

The first step is to create a "mirror" clone of your source repository. A mirror clone is an exact copy of the Git database, including all internal references, remote branches, and tags.

```bash
# Replace [SOURCE_REPOSITORY_URL] with the URL of your old repository
git clone --bare [SOURCE_REPOSITORY_URL]
```

This command creates a folder ending in `.git` (e.g., `my-project.git`).

### Step 2: Push the Mirror to the New Server

Next, navigate into the newly created folder and push the mirror to the new repository. Make sure you have already created an empty project on your new Git server to receive the data.

```bash
# Enter the mirror clone directory
cd my-project.git

# Replace [NEW_REPOSITORY_URL] with the URL of your new repository
git push --mirror [NEW_REPOSITORY_URL]
```

The `git push --mirror` command ensures that all references (branches, tags, etc.) are pushed to the new repository. It is the safest method for a complete migration.

### Step 3: Cleanup

Once the migration is complete, you can delete the local `.git` folder.

```bash
cd ..
rm -rf my-project.git
```

Your repository is now fully migrated to the new server. You can clone the project from its new URL and continue working normally.