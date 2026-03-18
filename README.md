# teams-as-code

Declaratively managing GitHub Team membership and permissions of *The Turing Way* community.

## Repository and file layout

Each GitHub Team in _The Turing Way_ GitHub organisation has its own YAML file in the [`teams`](./teams) folder named `<team-name>.yaml`.
The YAML files follow this format:

```yaml
name: new-team
description: "A short description of the team's purpose"  # Optional
privacy: closed  # Optional
parent-team: new-team-parent  # Optional
members:
  - user: username1
    maintainer: true  # This team member will be able to manage the team
  - user: username2
permissions:
  - repo: repo1
    role: push
  - repo: repo2
    role: pull
  - repo: repo3
    role: maintain
  - repo: repo4
    role: triage
```

> [!WARNING]
> Note the following requirements:
>
> - `name` MUST include only lowercase letters, numbers, or hyphens. For example, `my-new-team` will succeed, but `My New Téam` will not.
> - GitHub Teams can be [nested](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams#nested-teams).
>   If you wish your team to be the child of another, provide the name of that team here.
>   It MUST include only lowercase letters, numbers, or hyphens.
> - If defining `privacy`, it MUST take a value from `closed` or `secret`.
> - `role` MUST take a value from: `pull` (equivalent to `read`), `triage`, `push` (equivalent to `write`), `maintain`, or `admin`.

Editing any of these files, or adding a new file in this folder, will trigger a CI workflow to monitor the changes.
Open a Pull Request and the workflows will trigger to validate them and run a plan of the changes.
*The changes will only be applied after the Pull Request is merged.*

## Team members

- **Adding**: To add a new team member to a team, add their GitHub handle to the `members` list in the appropriate team YAML file
- **Removing**: To remove a team member, remove them from the `members` list

## Repos and permissions

- **Adding**: To give a team access to a repo, add an entry to the `permissions` list in the appropriate team YAML file, with the name of the repo and the level of access to grant
- **Removing**: To remove a team's access to a repo, remove the repo from the `permissions` list

## Teams

- **Create a new team**: To create a new team, add a new file under the [`teams`](./teams) folder named `<team-name>.yaml`, following the [format describe above](#repository-and-file-layout).
  When you open a Pull Request, a CI workflow will check the validity of the file.
  The new team will be created when your Pull Request is merged.
- **Deleting a team**: To delete a team, simply remove the file from the `teams` folder

## Self-managing your team

There are two pieces to include that will allow your team to self-manage through
this repository.

Firstly, in your team's file under the [`teams`](./teams) folder, under
`permissions`, ensure you have included this repository with the `push` role.

```yaml
permissions:
  - repo: teams-as-code
    role: push
```

Then in the [`CODEOWNERS`](./CODEOWNERS) file, add your team to the end of the
list, in the format:

```text
teams/<your-team>.yaml @the-turing-way/<your-team>
```

> [!WARNING]
> Note that your team's name will have been changed to lower case, whitespace swapped for hyphens, and any special characters replaced.
> For example: 'My Téam' will become `my-team`.

Open a Pull Request with these changes.
Once merged, this will request your team to review future Pull Requests against your team's file and allow that team's members to merge themselves, providing that tests pass.

## How does this repo work? (More detail.)

This repo uses [opentofu](https://opentofu.org/) to declaratively manage the
state of the GitHub Teams, their membership, and repo permissions in
[_The Turing Way_ GitHub organisation](https://github.com/the-turing-way).
This helps us know exactly who has what permissions where, and easily manage that.

The state of the teams, membership, and permissions we wish to declare are
stored in the YAML files in the [`teams` folder](./teams) file, so that making
changes to these files via Pull Requests are applied by opentofu through the
[`plan-and-apply.yaml`](.github/workflows/plan-and-apply.yaml) workflow.

The state is backed up into an object storage bucket currently hosted in GCP;
however, the state can be flexibly moved between buckets and cloud providers.

There are also some workflows that validate any changes made to the yaml file
and opentofu config to ensure its correctness.
