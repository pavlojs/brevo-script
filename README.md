# brevo-script

> Keep Brevo SMTP keys from being deleted after 90 days of inactivity — one dummy email, pure `bash` + `curl`.

[![License](https://img.shields.io/github/license/pavlojs/brevo-script)](LICENSE)
[![lint](https://github.com/pavlojs/brevo-script/actions/workflows/lint.yml/badge.svg)](https://github.com/pavlojs/brevo-script/actions/workflows/lint.yml)

Brevo deletes an SMTP key that has not sent anything for 90 days. If the key is
wired into a side project that only sends mail occasionally, it quietly stops
working. [`brevo-keepalive.sh`](brevo-keepalive.sh) sends a single throwaway
message through the relay, which resets that clock.

## Requirements

`bash` 4+ and `curl`. No package manager, no runtime, no dependencies — `curl`
speaks SMTP itself.

## Quick start

```bash
git clone https://github.com/pavlojs/brevo-script
cd brevo-script
cp .env.example .env   # fill in login, key, from, to
./brevo-keepalive.sh
```

```
relay:  smtp://smtp-relay.brevo.com:587
login:  you@example.com
from:   noreply@yourdomain.tld
to:     you@example.com
keys:   1

sending with key ****a1b2 ... ok

all 1 key(s) used, expiry clock reset
```

Check the message before sending anything:

```bash
./brevo-keepalive.sh --dry-run
```

The recipient can also be passed as an argument, which overrides `MAIL_TO`:

```bash
./brevo-keepalive.sh someone@example.com
```

## Configuration

Read from the environment, or from a `.env` file next to the script. Environment
variables win over `.env`. Credentials come from the Brevo dashboard under
**SMTP & API → SMTP**.

| Variable | Required | Default | Notes |
| --- | --- | --- | --- |
| `BREVO_SMTP_LOGIN` | yes | — | The SMTP login shown next to your keys, usually your account email. |
| `BREVO_SMTP_KEY` | yes | — | The SMTP key. Comma-separate to keep several keys alive in one run. |
| `MAIL_FROM` | yes | — | Sender address. Must be a verified sender or domain in Brevo, otherwise the relay answers `550`. |
| `MAIL_TO` | yes | — | Where the dummy message lands. Your own inbox is fine. |
| `BREVO_SMTP_HOST` | no | `smtp-relay.brevo.com` | |
| `BREVO_SMTP_PORT` | no | `587` | `587`/`2525` use STARTTLS, `465` uses implicit TLS. |
| `MAIL_SUBJECT` | no | `Brevo SMTP keep-alive` | |

Every key needs its own send, so several keys mean several messages:

```bash
BREVO_SMTP_KEY=xsmtpsib-key-one,xsmtpsib-key-two ./brevo-keepalive.sh
```

Exit codes: `0` all keys used, `1` configuration error, `2` at least one send
failed.

## Scheduling

Monthly is plenty for a 90-day window and leaves room for two missed runs.

```cron
0 6 1 * * /home/you/brevo-script/brevo-keepalive.sh >> /var/log/brevo-keepalive.log 2>&1
```

The script resolves its `.env` relative to its own location, so it does not care
about cron's working directory.

A scheduled GitHub Actions workflow deliberately is not offered here: GitHub
disables `schedule:` triggers in a repository after 60 days without commit
activity, which is exactly the "set it and forget it" case this script exists
for. It would go quiet before the keys it protects do.

## Security

The SMTP key is a sending credential — treat it like a password. Details and the
disclosure policy are in [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE)
