# How to convert Mercurial Repositories with subrepos

## Introduction

hg-fast-export supports migrating mercurial subrepositories in the
repository being converted into git submodules in the converted repository.

Git submodules must be git repositories while mercurial's subrepositories can
be git, mercurial or subversion repositories. hg-fast-export will handle any
git subrepositories automatically, any other kinds must first be converted
to git repositories. Currently hg-fast-export does not support the conversion
of subversion subrepositories. The rest of this page covers the conversion of
mercurial subrepositories which require some manual steps:

The first step for mercurial subrepositories involves converting the
subrepository into a git repository using hg-fast-export.  When all
subrepositories have been converted, a mapping file that maps the mercurial
subrepository path to a converted git submodule path must be created. The
format for this file is:

    "<mercurial subrepo path>"="<git submodule path>"
    "<mercurial subrepo path2>"="<git submodule path2>"
    ...

The path of this mapping file is then provided with the --subrepo-map
command line option.

## Example

Example mercurial repo folder structure (~/mercurial) containing two subrepos:

    src/...
    subrepos/subrepo1
    subrepos/subrepo2

### Setup
Create an empty new folder where all the converted git modules will be imported:

    mkdir ~/imported-gits
    cd ~/imported-gits

### Convert all submodules to git:
    mkdir submodule1
    cd submodule1
    git init
    hg-fast-export.sh -r ~/mercurial/subrepos/subrepo1
    cd ..
    mkdir submodule2
    cd submodule2
    git init
    hg-fast-export.sh -r ~/mercurial/subrepos/subrepo2

### Create mapping file
    cd ~/imported-gits
    cat > submodule-mappings << EOF
    "subrepos/subrepo1"="../submodule1"
    "subrepos/subrepo2"="../submodule2"
    EOF

### Convert main repository
    cd ~/imported-gits
    mkdir git-main-repo
    cd git-main-repo
    git init
    hg-fast-export.sh -r ~/mercurial --subrepo-map=~/imported-gits/submodule-mappings

### Result
The resulting repository will now contain the submodules at the paths
`subrepos/subrepo1` and `subrepos/subrepo2`. The created .gitmodules
file will look like:

    [submodule "subrepos/subrepo1"]
          path = subrepos/subrepo1
          url = ../submodule1
    [submodule "subrepos/subrepo2"]
          path = subrepos/subrepo2
          url = ../submodule2
