# Setting up Git LFS with Dropbox



## Intro

[git-lfs](https://git-lfs.github.com/) ([git repo](https://github.com/git-lfs/git-lfs)) is a system invented and maintained by GitHub to store large files out of a git repo. Locally, you still work with them the same way as usual, but on the remote repo (on GitHub), these files are actually not in the repo. Only a "fake" file appears there, while the actual file is stored elsewhere.

The main idea is to allow large files (>100Mb) in a Git repo, without hitting the GitHub limit.

On your own machine, once everything is properly configured, you don't have to worry about the details, everything is handled transparently by Git. Only know that some of the files are not actually hosted on the Git server, elsewhere.

Besides the Git repo, Git LFS needs a server/web service to send the large files to. This server must be set up to handle LFS transfers. GitHub offers paid LFS server space where these large files can be hosted. When using GitHub LFS servers, files are resolved/linked transparently with the normal GitHub repo, so the large files are visible and downloadable directly from the GitHub repo, even if they are actually elsewhere.

Instead of a LFS server, it is also possible to use a service like Dropbox or NextCloud. In order to achieve this, your LFS client will, instead of communicate to a LFS server, store and retrieve its files to/from a local folder. It's then up to you to have that folder managed by Dropbox, NextCloud or whatever. When you work with others, each person must have access to that Dropbox/Nextcloud share, and have it synced locally. Then, each person configures their LFS to point to the appropriate local folder.

#### Warnings

There are several things to know beforehand:

* You can enable LFS on an existing Git repo, but adding files to LFS that were already committed before will **rewrite the Git history**. As a result, the commit numbers will differ from other people who have cloned the repo. They will all need to delete it and clone again. Pull requests will also not be automatically mergeable anymore. It's always recommended to start LFS on a fresh repo.
* People who don't have access to the Dropbox folder (clients, etc) will be **unable to download the files**.  Also, if you stop sharing the Dropbox folder, parts of the Git repo become unavailable.
* Choosing files to be handled by LFS can be done only by name. It's therefore not possible to automatically handle, ex. all files bigger than 50Mb. The only solution there is to add them manually one by one.





## Setting things up

#### 1. Install Git

We suppose you did this already...

#### 2. Install Git LFS

On **Windows**: Check the [Git-LFS releases page](https://github.com/git-lfs/git-lfs/releases) and download the latest installer, if you wish to install git-lfs system-wide. Alternatively, download the Windows AMD64 zip file, extract the *git-lfs.exe* file and place it into the Git repo of your choice. This has the advantage of being ready for other users of your repo, who won't need to install git-lfs.

On **Linux**: Git-LFS is available in most distributions package managers. Alternatively, download it from the [Git-LFS releases page](https://github.com/git-lfs/git-lfs/releases) and place it in the Git repo or somewhere in your exectuables path.

#### 3. Install LFS-folderstore

[LFS-folderstore](https://github.com/sinbad/lfs-folderstore) is a plugin for Git-LFS that allows it to store and retrieve files to/from a local folder, instead of communicating with a server.

On **Windows** and **Linux**: Head to the [LFS-folderstore releases page](https://github.com/sinbad/lfs-folderstore/releases) and download the appropriate version for your operating system. Install it simply by placing the executable file (lfs-folderstore.exe) either inside the  Git repo (same advantage as Git-LFS above) or somewhere else on your system. It doesn't really matter where, because the path to that file will need to be configured for each repo. Just note where you placed it.

#### 4. Test that LFS is working

Open a terminal, navigate to an existing Git repo or any place where the git-lfs.exe is installed (or anywhere else if you are on Linux and have installed Git-LFS via your package manager), and issue:

```bash
git lfs
```

Git LFS should print a help text, which shows everything is properly installed.

#### 5. Testing with this repo

This repo is configured with LFS. The [test file.FCStd](test file.FCStd) file is present in the repo, but not actually there. It is on a Dropbox folder. To test, ask [@yorikvanhavre](https://github.com/yorikvanhavre) for a Dropbox share link.

1. Make sure you have obtained the the Dropbox link, and downloaded or synced the folder contents somewhere.
2. Clone this repo: `git clone https://github.com/yorikvanhavre/LFS-templte`
3. Verify that the [test file.FCStd](test file.FCStd) contains text
3. Set up LFS and folderstore:
```
git config --add lfs.customtransfer.lfs-folder.path "C:/path/to/lfs-folderstore.exe"
git config --add lfs.customtransfer.lfs-folder.args "C:/path/to/dropbox/shared_folder"
git config --add lfs.standalonetransferagent lfs-folder
git config lfs.url "https://localhost"
```
4. Retrigger an update: `git reset --hard main`
5. Verift that the  [test file.FCStd](test file.FCStd) now is an actual FreeCAD file that can be opened with FreeCAD.



## Working with Git-LFS and LFS-folderstore

Below is a more detailed explanation:

Make sure you have everything you need: either a test repo and some large files to test, and a Dropbox/Nextcloud folder set up (share it with others.

#### 1. Configure a fresh repo 

Starting a new repository is the easiest case. The instructions below also work if you have a repo which was already Git-versioned, but where no big files have been committed yet.

* Initialise your repository as usual with `git init`
* Add some files to be tracked by Git-LFS. Examples:
    * `git lfs track *.rvt` will track all the .rvt files found in the repo at this moment (.rvt files added after this command is run **will not be tracked**).
    * `git lfs track "*.rvt"` (note the double quotes) will track all the .rvt found in the repo **at any moment**. .rvt files added later will be tracked too.
    * `git lfs untrack` does the contrary of `git lfs track`, and works the same way (with or without double quotes).
    * Each time after using `git lfs track` or `untrack`, the `.gitattributes` file will be modified. You will need to commit it.

* Create some commits with some big files (.rvt files for example, to follow above rules)
* Add your normal git remote using `git remote add origin <url>`, as instructed on GitHub, Gitlab, etc
* Run these commands to configure your LFS folder (on Linux, replace lfs-folderstore.exe with lfs-folderstore)

```
git config --add lfs.customtransfer.lfs-folder.path "C:/path/to/lfs-folderstore.exe"
git config --add lfs.customtransfer.lfs-folder.args "C:/path/to/dropbox/folder"
git config --add lfs.standalonetransferagent lfs-folder
git config lfs.url "https://localhost"
```
* Push your changes: `git push origin master`. This will also copy the LFS files from your Git repo folder to your Dropbox folder.

**Notes for Windows users:**

* As shown above, even on Windows, **use forward slashes** for path separators, instead of backslashes
* If you have spaces in your path, add **additional single quotes** around the path, e.g. `git config --add lfs.customtransfer.lfs-folder.args "'C:/path with spaces/folder'"`

#### 3. Cloning an existing repo

There is one problem when cloning a repository which contains LFS files: git LFS does not know how to fetch the LFS content, until you configure things using `git config`. That's because the remote Git repo doesn't contain any information about where LFS files actually live  (that's different when using GitHub's LFS servers...).

It's not that hard to resolve though, you just need a couple of extra steps after you cloned. Here's the sequence:

* Start with cloning the repo: `git clone <url> <folder>`. This will work for the git data, but will report "Error downloading object" when trying to get LFS data
* Enter your newly cloned repo: `cd <folder>` 
* Configure as with a new repo:
```
git config --add lfs.customtransfer.lfs-folder.path "C:/path/to/lfs-folderstore.exe"
git config --add lfs.customtransfer.lfs-folder.args "C:/path/to/dropbox/folder"
git config --add lfs.standalonetransferagent lfs-folder
git config lfs.url "https://localhost"
```


* With LFS now properly set up,  retrigger the download of the LFS files:`git reset --hard master` (or `main`). The LFS files will now be copied from your Dropbox folder to your Git repo folder.

#### 4. Notes

* The shared folder is, to git, still a remote location. It only interacts with it during `fetch`, `pull` and `push`. Committing a LFS file is not enough to have it copied to the Dropbox folder. You also need to push as if it was a remote.
* Actual copies are always made in all cases, even if your Git repo folder is also managed by Dropbox. No links are used ever. Note that Dropbox and most other file storing platforms actually detect identical files and store them only once on their servers (this is actually even happening between different users...)
* You can use one single Dropbox folder for all your LFS-handled files, or different ones. Files are not saved under their actual name but under a git commit id number, so there is no risk of overwriting.



## Links

* [[Why You Shouldn't Use Git LFS](https://gregoryszorc.com/blog/2021/05/12/why-you-shouldn't-use-git-lfs)](https://gregoryszorc.com/blog/2021/05/12/why-you-shouldn%27t-use-git-lfs/)
* [Git LFS – Migrate from GitHub storage to Dropbox using lfs-folderstore](https://www.aurumnova.com/life-hacks/git-lfs-migrate-from-github-storeage-to-dropbox-using-lfs-folderstore/)
* [Git LFS: Shared Folder agent](https://github.com/sinbad/lfs-folderstore)
