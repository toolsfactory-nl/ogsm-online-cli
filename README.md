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

```bash
ogsm --url=tenant.ogsm.online login
```

This asks for your email address, password and, when your tenant requires it,
a one-time code. The url is saved to `$XDG_CONFIG_HOME/ogsm-online/settings.json`
and the token to `credentials.json` next to it (mode 0600), so later commands
need neither.

Where nobody can type a password, such as CI or a cloud agent, set both
environment variables instead; nothing is then read from or written to disk:

```bash
export OGSM_URL=tenant.ogsm.online
export OGSM_TOKEN=...
```

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
ogsm logout
```

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
