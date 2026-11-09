# ogsm-online-cli

`ogsm` is a command line client for the actions and comments of an
[OGSM.online](https://ogsm.online) tenant, built on its v3 API. It is a single
Ruby script that only uses the standard library, so it needs Ruby 3.1 or newer
and nothing else.

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/toolsfactory-nl/ogsm-online-cli/main/bin/ogsm -o /usr/local/bin/ogsm
chmod +x /usr/local/bin/ogsm
```

Or clone the repository and put `bin/ogsm` on your `PATH`.

## Sign in

`ogsm` signs in with a personal API token. Create one in OGSM.online under
**My settings**, in the *Personal API tokens* section (the tenant needs the API
enabled). Pick *Read and write* to create and change actions and comments; a
*Read only* token can only list them. The token is shown once, so copy it
straight away.

```bash
ogsm --url=tenant.ogsm.online login
```

This asks for the token without echoing it. You can also pipe it in, or pass
`--token TOKEN` (which ends up in your shell history). The token is checked
against the tenant first; the url is then saved to
`$XDG_CONFIG_HOME/ogsm-online/settings.json` and the token to
`credentials.json` next to it (mode 0600), so later commands need neither.

In CI or a cloud agent, set both environment variables instead; nothing is
then read from or written to disk:

```bash
export OGSM_URL=tenant.ogsm.online
export OGSM_TOKEN=...
```

When the tenant refuses the token (revoked, expired, or a token from the old
email and password login), `ogsm` stops and says so: create a new personal
token and run `ogsm login` again or update `OGSM_TOKEN`. A change with a read
only token stops with a message that the token is read only.

## Commands

Every command prints JSON.

```bash
ogsm list --canvas-id ID
ogsm statuses
ogsm create --canvas-id ID --title TITLE [--description TEXT | --description-file PATH] [--strategy-id ID] [--due DATE] [--start DATE] [--list STATUS] [--assignee USER_ID] [--labels "name1,name2"]
ogsm update ACTION_ID [--title TITLE] [--description TEXT | --description-file PATH] [--due DATE] [--list STATUS] [--assignee USER_ID] [--labels "name1,name2"]
ogsm comments ACTION_ID
ogsm comment-create ACTION_ID (--content CONTENT | --content-file PATH)
ogsm comment-update ACTION_ID COMMENT_ID (--content CONTENT | --content-file PATH)
ogsm comment-delete ACTION_ID COMMENT_ID
ogsm login [--token TOKEN]
ogsm logout
```

`comment-delete` only removes your own comments; it prints
`{"deleted": COMMENT_ID}`.

`--list` takes a status id, a status name or a base list key (`backlog`,
`todo`, `doing`, `done`); `ogsm statuses` shows the statuses your tenant uses.

## Images

Descriptions and comments are HTML. To embed an image, give an `<img>` a
base64 data URI as its `src`; OGSM.online stores it as an attachment. PNG,
JPEG, GIF, WebP and HEIC are accepted, up to 10 MB each and 10 per request.
An optional `data-filename` attribute names the file.

Such markup quickly outgrows a command line argument, so read it from a file
with `--description-file` or `--content-file`, or from stdin by passing `-`:

```bash
{ printf '<p>The new dashboard:</p><img src="data:image/png;base64,'; base64 < shot.png | tr -d '\n'; printf '">'; } |
  ogsm comment-create 123 --content-file -
```

## License

[ISC](LICENSE)
