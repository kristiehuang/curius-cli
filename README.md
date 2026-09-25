# curius-cli

Save links and highlights to [Curius](https://curius.app) from the terminal —
no browser extension required (useful when extensions are blocked by policy).

It talks directly to the same private API the official Curius extension uses,
which was reverse-engineered from the public Firefox add-on.

## Files

- `curius` — the CLI (Python 3, standard library only). This is the real file.
- `~/.local/bin/curius` — a **symlink** to this file, so it's callable on the
  PATH as just `curius`. Edit this file; the command picks up changes instantly.

## Setup (one time)

Put your Curius JWT in `~/.config/curius/token` (or set env `CURIUS_JWT`).

Get the token from https://curius.app while logged in:
DevTools → Application → Local Storage → `https://curius.app` → copy the value
of the key `jwt` (or Cookies → `jwt`). It's valid ~1 year.

Verify: `curius whoami`

## Usage

```
curius save  <url> [--title T] [--snippet S] [--highlight "text"]
curius highlight <url> "the highlighted text" [--title T]
curius whoami
```

`highlight` (and `save --highlight`) saves the link first, then attaches the
highlight — mirroring the extension's own flow.

## API contract (for reference)

Base `https://curius.app`, header `Authorization: Bearer <jwt>`:

| Purpose            | Request |
|--------------------|---------|
| Current user       | `GET  /api/user` |
| Look up saved link | `POST /api/links/url` `{url}` |
| Save a link        | `POST /api/links` `{link:{link,title,snippet}}` |
| Add a highlight    | `POST /api/links/{id}/highlights` `{highlight:{highlightText,position}}` |

Notes: the save body must be *minimal* — sending `doc`/`classify`/`highlight`
as null makes the server reject with "A valid link object must be provided".
Saving an already-saved URL returns `ERR_EXISTS` (handled gracefully).
