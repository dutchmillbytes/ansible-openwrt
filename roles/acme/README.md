# `flyoverhead.openwrt.acme`

OpenWRT `acme` configuration
- configure acme settings
- configure certificate sections

## Role Variables

| Variable | Descritpion | Status | Type | Example |
| :--- | :--- | :--- | :--- | :--- |
| `acme` | ACME client settings | | `dictionary` | |
| &emsp;`account_email` | Account e-mail registered with the CA | `required` | `string` | `admin@example.org` |
| &emsp;`debug` | Enable verbose logging | `optional` | `boolean` | `0` |
| `acme_certs` | Certificates to issue and renew | | `list` | |
| &emsp;`id` | UCI section name | `required` | `string` | `device` |
| &emsp;`state` | Section state | `optional` | `string` | `present` |
| &emsp;`enabled` | Issue and renew this certificate | `required` | `boolean` | `1` |
| &emsp;`domains` | Domains to request; the first is the main domain | `required` | `list` | `["host.example.org"]` |
| &emsp;`validation_method` | `webroot`, `standalone` or `dns` | `required` | `string` | `webroot` |
| &emsp;`webroot` | Directory served on port 80, for `webroot` validation | `optional` | `string` | `/www` |
| &emsp;`standalone` | Bind port 80 directly instead of using a webroot | `optional` | `boolean` | `0` |
| &emsp;`listen_port` | Port for `standalone` validation | `optional` | `integer` | `80` |
| &emsp;`key_type` | Key type | `optional` | `string` | `ec256` |
| &emsp;`keylength` | Key length (deprecated in favour of `key_type`) | `optional` | `string` | `2048` |
| &emsp;`days` | Renew when fewer than this many days remain | `optional` | `integer` | `10` |
| &emsp;`acme_server` | ACME directory URL | `optional` | `string` | `https://ca.example.org/acme/provisioner/directory` |
| &emsp;`staging` | Use the CA's staging environment | `optional` | `boolean` | `0` |
| &emsp;`use_staging` | Deprecated alias of `staging` | `optional` | `boolean` | `0` |
| &emsp;`cert_profile` | Certificate profile requested from the CA | `optional` | `string` | |
| &emsp;`calias` | Challenge alias domain | `optional` | `string` | `example.com` |
| &emsp;`dalias` | Domain alias | `optional` | `string` | `alias.example.com` |
| &emsp;`dns` | acme.sh DNS API, for `dns` validation | `optional` | `string` | `dns_cf` |
| &emsp;`dns_wait` | Seconds to wait for DNS propagation | `optional` | `integer` | `30` |
| &emsp;`credentials` | Environment assignments for the DNS API | `optional` | `list` | `['CF_Token="..."']` |

> Note: `domains` and `credentials` are UCI **lists**. Pass them as lists — do
> not join them into a string. `credentials` is read with
> `config_list_foreach`, which ignores an `option` outright.

## Notes

- Certificates are written to **`/etc/ssl/acme/`**. The `state_dir` option is
  deprecated and is not exposed by this role.
- There is **no `update_uhttpd` option** in this package, despite what some
  documentation suggests. Pointing uhttpd (or any other service) at the issued
  files is a separate step, as is reloading it after renewal — see
  `/usr/lib/acme/hook/`.
- Renewal scheduling is handled by the package itself: enabling the service
  appends `0 0 * * * /etc/init.d/acme renew` to `/etc/crontabs/root`.
- The `Reload acme` handler **restarts the service, which triggers issuance**.
  A configuration change therefore results in a conversation with the CA.
- Install the client with the `apk` role (OpenWrt 24.10+); `acme-acmesh`
  provides the virtual name `acme`.

## Dependencies

- `gekmihesg.openwrt`
