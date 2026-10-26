# omnischolar/hal

HAL, the French open archive, for [omnisense/omnischolar](https://github.com/glitchr-omnisense/omnischolar):
the deposits of an author - by idHAL, ORCID or the name as signed - with their full texts in open
access; a deposit by its HAL id, DOI or arXiv id; a search; and, on every query, a filter by
domain - `shs.droit` is the legal scholarship HAL holds. The source calls HAL's Solr API, without
a key:

| Operation | Request |
|---|---|
| `author($id)` | `GET https://api.archives-ouvertes.fr/ref/author/?q=idHal_s:"..."` or `orcidId_s:"https://orcid.org/..."` |
| `works($author)` | `GET .../search/?q=*:*&fq=authIdHal_s:"..."` (or `authORCIDIdExt_s`, `authFullName_s`), Solr `cursorMark` paging |
| `work($id)` | `GET .../search/?q=halId_s:"..."` (or `doiId_s`, `arxivId_s`)`&rows=1` |
| `search($query)` | `GET .../search/?q=text:(...) AND title_t:(...) AND authFullName_t:"..."` |

The base address is `HalSource::BASE_URI` (`https://api.archives-ouvertes.fr/`). There is no
sandbox.

## Installation

```sh
composer require omnischolar/hal
```

Requires PHP 8.2, `omnisense/omnischolar` (1.x) and `symfony/http-client` (6.4, 7 or 8). In a
Symfony application with `OmnischolarBundle`, the factory is registered as soon as the package is
installed; configure it twice for a general and a filtered source (`hal`, and `droit` on
`shs.droit`).

## Credentials

None.

## Options

| Option | Type | Default | Required | |
|---|---|---|---|---|
| `domains` | list of strings | `[]` | no | HAL domain codes every query of the source is filtered on (`shs.droit`, `chim`, `phys.cond`...); `Query::$domains` adds to them |
| `mailto` | string | `null` | no | added to the `User-Agent` |
| `user_agent` | string | `omnischolar/1.x (+https://github.com/glitchr-omnisense/omnischolar)` | no | |
| `base_uri` | string | `https://api.archives-ouvertes.fr/` | no | |

> [!WARNING]
> Known defect: `throttle` is not read by this factory (the source has no interval), where every
> factory is said to take it.

## Capabilities

`capabilities()`: `author`, `works`, `work`, `search`, `abstracts`, `open_access`, `domains`.

| Operation | Supported | Reads |
|---|---|---|
| `author()` | yes | idHAL, ORCID |
| `works()` | yes | idHAL (`authIdHal_s`), ORCID (`authORCIDIdExt_s`: only the deposits where it was linked), or the name as signed (`authFullName_s`, exact) |
| `work()` | yes | HAL id, DOI, arXiv id |
| `search()` | yes | `text` (`text:`, Solr's operators escaped), `title` (`title_t`), `author` (`authFullName_t`) |

## Behaviour

**Filters.** The source's and the query's `domains` → `fq=domainAllCode_s:(...)`; `from`/`to` →
`producedDateY_i:[from TO to]`; `types` → `docType_s:(...)` with HAL's codes for each kind (`ART`;
`OUV`, `DOUV`, `PROCEEDINGS`, `ISSUE`; `COUV`, `NOTICE`, `CREPORT`; `COMM`, `POSTER`, `PRESCONF`;
`UNDEFINED` for preprints; `THESE`, `HDR`, `MEM`, `ETABTHESE`; `REPORT`, `REPACT`, `SYNTHESE`;
`SOFTWARE`; `NOTE` for reviews; `OTHER`, `PATENT`, `LECTURE`, `BLOG`, `TRAD`, `IMG`, `VIDEO`,
`SON`, `MAP`) - a kind HAL has no code for matches nothing; `openAccess` → `openAccess_bool`.
`sort`: `oldest` → `producedDate_tdate asc`, `relevance` → `score desc`, any other (`cited`
included) → `producedDate_tdate desc`, then `docid asc`. `rows` = `limit` 1 to 10 000; the cursor
is Solr's `cursorMark` (`*` first); `Page::$total` is `numFound`.

**An author.** The `ref/author` records of that idHAL or ORCID, the preferred one first: its full
name, given and family names; the other forms as alternative names; the idHAL and every ORCID;
the HAL CV page, IdRef and ResearcherID addresses as `urls`.

**A deposit.** Title and subtitle; type from `docType_s`; contributors (name, given and family
names, idHAL, role: `aut` and `crp` author, `edt` editor, `trl` translator, or HAL's own code - a
`DOUV`'s authors are its editors); `date` (`producedDate`) and year; venue (the journal with its
ISSNs and publisher, the book title, or the conference); publisher (books, chapters, reports);
volume, issue, pages; abstract; language; keywords; `domains` (`domainAllCode_s`); open access;
the HAL page as `url`; the deposited PDF, or the open-access or arXiv link HAL records; licence;
identifiers: HAL, DOI, arXiv, PMID, ISBN. No citation counts, no metrics.

## Example

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Omnischolar\Hal\HalSourceFactory;
use Omnischolar\Model\Scheme;
use Omnischolar\Source\Query;
use Symfony\Component\HttpClient\HttpClient;

$droit = (new HalSourceFactory(HttpClient::create()))->create(['domains' => ['shs.droit']]);

$page = $droit->search(new Query(text: 'responsabilité civile', from: 2025, limit: 3));
echo $page->total, " deposits in shs.droit since 2025, first 3:\n";
foreach ($page as $work) {
    printf("  %s  %-10s %s%s %s\n", $work->date ?? $work->year, $work->type->value, mb_substr($work->fullTitle(), 0, 80), $work->venue ? ' - '.mb_substr($work->venue->name, 0, 40) : '', $work->id(Scheme::HAL));
}
$author = $droit->author('remi-metivier');
echo $author->name, ' ', implode(' ', $author->identifiers->keys()), "\n";
```

```
308 deposits in shs.droit since 2025, first 3:
  2027  book       Responsables mais pas coupables ? La responsabilité des gouvernants en questions hal-05362813
  2026-10-15  article    Le nouvel article 1254 du code civil : entre appel à une meilleure utilisation d - Recueil Dalloz halshs-05798370
  2026-10-13  article    La diagonale des prudences : l'opérateur sportif associatif saisi par le droit c - Jurisport : La revue juridique et économ halshs-05802506
Rémi Métivier idhal:remi-metivier orcid:0000-0001-5612-8327
```

(called live on 2026-10-26 at 14:18 UTC, from the core's harness image; the first deposit is a
forthcoming book, dated 2027 by HAL)

In Symfony:

```yaml
omnischolar:
    sources:
        hal: { factory: hal }
        droit: { factory: hal, options: { domains: [shs.droit] } }
```

## Errors

| Case | Result |
|---|---|
| an unknown author or deposit | `null` |
| `author()` of anything but an idHAL or ORCID, `work()` of anything but a HAL id, DOI or arXiv id | `NotSupportedException` |
| another 4xx (a malformed Solr query) | `ProviderException` |
| a 429 | `RateLimitedException` |
| a 5xx, a timeout | `UnavailableException` |

## Verified, and not

Tested offline: `Tests/HalSourceTest.php` (7 tests), on `MockHttpClient` with answers recorded on
2026-10-04 - two pages of Keitaro Nakatani's deposits (108), a search in `shs.droit` (510 answers
for "responsabilité civile" since 2024), a deposit by DOI, an author by idHAL (remi-metivier).
Called live on 2026-10-04 and on 2026-10-26 (the example above).

Limits of the service: the contributors' ORCIDs are not aligned with their names in the search
answers (a work's contributors carry their idHAL only); `works()` by ORCID finds only the deposits
where the ORCID was linked (24 of Keitaro Nakatani's 108 on 2026-10-04); by name, only the exact
form signed; some deposits are dated in the future (a forthcoming book).

## Responsibility

An independent, unofficial client, not affiliated with the CCSD (HAL) or its supervising bodies.
Whoever runs it complies with HAL's terms ([CGU of the HAL
portals](https://www.ccsd.cnrs.fr/wp-content/uploads/2023/01/CCSD_CGU_PortailsHAL_2022_VF_valideeDAJ-1.pdf)):
see the README's Responsibility section.
