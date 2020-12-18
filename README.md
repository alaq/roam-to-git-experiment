# Automatic RoamResearch backup to a digital garden

First, create a note in Roam called Public and reference all the notes that you want to publish (use the `[[Notation]]`, `#This` won't work)

Then, clone this repo and mark it as private: <https://github.com/maximevaillancourt/digital-garden-jekyll-template>

Edit the Jekyll config in the repo (`_config.yml`) to contain:
`exclude: ['_includes/notes_graph.json', 'markdown', 'formatted', 'json', 'LICENSE', 'default.nix']`

Then configure GitHub secrets as per the instructions here: <https://github.com/MatthieuBizien/roam-to-git#configure-github-secrets>

And finally, create `.github/workflows/main.yml` with the following content:

```yaml
name: "Roam Research backup"
on:
  push:
    branches:
      - master
  schedule:
    -   cron: "0 * * * *"
jobs:
  backup:
    runs-on: ubuntu-latest
    name: Backup
    timeout-minutes: 15
    steps:
      -   uses: actions/checkout@v2
      -   name: Set up Python 3.8
          uses: actions/setup-python@v1
          with:
            python-version: 3.8
      -   name: Setup dependencies
          run: |
            pip install git+https://github.com/DoomHammer/roam-to-git.git@roam-to-garden
      -   name: Run backup
          run: roam-to-git --skip-git .
          env:
            ROAMRESEARCH_USER: ${{ secrets.ROAMRESEARCH_USER }}
            ROAMRESEARCH_PASSWORD: ${{ secrets.ROAMRESEARCH_PASSWORD }}
            ROAMRESEARCH_DATABASE: ${{ secrets.ROAMRESEARCH_DATABASE }}
      -   name: Commit changes
          uses: elstudio/actions-js-build/commit@v3
          with:
            commitMessage: Automated snapshot
# Automatic RoamResearch backup

[![Roam Research backup](https://github.com/MatthieuBizien/roam-to-git-demo/workflows/Roam%20Research%20backup/badge.svg)](https://github.com/MatthieuBizien/roam-to-git-demo/actions)
[![roam-to-git tests.py](https://github.com/MatthieuBizien/roam-to-git/workflows/roam-to-git%20tests.py/badge.svg)](https://github.com/MatthieuBizien/roam-to-git/actions)

This script helps you backup your [RoamResearch](https://roamresearch.com/) graphs!

This script automatically
- Downloads a markdown archive of your RoamResearch workspace
- Downloads a json archive of your RoamResearch workspace
- Unzips them to your git directory
- Commits and pushes the difference to Github

# Demo
[See it in action!](https://github.com/MatthieuBizien/roam-to-git-demo). This repo is updated using roam-to-git.

# Why to use it

- You have a backup if RoamResearch loses some of your data.
- You have a history of your notes.
- You can browse your Github repository easily with a mobile device


# Use it with Github Actions (recommended)

**Note**: [Erik Newhard's guide](https://eriknewhard.com/blog/backup-roam-in-github) shows an easy way of setting up Github Actions without using the CLI.

##  Create a (private) Github repository for all your notes

With [gh](https://github.com/cli/cli): `gh repo create notes` (yes, it's private)

Or [manually](https://help.github.com/en/github/getting-started-with-github/create-a-repo)

## Configure Github secrets

- Go to github.com/your/repository/settings/secrets

###

Add 3 (separate) secrets where the names are

`ROAMRESEARCH_USER`

`ROAMRESEARCH_PASSWORD`

`ROAMRESEARCH_DATABASE`

- Refer to [env.template](env.template) for more information

- when inserting the information, there is no need for quotations or assignments

![image](https://user-images.githubusercontent.com/173090/90904133-2cf1c900-e3cf-11ea-960d-71d0543b8158.png)


## Add GitHub action

```

cd notes
mkdir -p .github/workflows/
curl https://raw.githubusercontent.com/MatthieuBizien/roam-to-git-demo/master/.github/workflows/main.yml > \
 .github/workflows/main.yml
git add .github/workflows/main.yml
git commit -m "Add github/workflows/main.yml"
git push --set-upstream origin master

```

To publish your digital garden, go to [Netlify](https://netlify.com/) and create a new site from the repository you've just created.

This will run every hour, fetch the notes referenced by your Public note, and publish them.
```
