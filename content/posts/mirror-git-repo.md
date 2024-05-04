+++
title = 'How to mirror your git repo?'
date = 2024-04-28T00:00:00+00:00
draft = false
tags = ['git', 'github', 'gitlab', 'codeberg']
+++

{{< lead >}}
Tutorial on how to mirror your Github repositories to Gitlab and Codeberg.
{{< /lead >}}

## Motivation

I have seen an uptick in Reddit posts around Github accounts being locked out. In particular,
the account holder of one of my favorite Neovim plugins [fzf-lua](https://github.com/ibhagwan/fzf-lua) was randomly
suspended. See [PSA: Fzf-lua pulls cause an error, my GitHub account has been “flagged” for no apparent reason?](https://www.reddit.com/r/neovim/comments/1bqf1w3/psa_fzflua_pulls_cause_an_error_my_github_account/).
Most of the time the accounts are reinstated, but now I realize how easily that could disruptive my workflow and
potentially cause me to lose a lot of my work. Additionally, it is nice to have backups available in the case of Github outages.

## Problem

For my situation, I wanted two solve to problems:

1. Create backups of my work that are easily accessible.
2. Have my public facing projects hosted by an alternative provider.

## Solution

I created a bash script `mirror.sh` that does the following:

- Creates a temporary directory 
- Clones a list of repositories in the temporary directory using the [--mirror](https://git-scm.com/docs/git-clone#Documentation/git-clone.txt-code--mirrorcode) option
- For each of the target destination repositories, push the repository using the [--mirror](https://git-scm.com/docs/git-push#Documentation/git-push.txt---mirror) option
- Delete the temporary directory

The benefit of this is that is it portable. I can run it on my local machine or schedule it to run on a remote machine.

### mirror.sh
```bash
#!/usr/bin/env bash
set -e -o pipefail

git_push_args=("$*") # for example --dry-run

mirror_repos=(
	'git-prompt-string'
	'git-prompt-string-lualine.nvim'
	'kitty-scrollback.nvim'
)

source='git@github.com:mikesmithgh'
targets=('git@codeberg.org:mikesmith' 'git@gitlab.com:mikesmithgl')

mirror_dir="$(mktemp -d)/mirror"
mkdir -p "$mirror_dir"
printf 'temporary mirror directory: %s\n' "$mirror_dir"

for repo_name in "${mirror_repos[@]}"; do
	cd "$mirror_dir" || exit 1
	source_repo="$source/$repo_name.git"
	printf 'cloning %s ...\n' "$source_repo"
	git clone --mirror "$source_repo"
	cd "$repo_name.git" || exit 1

	for target in "${targets[@]}"; do
		target_repo="$target/$repo_name.git"
		git remote set-url --push origin "$target_repo"
		printf 'mirroring %s to: %s\n' "$source_repo" "$target_repo"
		printf 'git push --mirror %s\n' "${git_push_args[@]}"
		if ((${#git_push_args[@]})); then
			git push --mirror
		else
			git push --mirror "${git_push_args[@]}"
		fi
	done

done

rm -rf "$mirror_dir"
```

## Run on a schedule

I chose to set this up on a remote virtual machine and configured a cronjob to run `mirror.sh` every eight hours.

For the cloud hosting provider, I went with [Oracle Cloud Infrastructure](https://www.oracle.com/cloud) because they offer [Always Free Resources](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm).
The VM is running `Ubuntu-22.04` on a `VM.Standard.E2.1.Micro` instance so that it falls into the always free category.

### Setup
- Create an SSH key, see [Generating a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key) for instructions
  - For example,
  ```sh
  ssh-keygen -t ed25519 -C "mirror"
  ```
- Add the `id_ed25519.pub` public key to your [GitLab](https://gitlab.com/) account, see [Add an SSH key to your GitLab account](https://docs.gitlab.com/ee/user/ssh.html#add-an-ssh-key-to-your-gitlab-account)
- Add the `id_ed25519.pub` public key to your [Codeberg](https://codeberg.org/) account, see [Add the SSH key to Codeberg](https://docs.codeberg.org/security/ssh-key/#add-the-ssh-key-to-codeberg)
- Install `keychain` 

  > keychain is a manager for ssh-agent, typically run from ~/.bash_profile. It allows your shells and cron jobs to easily share a single ssh-agent process.

  `keychain` is necessary to allow the cronjob ssh access to the git repositories without the need of reentering the ssh passphrase every time.

  ```sh
  sudo apt update && sudo apt install keychain
  ```

- Setup files containing ssh-agent environment variables to allow passwordless ssh connections

  ```sh
  keychain --noask --eval id_dsa
  ```

  This will generate files under the `~/.keychain` in the form of `$HOSTNAME-$SHELL`. In our case, we will use `$HOSTNAME-sh`.

- Add the cronjob to run `mirror.sh` every 8 hours

  - Run the command `crontab -e` and add the following

    ```bash
    SHELL=/bin/bash

    0 */8 * * * source "$HOME"/.keychain/$(hostname)-sh; /home/ubuntu/gitrepos/dotfiles/bash/scripts/mirror.sh 2>&1 | logger -t 'CRON(mirror)'
    ```

    This will run a cronjob every 8 hours that does the following:
    
      - sources the `keychain` file to use ssh-agent environment variables which allows for passwordless ssh connections
      - runs `mirror.sh` to mirror the git repositories
      - pipes the results of `mirror.sh` into `logger` to save the output as system logs `/var/log/syslog` with a prefix of `CRON(mirror):`

