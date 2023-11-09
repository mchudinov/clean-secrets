# Remove sensitive text from Git history

## Prerequisites

The following tools must be available on on your workstation:

1. Java JRE
2. git
3. wget/curl in order to download the BFG binary
4. Docker is optional if the tools above are not available on your workstation

If you do not have these tools installed a Docker image can be used. Start a Docker container like this:

```sh
docker run -it alpine ash
```

Install the needed software in the Linux Alpine container

```sh
apk update && apk upgrade && apk add git && apk add wget && apk add openjdk11
```

## Process

1. Download the BFG tool


```sh
cd /tmp
wget https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar
cp bfg-1.14.0.jar bfg.jar
```

2. Clone-mirror the repository

```sh
git clone --mirror https://github.com/mchudinov/clean-secrets.git 
```

3. Create the files that contains secrets 

```sh
vi secrets.txt
cat secrets.txt
1_2_3_4_5
6_7_8_9_0
a_b_c_d_e_f
```

4. Remove the secrets from the repository history. Run this command from outside the repo directory. Note that repo is referenced with <repo-name>.git

```sh
java -jar bfg.jar --replace-text secrets.txt  clean-secrets.git
```

5. A report is created after run. You may check changed files and commits

```sh
cat clean-secrets.git.bfg-report/2023-11-09/13-05-50/changed-files.txt
5ba3dcff237b4634fbf5d0ff2da4aef2b2c88973 8200eca6d9edd75a427d0e3f7d93d798497ba27d passwords.yaml
...
```

6. Commit the changes

```sh
cd clean-secrets.git
git reflog expire --expire=now --all && git gc --prune=now --aggressive
git push --force
```

7. Example BFG run output

```sh
java -jar bfg.jar --replace-text secrets.txt  clean-secrets.git

Using repo : /tmp/clean-secrets.git

Found 3 objects to protect
Found 3 commit-pointing refs : HEAD, refs/heads/dev, refs/heads/master

Protected commits
-----------------

These are your protected commits, and so their contents will NOT be altered:

 * commit 78fb2d9c (protected by 'HEAD') - contains 2 dirty files :
        - hidden/passwords.yaml (87 B )
        - hidden/stash/secrets.json (146 B )

WARNING: The dirty content above may be removed from other commits, but as
the *protected* commits still use it, it will STILL exist in your repository.

Details of protected dirty content have been recorded here :

/tmp/clean-secrets.git.bfg-report/2023-11-09/13-05-50/protected-dirt/

If you *really* want this content gone, make a manual commit that removes it,
and then run the BFG on a fresh copy of your repo.


Cleaning
--------

Found 6 commits
Cleaning commits:       100% (6/6)
Cleaning commits completed in 94 ms.

Updating 2 Refs
---------------

        Ref                 Before     After
        ---------------------------------------
        refs/heads/dev    | 0b508c0c | 94f703c1
        refs/heads/master | 78fb2d9c | ec106e3f

Updating references:    100% (2/2)
...Ref update completed in 42 ms.

Commit Tree-Dirt History
------------------------

        Earliest      Latest
        |                  |
         .  D   D  D  m   D

        D = dirty commits (file tree fixed)
        m = modified commits (commit message or parents changed)
        . = clean commits (no changes to file tree)

                                Before     After
        -------------------------------------------
        First modified commit | 2fddae55 | 1d59b3ca
        Last dirty commit     | 0b508c0c | 94f703c1

Changed files
-------------

        Filename         Before & After
        ------------------------------------------------------------------------------
        passwords.yaml | 5ba3dcff ⇒ 8200eca6, a23dc204 ⇒ 1df97237, c482c3dc ⇒ 915bf49b
        secrets.json   | cb51340b ⇒ 947671ae, 9cf06028 ⇒ c93dc3a5, 71f1bfc9 ⇒ dc5accde


In total, 16 object ids were changed. Full details are logged here:

        /tmp/clean-secrets.git.bfg-report/2023-11-09/13-05-50

BFG run is complete! When ready, run: git reflog expire --expire=now --all && git gc --prune=now --aggressive
```