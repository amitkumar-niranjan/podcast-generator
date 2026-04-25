# Podcast Generator Using YAML file

Generate a podcast RSS XML file (`podcast.xml`) from a YAML file (`feed.yaml`) and commit the result back to your repository.

## What This Action Does

This action runs a Docker container that:

1. Reads `feed.yaml`.
2. Generates `podcast.xml` using the values in the YAML file.
3. Commits the updated file.
4. Pushes the commit to the current checked-out branch.

The parser logic is implemented in [feed.py](feed.py).

## Required YAML File

The action expects a file named `feed.yaml` at the repository root.

Example `feed.yaml`:

```yaml
title: "My Podcast"
author: "Amit Kumar"
language: "en-us"
image: "cover.jpg"
subtitle: "Tech talks and updates"
format: "episodic"
link: "https://example.com/podcast/"
category: "Technology"
item:
	- title: "Episode 1"
		description: "Introduction episode"
		duration: "00:10:15"
		published: "Fri, 25 Apr 2026 10:00:00 GMT"
		file: "episodes/ep1.mp3"
		length: 12345678
```

Notes:

1. `link` is used as prefix for media and image URLs.
2. Each object inside `item` becomes one RSS `<item>` entry.
3. `length` should be a number (bytes).

## Action Inputs

Defined in [action.yml](action.yml):

1. `EMAIL` (required): Commit email address.
2. `NAME` (required): Committer name used for Git commit author.

## Workflow Example

Create a workflow file such as `.github/workflows/generate-podcast.yml`:

```yaml
name: Generate Podcast Feed

on:
	push:
		paths:
			- "feed.yaml"
	workflow_dispatch:

permissions:
	contents: write

jobs:
	build-feed:
		runs-on: ubuntu-latest
		steps:
			- name: Checkout
				uses: actions/checkout@v4

			- name: Generate and commit podcast feed
				uses: ./
				with:
					EMAIL: ${{ github.actor }}@users.noreply.github.com
					NAME: ${{ github.actor }}
```

## How The Workflow Works

1. You update and push `feed.yaml`.
2. GitHub Actions starts the workflow.
3. `actions/checkout` downloads your repository.
4. This action runs inside Docker using [Dockerfile](Dockerfile).
5. [entrypoint.sh](entrypoint.sh) runs [feed.py](feed.py), commits changes if needed, and pushes to the current branch.
```
name: Generate Podcase Feeds
on: [push]
jobs: 
  build: 
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v3
      - name: Run Feed Generator
        uses: amitkumar-niranjan/podcast-generator@main
```

## Repository Files Used

1. [action.yml](action.yml): Action metadata and inputs.
2. [Dockerfile](Dockerfile): Action runtime image.
3. [entrypoint.sh](entrypoint.sh): Runtime script (generate, commit, push).
4. [feed.py](feed.py): YAML to RSS XML generator.

## Output

After a successful run, your repository contains an updated `podcast.xml` generated from `feed.yaml`.

## Troubleshooting

1. Push fails with 403 (Permission denied):
Use a GitHub account that has write access to the target repository. Clear cached credentials and authenticate again with the correct account.

2. docker command not found:
Install Docker Desktop, then open a new terminal so PATH updates are applied.

3. Docker build fails with docker-credential-desktop not found:
Ensure Docker Desktop is installed correctly and available in PATH. Restart Docker Desktop and your terminal.

4. Docker build fails on pip install with externally-managed-environment (PEP 668):
Prefer distro packages (for example `python3-yaml`) instead of global pip install in Ubuntu-based images.

5. Action skips commit and push when there are no changes:
This is expected behavior. The script exits successfully after logging that no changes were detected.

6. Action pushes to wrong branch:
The script pushes to the current checked-out branch. Make sure your workflow checks out the branch you expect.
