# Trabalho realizado nas Semanas #2 e #3


## Identificação


- Improper neutralization of special elements used in an SQL Command - `SQL Injection` ([CWE-89](https://cwe.mitre.org/data/definitions/89.html)).
- Affects [GeoServer](https://geoserver.org) (versions < 2.21.4, < 2.22.2), an open-source server that allows users to share and edit geospatial data, like the position of ships in real time.
- GeoServer includes support for the OGC Filter expression language and the OGC Common Query Language (CQL) as part of the Web Feature Service (WFS) and Web Map Service (WMS) protocols.
- This vulnerability is found in several GeoServer filters and functions, particularly those using the OGC Filter expression language and OGC Common Query Language (CQL).


## Catalogação


- [Steve Ikeoka](https://github.com/sikeoka), a security researcher, reported and fixed these issues.
- [Statement](https://geoserver.org/vulnerability/2023/02/20/ogc-filter-injection.html) on GeoServer Blog on 2023-02-20, [public released](https://github.com/geoserver/geoserver/security/advisories/GHSA-7g5f-wrx8-5ccf) in 2023-02-21.
- CVSS severity 3.1 base score of `9.8 (critical)`; Exploitability Score: 3.9; Impact Score: 5.9.
- There is no bug-bounty program, but GeoServer gives [instructions](https://geoserver.org/issues/) to report a security vulnerability in a responsible way.


## Exploit


- The affected filters/functions include `PropertyIsLike`, `strEndsWith`, `strStartsWith`, `FeatureId`, `jsonArrayContains` and `DWithin`.
- [This repository](https://github.com/murataydemir/CVE-2023-25157-and-CVE-2023-25158) contains a detailed description and replication steps of the SQL Injection vulnerabilities.
- Since there are multiple injection points, `strStartsWith` is taken as an example for analysis [here](https://medium.com/@knownsec404team/geoserver-sql-injection-vulnerability-analysis-cve-2023-25157-413c1f9818c3). This analysis uses the above repository as a reference.
- To sum up, after handling of filter, the final concatenated SQL statement is as follows `SELECT "gid","bin",encode(ST_AsEWKB("the_geom"), 'base64') as "the_geom" FROM "public"."nyc_buildings" WHERE ("bin"::text LIKE 'x') = true and 1=(SELECT CAST ((SELECT version()) AS INTEGER)) -- %') = true`.


## Ataques


- There are no known reports of successful attacks, vulnerabilty [patched](https://github.com/geoserver/geoserver/commit/145a8af798590288d270b240235e89c8f0b62e1d) since 2023-02-13 (by Steve Ikeoka). Patched versions - 2.21.4, 2.22.2, 2.20.7, 2.19.7, 2.18.7.
- Users are advised to `upgrade` to either version 2.21.4, or version 2.22.2. Disable the PostGIS Datastore encode functions or enable prepared statements as a temporary workaround if upgrading is not feasible.
- These vulnerabilities can allow attackers to inject malicious SQL queries, potentially leading to unauthorized access, data manipulation, or full compromise of the GeoServer instance.

