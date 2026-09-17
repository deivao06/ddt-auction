# DDTank Auction House — DD Clássico

A single-file web viewer for the **auction house of DDTank**, reading live from
the **DD Clássico** private server (<https://ddclassico.com/>).

It lists the items currently on sale with name, quantity, time left, seller and
price, with sortable columns, a seller filter, and a panel for naming items. It
is read-only: it never bids, buys or logs in — it only queries the server's
public auction listing endpoint (`s1.ddclassico.com/auctionpagelist.ashx`) from
the browser.

This is an unofficial, fan-made tool. It is not affiliated with DDTank, DD
Clássico, or their operators.

## Running locally

Requires Docker with the Compose plugin.

```sh
git clone <repository-url>
cd ddt-auction
docker compose up
```

Open <http://localhost:8080>.

### Changing the port

The default port is 8080. To change it, copy the example file and edit it:

```sh
cp .env.example .env
```

Or set it inline, without creating any file:

```sh
PORT=9000 docker compose up
```

## Item names

The DD Clássico server never returns item names, only a numeric `TemplateID` —
the DDTank game client is what normally resolves them. That is why the table
starts out full of `Item #7015`.

To fix that, click an item's name in the table and type. The left panel lists the
IDs on the current page that still have no name, so you don't have to hunt for
them.

Names come from two layers:

| Layer | Where it lives | Who sees it |
|---|---|---|
| Instance default | `names.json`, in the project root | every visitor |
| Personal override | the browser's `localStorage` | only you, in that browser |

What you type always lands in the personal layer. To promote your names to the
instance default, click **Exportar** in the panel, replace the file in the project
root and commit it. The `✕` next to a name discards your override and restores the
`names.json` value, if there is one.

**Importar** reads a `names.json` back in — useful to carry your names to another
browser, or to pick up a file someone else exported. It merges rather than
replaces, so a partial file never wipes names you already have, and entries that
already match what you see are skipped instead of being copied into your personal
layer. Whatever it brings in lands in the personal layer too: importing never
writes `names.json` itself, which is a served file, not a stored one.

`names.json` is served from the volume, so editing the file and reloading the page
is enough — no need to restart the container.

### Opening it without Docker

You can open `index.html` straight from disk, but `names.json` **will not load**
over `file://`: Chrome blocks `fetch` on that protocol. The page still works, just
with the names saved in your browser only.

## Publishing it online

The project is fully static and the DD Clássico auction API answers with
`Access-Control-Allow-Origin: *`, so no application server and no proxy are
needed. On GitHub Pages, point it at the `main` branch, folder `/ (root)`.

Note that `nginx.conf` only applies to the Docker setup. Pages serves the whole
published branch, so every committed file is reachable there — keep out of the
repository anything the allowlist was protecting.

## Files

| File | Purpose |
|---|---|
| `index.html` | the entire application: markup, style and script |
| `names.json` | `TemplateID` → name map, the instance default |
| `docker-compose.yml` | the nginx service for self-hosting |
| `nginx.conf` | allowlist of what is served over the web |
| `.env.example` | template for choosing the port |

## Note on language

The user interface is in Brazilian Portuguese, matching the DD Clássico player
base. Code, identifiers, comments and documentation are in English.
