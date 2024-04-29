+++
title = 'How to mirror your git repo?'
date = 2024-04-28T00:00:00+00:00
draft = false
tags = ['git', 'github', 'gitlab', 'codeberg']
+++

{{< lead >}}
Tutorial on how to mirror your Github repositories to Gitlab and Codeberg.
{{< /lead >}}

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ultrices gravida dictum fusce ut placerat orci. Cursus mattis molestie a iaculis at erat pellentesque adipiscing. Tellus orci ac auctor augue mauris augue neque. Consectetur libero id faucibus nisl tincidunt eget nullam non. Imperdiet nulla malesuada pellentesque elit eget gravida cum. Accumsan tortor posuere ac ut consequat semper viverra nam libero. Arcu cursus vitae congue mauris. Ut faucibus pulvinar elementum integer enim. Dolor sit amet consectetur adipiscing elit pellentesque. Tincidunt praesent semper feugiat nibh sed pulvinar proin gravida. Ut sem viverra aliquet eget. Potenti nullam ac tortor vitae purus. Ac placerat vestibulum lectus mauris ultrices eros in cursus turpis.

Felis imperdiet proin fermentum leo vel orci porta non pulvinar. Malesuada fames ac turpis egestas integer. Tincidunt ornare massa eget egestas purus viverra accumsan in nisl. Fringilla urna porttitor rhoncus dolor purus non enim. Massa placerat duis ultricies lacus sed turpis tincidunt id. Pulvinar pellentesque habitant morbi tristique senectus et netus et. Commodo elit at imperdiet dui. Mauris in aliquam sem fringilla ut morbi tincidunt. Ac odio tempor orci dapibus ultrices in iaculis nunc. Neque sodales ut etiam sit. Consectetur a erat nam at lectus urna duis. Cras sed felis eget velit aliquet sagittis id. Risus viverra adipiscing at in tellus.

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

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ultrices gravida dictum fusce ut placerat orci. Cursus mattis molestie a iaculis at erat pellentesque adipiscing. Tellus orci ac auctor augue mauris augue neque. Consectetur libero id faucibus nisl tincidunt eget nullam non. Imperdiet nulla malesuada pellentesque elit eget gravida cum. Accumsan tortor posuere ac ut consequat semper viverra nam libero. Arcu cursus vitae congue mauris. Ut faucibus pulvinar elementum integer enim. Dolor sit amet consectetur adipiscing elit pellentesque. Tincidunt praesent semper feugiat nibh sed pulvinar proin gravida. Ut sem viverra aliquet eget. Potenti nullam ac tortor vitae purus. Ac placerat vestibulum lectus mauris ultrices eros in cursus turpis.

Felis imperdiet proin fermentum leo vel orci porta non pulvinar. Malesuada fames ac turpis egestas integer. Tincidunt ornare massa eget egestas purus viverra accumsan in nisl. Fringilla urna porttitor rhoncus dolor purus non enim. Massa placerat duis ultricies lacus sed turpis tincidunt id. Pulvinar pellentesque habitant morbi tristique senectus et netus et. Commodo elit at imperdiet dui. Mauris in aliquam sem fringilla ut morbi tincidunt. Ac odio tempor orci dapibus ultrices in iaculis nunc. Neque sodales ut etiam sit. Consectetur a erat nam at lectus urna duis. Cras sed felis eget velit aliquet sagittis id. Risus viverra adipiscing at in tellus.

At quis risus sed vulputate odio. Faucibus a pellentesque sit amet porttitor eget dolor morbi. Facilisis volutpat est velit egestas dui id ornare arcu odio. Orci dapibus ultrices in iaculis nunc sed augue lacus viverra. Cras semper auctor neque vitae. Sed odio morbi quis commodo. Sed vulputate odio ut enim blandit volutpat maecenas volutpat blandit. Auctor elit sed vulputate mi sit amet. Bibendum arcu vitae elementum curabitur vitae. Vel fringilla est ullamcorper eget nulla facilisi etiam dignissim diam. Penatibus et magnis dis parturient. Non enim praesent elementum facilisis leo. Sit amet commodo nulla facilisi.

