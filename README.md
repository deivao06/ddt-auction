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

## Charts

The card holding the table has two tabs. **Tabela** is the listing; **Gráficos**
ranks the top 10 items by volume on the market.

The server offers no aggregate by item — `type` only groups coarse categories and
`name` is a text search — so the ranking is computed in the browser from every
page of the listing. That is ~260 requests and ~7.5 MB of XML (about 10 seconds),
against a server you do not own, so it is deliberate about when it runs:

- never on page load — only when the **Gráficos** tab is opened
- once per session; the same read is reused until **Atualizar**
- six requests in flight at a time, with progress and a cancel button

Each row carries three numbers. The bar is total **units** on sale; beside it are
the number of **listings** and the average price **per unit**. Units and listings
rank differently — an item sold in stacks tops the units ranking on a handful of
listings, while a one-per-listing item can lead on listings and barely register on
units — which is why both are shown rather than one standing in for the other.

The average is weighted by units (total coupons asked ÷ total units), not the mean
of each listing's unit price: a 500-unit lot and a 1-unit lot are not equal
evidence of what a unit costs. `Price` is the current bid, or the opening price
where nobody has bid.

## Following one item

The third tab, **Item**, follows an item's listings instead of the market as a
whole. Type the item's in-game name and the matches come back as a table — one
row per item, with its listings, units and average price — and the bell in the
last column watches it, exactly as it does for an auction in the first tab.

Watched items collect in **Itens vigiados** in the side panel, alongside the
watched auctions but in their own list: a watched auction names one listing and
has a price and a deadline, a watched item counts many and has an average.
Clicking a row reopens the Item tab on that item's search.

Every couple of minutes each watched item reports what changed:

- `Novo anúncio: 250 un por 1.800 cupons, fixo 2.400.`
- `Fulano lançou 1.048 cupons em 10 un.`
- `Saiu da listagem: 85 un a 4.000 cupons no preço fixo — comprado ou removido.`

These land in the same **Notificações** panel as the watched auctions, so they are
clickable and filter the table by the seller.

Watching several items costs less than it looks, because the poll groups them by
search term: three kinds of `Poção` are three items but one round of requests.

### Why the name, and not the item ID

The `name` parameter is a server-side substring match against the item's real
name — the one the server knows but never returns. That is what makes this cheap:
following an item costs the pages that match the term (one for a narrow name,
seven for `Poção`), not the ~260 a whole-market read takes. There is no
TemplateID filter — `name=7015` returns nothing, and `type` only accepts a handful
of category values, returning the entire market for anything else.

The search was verified complete against a full sweep: for every TemplateID it
returned, the search and the sweep agreed on the exact set of AuctionIDs, with
none missing and none extra. Matching ignores case but **not** accents — `pocao`
finds nothing, `poção` finds 333.

Because one term can cover several items — `Poção` matches five TemplateIDs — the
search lists them all and the bell picks. A term matching more than 1,500 listings
is refused rather than paged through.

The name cell is editable here too, so an item can be named from this tab and not
only from the auction table. Naming redraws every surface that prints an item
name at once: both tables, both watch lists, the top 10 and the notification
history.

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
