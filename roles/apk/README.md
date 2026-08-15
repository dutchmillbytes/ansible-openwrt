# `flyoverhead.openwrt.apk`

Package management for OpenWrt 24.10+
- install packages
- remove packages
- refresh the package index when, and only when, something needs installing

## Role Variables

| Variable | Descritpion | Status | Type | Example |
| :--- | :--- | :--- | :--- | :--- |
| `apk_packages` | Packages to install | `optional` | `list` | `["acme-acmesh"]` |
| `apk_packages_absent` | Packages to remove | `optional` | `list` | `["luci-app-acme"]` |
| `apk_update_cache` | Run `apk update` before installing | `optional` | `boolean` | `true` |

> Note: both lists are empty by default, so including this role without
> configuring it does nothing.

## Why this role exists

OpenWrt 24.10 replaced `opkg` with `apk`. The `opkg` binary is **not present**
on those releases, so this collection's other roles — which call `opkg` and the
`opkg` Ansible module in their `prepare.yml` — fail outright. Use this role for
package management on 24.10 and later.

## Behaviour

- **Idempotency** comes from `apk info -e <pkgs>`, which prints the subset that
  is installed. Its exit status is 1 when *any* requested package is missing,
  so the status is not usable as a boolean — the role reads stdout. One round
  trip covers the whole list.
- **`apk update` runs only when something is missing.** It costs a few seconds
  and a no-op run should not pay for it.
- **Check mode is real.** Rather than letting Ansible skip the command, the
  role passes `--simulate`, so `--check` reports what would be installed or
  removed instead of showing nothing.
- **Removals are gated on what is present.** `apk del` on an absent package
  prints `ERROR: No such package` but still exits 0, which would otherwise look
  like a successful change on every run.

## Example

```yaml
- hosts: openwrt
  roles:
    - role: apk
      vars:
        apk_packages: ["acme-acmesh", "acme-common"]
        apk_packages_absent: ["wpad-basic"]
```

## Dependencies

- `gekmihesg.openwrt`
