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

### `linux-weekly-update.yml`

Package-manager-abstracted weekly update cycle for a multi-distro fleet.
Targets the `linux_test_vms` group (override with `--limit`) and:

1. Starts every VM in the group via `qm start` on the PVE host.
2. Runs the distro's native package manager (`dnf` / `apt` / `zypper`) to
   update all packages.
3. Reboots if a new kernel landed (RedHat: `needs-restarting -r`, Debian:
   `/run/reboot-required`, Suse: `zypper ps -s`).
4. Emits a per-host JSON report.
5. Shuts everything back down.

Safe to run repeatedly. Schedule on Ascender / AWX / a cron.

### `inventory/example.yml`

Starter Ansible inventory showing the group structure torpor expects
(`linux_test_vms` parent, per-vendor child groups). Copy and populate with
your real hosts.

## Usage

Point an Ascender / AWX Project at this repo (public HTTPS clone, no
credentials needed), then create a Job Template that references the
desired playbook.

Or run directly with `ansible-playbook`:

```sh
ansible-playbook ~/torpor-playbooks/linux-weekly-update.yml \
  -i /path/to/your/inventory.yml --limit rocky9 --check
```
