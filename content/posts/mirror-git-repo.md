+++
title = 'How to mirror your git repo?'
date = 2024-05-05T13:36:47-04:00
draft = false
tags = ['git', 'github', 'gitlab', 'codeberg']
description = 'Tutorial on how to mirror your Github repositories to Gitlab and Codeberg.'
summary = 'Tutorial on how to mirror your Github repositories to Gitlab and Codeberg.'
featuredImage = '/images/mirror-box-drawing.png'
images = ['/images/mirror-box-drawing.png']
+++

## Motivation

I have seen an uptick in Reddit posts around Github accounts being locked out. In particular,
the account holder of one of my favorite Neovim plugins [fzf-lua](https://github.com/ibhagwan/fzf-lua) was randomly
suspended. See [PSA: Fzf-lua pulls cause an error, my GitHub account has been “flagged” for no apparent reason?](https://www.reddit.com/r/neovim/comments/1bqf1w3/psa_fzflua_pulls_cause_an_error_my_github_account/).
Most of the time, the accounts are reinstated. However, I now realize how easily that could disrupt my workflow and
potentially cause me to lose work. Additionally, it is nice to have backups available in the case of Github outages.

## Problem

I wanted to solve two problems:

1. Create backups of my work that are easily accessible.
2. Have my public facing projects hosted by an alternative provider.

## Solution

I created a bash script `mirror.sh` that does the following:

- Creates a temporary directory 
- Clones a list of repositories in the temporary directory using the [--mirror](https://git-scm.com/docs/git-clone#Documentation/git-clone.txt-code--mirrorcode) option
- For each of the target destination repositories, push the repository using the [--mirror](https://git-scm.com/docs/git-push#Documentation/git-push.txt---mirror) option
- Delete the temporary directory

The benefit of this is that it is portable. I can run it on my local machine or schedule it to run on a remote machine.

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

## Mirror on a schedule

I configured a cronjob to run `mirror.sh` every eight hours on a remote virtual machine.
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
- Setup `keychain`
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

  - Add the following to `~/.bashrc` to start `keychain` and provide your passphrase on login

      ```bash
      keychain "$HOME/.ssh/id_ed25519"
      source "$HOME/.keychain/${HOSTNAME}-sh"
      ```

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

## Example

The following is an example of the repository [kitty-scrollback.nvim](https://github.com/mikesmithgh/kitty-scrollback.nvim) being mirrored from Github to Gitlab and Codeberg.

- [kitty-scrollback.nvim](https://github.com/mikesmithgh/kitty-scrollback.nvim) @ 
<svg class='icon' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 496 512'><!-- Font Awesome Free 5.15.4 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free (Icons: CC BY 4.0, Fonts: SIL OFL 1.1, Code: MIT License) --><path d='M165.9 397.4c0 2-2.3 3.6-5.2 3.6-3.3.3-5.6-1.3-5.6-3.6 0-2 2.3-3.6 5.2-3.6 3-.3 5.6 1.3 5.6 3.6zm-31.1-4.5c-.7 2 1.3 4.3 4.3 4.9 2.6 1 5.6 0 6.2-2s-1.3-4.3-4.3-5.2c-2.6-.7-5.5.3-6.2 2.3zm44.2-1.7c-2.9.7-4.9 2.6-4.6 4.9.3 2 2.9 3.3 5.9 2.6 2.9-.7 4.9-2.6 4.6-4.6-.3-1.9-3-3.2-5.9-2.9zM244.8 8C106.1 8 0 113.3 0 252c0 110.9 69.8 205.8 169.5 239.2 12.8 2.3 17.3-5.6 17.3-12.1 0-6.2-.3-40.4-.3-61.4 0 0-70 15-84.7-29.8 0 0-11.4-29.1-27.8-36.6 0 0-22.9-15.7 1.6-15.4 0 0 24.9 2 38.6 25.8 21.9 38.6 58.6 27.5 72.9 20.9 2.3-16 8.8-27.1 16-33.7-55.9-6.2-112.3-14.3-112.3-110.5 0-27.5 7.6-41.3 23.6-58.9-2.6-6.5-11.1-33.3 2.6-67.9 20.9-6.5 69 27 69 27 20-5.6 41.5-8.5 62.8-8.5s42.8 2.9 62.8 8.5c0 0 48.1-33.6 69-27 13.7 34.7 5.2 61.4 2.6 67.9 16 17.7 25.8 31.5 25.8 58.9 0 96.5-58.9 104.2-114.8 110.5 9.2 7.9 17 22.9 17 46.4 0 33.7-.3 75.4-.3 83.6 0 6.5 4.6 14.4 17.3 12.1C428.2 457.8 496 362.9 496 252 496 113.3 383.5 8 244.8 8zM97.2 352.9c-1.3 1-1 3.3.7 5.2 1.6 1.6 3.9 2.3 5.2 1 1.3-1 1-3.3-.7-5.2-1.6-1.6-3.9-2.3-5.2-1zm-10.8-8.1c-.7 1.3.3 2.9 2.3 3.9 1.6 1 3.6.7 4.3-.7.7-1.3-.3-2.9-2.3-3.9-2-.6-3.6-.3-4.3.7zm32.4 35.6c-1.6 1.3-1 4.3 1.3 6.2 2.3 2.3 5.2 2.6 6.5 1 1.3-1.3.7-4.3-1.3-6.2-2.2-2.3-5.2-2.6-6.5-1zm-11.4-14.7c-1.6 1-1.6 3.6 0 5.9 1.6 2.3 4.3 3.3 5.6 2.3 1.6-1.3 1.6-3.9 0-6.2-1.4-2.3-4-3.3-5.6-2z'/></svg>
Github

- [kitty-scrollback.nvim](https://gitlab.com/mikesmithgl/kitty-scrollback.nvim) @
<svg class='icon' xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><!-- Font Awesome Free 5.15.4 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free (Icons: CC BY 4.0, Fonts: SIL OFL 1.1, Code: MIT License) --><path d="M105.2 24.9c-3.1-8.9-15.7-8.9-18.9 0L29.8 199.7h132c-.1 0-56.6-174.8-56.6-174.8zM.9 287.7c-2.6 8 .3 16.9 7.1 22l247.9 184-226.2-294zm160.8-88l94.3 294 94.3-294zm349.4 88l-28.8-88-226.3 294 247.9-184c6.9-5.1 9.7-14 7.2-22zM425.7 24.9c-3.1-8.9-15.7-8.9-18.9 0l-56.6 174.8h132z"/></svg>
Gitlab

- [kitty-scrollback.nvim](https://codeberg.org/mikesmith/kitty-scrollback.nvim) @
<svg class='icon' xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><title>Codeberg</title><path d="M11.955.49A12 12 0 0 0 0 12.49a12 12 0 0 0 1.832 6.373L11.838 5.928a.187.14 0 0 1 .324 0l10.006 12.935A12 12 0 0 0 24 12.49a12 12 0 0 0-12-12 12 12 0 0 0-.045 0zm.375 6.467l4.416 16.553a12 12 0 0 0 5.137-4.213z"/></svg>
Codberg

