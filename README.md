# torpor-playbooks

Ansible playbooks that complement [torpor](https://github.com/bradydibble/torpor)
(a multi-distro test-VM harness). Designed to run from Ascender / AWX against
host inventories that torpor populates.

Content here stays focused on the Ansible side; the torpor CLI and matrix
tooling live in the main torpor repo.

## Playbooks

### `gather_facts_ephemeral.yml`

Same behaviour as the standard `gather_facts.yml` from
[ctrliq/ascender-playbooks](https://github.com/ctrliq/ascender-playbooks),
but with `ignore_unreachable: true` at the play level.

Intended for host groups that are **expected to be off most of the time** —
test VMs, game servers, and similar ephemeral fleets. Hosts that happen to
be awake still contribute their package and service facts; hosts that are
asleep don't mark the job as failed. This keeps the run status honest:
"failed" means "something went wrong," not "half the fleet is doing exactly
what it's supposed to be doing."

## Usage

Point an Ascender / AWX Project at this repo (public HTTPS clone, no
credentials needed), then create a Job Template that references the
desired playbook.
