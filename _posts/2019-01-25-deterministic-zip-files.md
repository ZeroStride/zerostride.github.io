---
layout: post
title:  "Building Deterministic Zip Files with Built-In Commands"
date:   2019-01-25
categories: blog
---

Zip files are not deterministic by nature, and this can cause some problems when you’re trying to do what you gotta do.

One specific use-case (my use case) was knowing when our continuous integration should deploy a new version of our AWS Lambda functions.

# Why Aren’t Zip Files Deterministic

Non-determinism in zip files is not due to any of the underlying algorithms, but due to the other elements involved in building the zip file.

## “Extra Information”

By default, zip will include “extra information” in the zip file. This depends on OS and such, but can include owner, creation time, access time etc.

```shell
$ openssl rand -base64 32 > a.txt
$ openssl rand -base64 32 > b.txt
$ zip - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
c9623960732a0be4afeee97b5b34f6d1
$ zip - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
857161ad22e6d31460a1ff39a9e1d955
```

The md5s are different, even though the files are the same.

To correct this part of the puzzle, use the `-X` (or `--no-extra`) flag.

```shell
$ openssl rand -base64 32 > a.txt
$ openssl rand -base64 32 > b.txt
$ zip -X - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
54f3bd89a49f0b7d06e387e76c67f77c
$ zip -X - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
54f3bd89a49f0b7d06e387e76c67f77c
```

## Permissions

The permissions on files, with identical contents, will create different zip files.

```shell
$ openssl rand -base64 32 > a.txt
$ openssl rand -base64 32 > b.txt
$ zip -X - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
987e6c2956d6d34a9d6d4999af0a1c31
$ chmod +x a.txt
$ zip -X - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
8d1addee9f37fa71eaa349597222c57f
$ chmod -x a.txt
$ zip -X - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
987e6c2956d6d34a9d6d4999af0a1c31
```

## Timestamp

The `-X` flag still includes the timestamp of the file, so two files with the same contents, but different timestamps, will create different zip files.

```shell
$ openssl rand -base64 32 > a.txt
$ openssl rand -base64 32 > b.txt
$ zip -X - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
9281791dffcb940fa2818722bc79a74c
$ touch -t 201301250000 a.txt
$ zip -X - a.txt b.txt | md5
  adding: a.txt (deflated -3%)
  adding: b.txt (deflated -3%)
6b55b43c7119aa413206b8953763d85c
```

In order to create a deterministic zip file, we need to make sure that the timestamp of each included file stays the same.

To do this we could simply set the timestamp of all the zipped files to a constant value.

```shell
$ openssl rand -base64 32 > a.txt
$ openssl rand -base64 32 > b.txt
$ ls -l
total 16
-rw-r--r--  1 pat  staff    45B Jan 25 11:43 a.txt
-rw-r--r--  1 pat  staff    45B Jan 25 11:43 b.txt
$ find . -exec touch -t 201301250000 {} +
$ ls -l
total 16
-rw-r--r--  1 pat  staff    45B Jan 25  2013 a.txt
-rw-r--r--  1 pat  staff    45B Jan 25  2013 b.txt
```

But in my case, I needed something a bit more clever, and I suspect you will as well...

# Generating Meaningful Timestamps with Git

Setting the timestamp to a constant value does solve the problem of generating a deterministic zip, but it’s not very meaningful. What if we could get the timestamp for the most recent modification to an included file.

We can with git! (Probably other version control also, but that’s left as an exercise for the reader).

This uses the ls-files git command, pipes the output to the log git command and formats the output to just output the author date.

```shell
$ git ls-files -z . | xargs -0 -n1 -I{} -- git log -1 --format="%ad" {}
Sat Jun 17 13:34:28 2017 -0700
Thu Dec 13 14:56:15 2018 -0800
Fri May 5 13:01:16 2017 -0700
Thu Dec 13 14:31:07 2018 -0800
Sat Jun 17 10:32:11 2017 -0700
Mon Jun 19 11:16:41 2017 -0700
Thu May 18 14:38:53 2017 -0700
Tue Oct 9 09:39:04 2018 -0700
Thu Dec 20 07:50:42 2018 -0800
Thu Dec 13 15:47:04 2018 -0800
Mon Jul 10 12:48:59 2017 -0700
Tue Aug 1 15:49:51 2017 -0700
Sat Oct 6 14:48:55 2018 -0700
Wed Jan 23 18:12:33 2019 -0800
Tue Jul 11 21:55:10 2017 -0700
```

We now need to format the date into something we can sort, and provide as input to touch `-t`.

(Using the `--date` flag requires git version 2.6 or higher.)

```shell
$ git ls-files -z . | \
  xargs -0 -n1 -I{} -- git log -1 --date=format:"%Y%m%d%H%M" --format="%ad" {} | \
  sort -r | \
  head -n 1
201901231812
```

We can now put this together with `touch`.

```shell
$ find . -exec touch -t `git ls-files -z . | \
  xargs -0 -n1 -I{} -- git log -1 --date=format:"%Y%m%d%H%M" --format="%ad" {} | \
  sort -r | head -n 1` {} +
```

This will recursively set the timestamp of all files to the timestamp of the timestamp of the last change in git.

# Reusable Command for CircleCI

Here is the code as a reusable command for CircleCI.

```yaml
commands:
  #
  # Deterministic Zip
  #
  zip_r_deterministic:
    description: "Create a deterministic zip of a directory."
    parameters:
      before:
        type: string
        default: ""
      dir:
        type: string
      out:
        type: string
        description: Relative to CIRCLE_WORKING_DIRECTORY
      after:
        type: string
        default: ""
      date:
        type: string
        default: |
          `git ls-files -z . | xargs -0 -n1 -I{} -- git log -1 --date=format:"%Y%m%d%H%M" --format="%ad" {} | sort -r | head -n 1`
    steps:
      - when:
          condition: << parameters.before >>
          steps:
            - run:
                name: Zip Deterministic - PreProcess
                command: |
                  cd << parameters.dir >>
                  << parameters.before >>
      - run:
          name: Zip Deterministic - << parameters.dir >>
          command: |
            cd << parameters.dir >>
            export TEAK_ZIP_R_DETERMINISTIC_DATETIME=<< parameters.date >>
            find . -exec touch -t $TEAK_ZIP_R_DETERMINISTIC_DATETIME {} +
            zip -rX /tmp/teak_zip_r_deterministic.zip .
            touch -t $TEAK_ZIP_R_DETERMINISTIC_DATETIME /tmp/teak_zip_r_deterministic.zip
            cd -
            mv /tmp/teak_zip_r_deterministic.zip << parameters.out >>
      - when:
          condition: << parameters.after >>
          steps:
            - run:
                name: Zip Deterministic - PostProcess
                command: |
                  cd << parameters.dir >>
                  << parameters.after >>
```

This command also sets the timestamp of the resulting zip file to the timestamp of the last changed file that it contains.

This makes it easy to use the aws `s3 sync` to update only when there is an update to the contained code.

```yaml
jobs:
  build:
    docker:
      - image: circleci/node:6.10.2-browsers

    steps:
      - checkout

      - run:
          name: Install AWS CLI
          command: sudo apt-get -y -qq install awscli

      - run:
          name: Update git
          command: |
            sudo sh -c "echo 'deb http://ftp.debian.org/debian jessie-backports main' >> /etc/apt/sources.list.d/git.list"
            sudo apt-get update && sudo apt-get -t jessie-backports install "git"

      - run: mkdir packages
      - zip_r_deterministic:
          before: yarn run prepare-for-lambda-zip
          dir: origin
          out: packages/origin.zip
      - zip_r_deterministic:
          before: yarn run prepare-for-lambda-zip
          dir: edge
          out: packages/edge.zip

      - run:
          name: Upload Lambda Packages to S3
          command: |
            aws s3 sync packages s3://your-lambda-packages
```

I included how to update git on the (very old) version of Debian that CircleCI uses, since that’s a headache.

# Learning!

I hope this helped demystify the process of creating zip files, and some of the reasons why they can be non-deterministic.

If it didn’t, well, write your own damn blog.
