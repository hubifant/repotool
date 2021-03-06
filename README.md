# repotool - a tool to aggregate source code repositories metadata

[![Build Status](https://travis-ci.org/DevMine/repotool.png?branch=master)](https://travis-ci.org/DevMine/repotool)
[![GoDoc](http://godoc.org/github.com/DevMine/repotool?status.svg)](http://godoc.org/github.com/DevMine/repotool)
[![GoWalker](http://img.shields.io/badge/doc-gowalker-blue.svg?style=flat)](https://gowalker.org/github.com/DevMine/repotool)

`repotool` is a command line tool that aggregates source code repositories
metadata (such as VCS type, commits and so on) and produces JSON objects out of
it.  It is also able to store repository information into a database.

A repository contains a list of commits that may contain, if you enable this
option, a list of deltas. A delta contains information about a file
touched by a commit. It may also contain patches if specified via an option.

Currently, only [git](http://git-scm.com/) is supported.

Below is an example of the data produced, without commit deltas and patches:

```
{
  "name": "repotool",
  "vcs": "git",
  "clone_url": "https://github.com/DevMine/repotool.git",
  "clone_path": "/home/robin/Hacking/repotool",
  "default_branch": "master",
  "commits": [
    {
      "vcs_id": "df55def5e6185447c6bd360ec1144a847d73b986",
      "message": "repotool: Add possibility to insert commit diff deltas into the db.\n\nFor this purpose, create a new 'commit_diff_deltas' table.\n",
      "author": {
        "name": "Robin Hahling",
        "email": "robin.hahling@gw-computing.net"
      },
      "committer": {
        "name": "Robin Hahling",
        "email": "robin.hahling@gw-computing.net"
      },
      "author_date": "2015-01-14T18:12:47+01:00",
      "commit_date": "2015-01-14T18:12:47+01:00",
      "file_changed_count": 2,
      "insertions_count": 89,
      "deletions_count": 2
    },
    ...
  ]
}
```

And with deltas enabled (without patches):

```
{
  "name": "repotool",
  "vcs": "git",
  "clone_url": "https://github.com/DevMine/repotool.git",
  "clone_path": "/home/robin/Hacking/repotool",
  "default_branch": "master",
  "commits": [
    {
      "vcs_id": "863f9ed113f06829359d0fd4040ae4a6b5c1cf5e",
      "message": "tools/batch: Use a channel to create a pool of tasks for goroutines.\n\nUse a channel on which each tasks (ie call to repotool) is added.\nThis allows to have goroutines picking up tasks from the channel as soon\nas they are done. This way, there is no waiting time as long as there\nare tasks in the pool.\n",
      "author": {
        "name": "Robin Hahling",
        "email": "robin.hahling@gw-computing.net"
      },
      "committer": {
        "name": "Robin Hahling",
        "email": "robin.hahling@gw-computing.net"
      },
      "author_date": "2015-01-13T15:24:40+01:00",
      "commit_date": "2015-01-13T15:24:40+01:00",
      "diff_delta": [
        {
          "status": "modified",
          "binary": false,
          "old_file_path": "tools/batch.go",
          "new_file_path": "tools/batch.go"
        }
      ],
      "file_changed_count": 1,
      "insertions_count": 25,
      "deletions_count": 23
    },
    ...
  ]
}
```

And you can even include patches:

```
{
  "name": "repotool",
  "vcs": "git",
  "clone_url": "https://github.com/DevMine/repotool.git",
  "clone_path": "/home/robin/Hacking/repotool",
  "default_branch": "master",
  "commits": [
    {
      "vcs_id": "fe8aaac0c7650d8ce9c8f4ddeaa63105b3dd0e9e",
      "message": "repotool: Print repository name before processing db insertions.\n",
      "author": {
        "name": "Robin Hahling",
        "email": "robin.hahling@gw-computing.net"
      },
      "committer": {
        "name": "Robin Hahling",
        "email": "robin.hahling@gw-computing.net"
      },
      "author_date": "2015-01-14T18:14:18+01:00",
      "commit_date": "2015-01-14T18:14:18+01:00",
      "diff_delta": [
        {
          "patch": "diff --git a/repotool.go b/repotool.go\nindex ba1eed0..d1ce7a3 100644\n--- a/repotool.go\n+++ b/repotool.go\n@@ -97,8 +97,9 @@ func main() {\n \t\t}\n \t\tdefer db.Close()\n \n-\t\tfmt.Fprintf(os.Stderr, \"inserting %d commits into the database...\\n\",\n-\t\t\tlen(repository.GetCommits()))\n+\t\tfmt.Fprintf(os.Stderr,\n+\t\t\t\"inserting %d commits from %s repository into the database...\\n\",\n+\t\t\tlen(repository.GetCommits()), repository.GetName())\n \t\ttic := time.Now()\n \t\tinsertRepoData(db, repository)\n \t\ttoc := time.Now()\n",
          "status": "modified",
          "binary": false,
          "old_file_path": "repotool.go",
          "new_file_path": "repotool.go"
        }
      ],
      "file_changed_count": 1,
      "insertions_count": 3,
      "deletions_count": 2
    },
    ...
  ]
}
```

## Installation

`repotool` depends on [git2go](https://github.com/libgit2/git2go), which is a
[Go](http://golang.org/) binding to [libgit2](https://libgit2.github.com/), a C
library that implements `git` core methods.

`git2go` needs to be installed as described in their
[README.md](https://github.com/libgit2/git2go/blob/master/README.md#installing)
in order to statically link `libgit2` to the `git2go` package.

To install `repotool`, run this command in a terminal, assuming
[Go](http://golang.org/) is installed:

    go get github.com/DevMine/repotool

Or you can download a binary for your platform from the DevMine project's
[downloads page](http://devmine.ch/downloads).

## Usage

`repotool` produces JSON, provided that you feed it with a path to a source code
repository. By default, informative messages are outputted to `stderr` whereas
JSON is outputted to `stdout`. Example usage:

    repotool ~/Code/myawesomeproject > myawesomeproject.json

To insert data into the PostgreSQL database, you need to provide a configuration
file in argument. Simply copy `repotool.conf.sample` to `repotool.conf` and
adjust database connection information. See this
[README.md](https://github.com/DevMine/repotool/blob/master/db/README.md) for
more information about the database schema. Example usage:

    repotool -json=false -db -c repotool.conf ~/Code/myawesomeproject

With the configuration file, you can also tell `repotool` to output commit
deltas and commits patches (the latter works only if you enable commit deltas,
quite logically). Simply set the `commit_deltas` and, eventually,
`commit_patches`, to `true`. Be careful with the `commit_patches` option as this
may rapidly produce a lot of data. Also note that if you plan on inserting data
into a database, patches are not going to be inserted, whether you set
`commit_patches` to `true` or not.

If you plan on batch processing multiple repositories, look at `batch.go` in the
`tools` folder. It can process repositories concurrently by recursively
traversing directories and calling `repotool`, spawning goroutines in the
process.  When using it, bear in mind that `repotool` is IO and CPU intensive,
hence do not spawn too many goroutines or you might reach the number of open
files limit. The number of goroutines can be adjusted with the `-g` parameter.
Using about the same number of goroutines as the number of cpu cores should be a
reasonable choice.
