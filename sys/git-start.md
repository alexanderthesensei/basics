## What is git and why do I need it?

![git logo](https://upload.wikimedia.org/wikipedia/commons/2/2b/Git-logo-white.svg)

Git is a distributed version control system designed to be fast and efficient. You can use it to version anything, from code to vector graphics to plain text.

## Why version control?

![folder-versions](../Excalidraw/folder-versions.png)

Imagine you're writing something, be it a program or something in a natural language, and half way through you get the urge to rewrite a piece of it. You delete the paragraph, rewrite it, then in a few days you realize that you like the original version better. Since then, you have closed your editor, so CTRL+Z is no longer an option. Fortunately, you've copied the file before making any changes. You copy it back, and make another copy to keep the second version. Now you have three files. Multiply that by a month of work and you'll get a tree that looks like this

![folder-versions-with-git](../Excalidraw/folder-versions-with-git.png)

Git turns this mess into a single, neat folder with the entire history of changes, each one with a comment, a date, and an author.

## Using git

![a-directory](../Excalidraw/git-showcase/a-directory.png)

Such a git-controlled directory is called a *repository.*

![init](../Excalidraw/git-showcase/init.png)

Let's make one. Git has a command line interface, I'll have the visual representation on the left and a terminal on the right. The command to create a repository is `init`.

![first-status](../Excalidraw/git-showcase/first-status.png)

We can always ask git what state the repository is in using the `status` command. Right now it tells us that there are no saved versions yet and that there is a file that it isn't tracking the changes in. That means that if we say, delete or change the file right now, git will not be able to do anything about it.

![git-add](../Excalidraw/git-showcase/git-add.png)

We can add the file to version control, telling git to track its state. Now it reports that this will be added to a commit. A commit is a snapshot of the repository with the metadata that I talked about earlier.

![git-commit](../Excalidraw/git-showcase/git-commit.png)

If we create that commit, the status goes to "work tree clean", meaning that everything is under control and you won't loose any changes. For example,

![rm-description](../Excalidraw/git-showcase/rm-description.png)

Let's say we accidentally deleted the file. Status shows this now:

![git-restore](../Excalidraw/git-showcase/git-restore.png)

We can ask git to restore the file, which it will do.

![git-log](../Excalidraw/git-showcase/git-log.png)
This is basically 80% of what you do, add, commit, and restore. Imagine we've been working with this project for a bit now and we have multiple files and multiple commits. We can view the list of them using the log command,

![git-checkout](../Excalidraw/git-showcase/git-checkout.png)

Or travel back in time to any of them. This doesn't delete anything, you can always go to the last state, assuming it was committed.

## Conclusion
We'll cover branches next time, I won't go into them now.