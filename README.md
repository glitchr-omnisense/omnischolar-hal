# omnischolar/hal

HAL, the French open archive, for [omnisense/omnischolar](https://github.com/glitchr-omnisense/omnischolar):
an author's deposits - by idHAL, ORCID or the name they sign - with their full texts in open
access; a deposit by its HAL id, DOI or arXiv id; a search; and on every query a filter by domain:
`shs.droit` is the legal scholarship (doctrine) HAL holds. No key.

```php
$hal = (new HalSourceFactory($http))->create();
$hal->works('Keitaro Nakatani');                                         // the deposits signed so
$hal->works('remi-metivier');                                            // by idHAL
$droit = (new HalSourceFactory($http))->create(['domains' => ['shs.droit']]);
$droit->search(new Query(text: 'responsabilité civile', from: 2024));
```

```yaml
omnischolar:
    sources:
        hal: { factory: hal }
        droit: { factory: hal, options: { domains: [shs.droit] } }   # every query filtered on that domain
```

See [docs/](docs/index.md).

## Usage policy

Declared as data in [`policy.json`](policy.json) (read at https://api.archives-ouvertes.fr/docs on 2026-11-10), and honoured on every request by the
core: a limit reached waits when its window ends within `max_wait` seconds, else the request is not
sent ([Usage policies](https://github.com/glitchr-omnisense/omnischolar/blob/1.x/docs/policies.md)).

- Limits: not published (unknown).
- Licence: CC0-1.0; commercial use: allowed.
- Credit: not required.
- Needs: nothing.

## Responsibility

This package is an independent, unofficial client: it is not affiliated with, endorsed by or
sponsored by the CCSD (HAL) or its supervising bodies (CNRS, Inria, INRAE). It reaches their service
on behalf of whoever runs it, and it does not redistribute their data beyond the short excerpts
recorded as test fixtures. Its MIT licence covers this package's code only and grants no right to
the service or to its data.

Whoever installs and runs this package is responsible for complying with HAL's terms ([CGU of the
HAL
portals](https://www.ccsd.cnrs.fr/wp-content/uploads/2023/01/CCSD_CGU_PortailsHAL_2022_VF_valideeDAJ-1.pdf)),
for obtaining whatever they require (an account, a key, a licence or subscription, an attribution or
a citation, a rate limit to respect), and for any use made of what it returns. The authors publish
it free of charge, make no commercial use of it, and accept no liability for how it is used (see
[LICENSE](LICENSE)).

License: MIT since 2026-10-09; earlier versions remain published under LGPL-3.0-or-later.
