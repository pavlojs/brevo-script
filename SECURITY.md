# Security policy

## Reporting a vulnerability

Report privately through GitHub: **Security → Report a vulnerability** on
[this repository](https://github.com/pavlojs/brevo-script/security/advisories/new).
Please do not open a public issue for anything exploitable.

Expect a first response within a few days. This is a hobby project maintained by
one person, so there is no formal SLA.

## Handling credentials

A Brevo SMTP key can send mail as your account. It is a password, not an
identifier.

What the script does to keep it contained:

- **Never passed on the command line.** Credentials go to `curl` through a
  config file on stdin, so they do not appear in `ps` output or the shell
  history of a shared machine.
- **Never printed.** Progress output masks the key down to its last four
  characters.
- **`.env` is parsed, not sourced.** A malformed or hostile `.env` cannot
  execute shell code, and `.env` is gitignored so it cannot be committed by
  accident.
- **Addresses are validated.** `MAIL_FROM` and `MAIL_TO` must be bare email
  addresses, which rejects CR/LF and blocks SMTP command or mail header
  injection.
- **The message file is created with `umask 077`** and removed on exit.
- **TLS is mandatory.** `--ssl-reqd` makes `curl` fail rather than fall back to
  a plaintext session.

What is still on you:

- Keep the key in `.env` (locally, mode `600`) or in GitHub Actions secrets —
  never in the repository.
- Rotate the key in the Brevo dashboard if it has ever been logged, pasted, or
  shared.
- Note that anyone with write access to the repository can read Actions secrets
  through a workflow change. Do not add collaborators you would not hand the key
  to directly.
