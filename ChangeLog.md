# TARMED Validator API

Current version:

* Production: [1.3.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/1.3.0)
* Staging: [2.3.2](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/2.3.2)

## Changelog

[2.3.2](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/2.3.2)
- [x] Implement ancestry search (parent, children) of LEISTUNG_HIERARCHIE.

[2.3.1](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/2.3.1)
- [x] Bugfix: Fix documentation error where `include` was documented as `extra` intead of `extras`

[2.3.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/2.3.0)
- [x] Add extra key-value pairs in response for `/common/items` for tariff-specific values
- [x] New endponts `/common/categories` to request categories of tariffs (452 migel included)

[2.2.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/2.2.0)
- [x] Validating other tariffs (drug, migel, lab) with `/validate` and `/validate-invoices` endpoint

[2.1.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/2.1.0)
- [x] Add drug (Spezialitätenliste) tariff to endpointis `/common/`
- [x] New endpoint `/common/search-items` for fulltext search capabilities in all tariffs
- [x] Implement pagination for `/common/items`

[2.0.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/2.0.0)
- [x] Add new endpoints /common/ for listing and searching other tariffs (lab = Analysenliste, migel = Mittel und Genständeliste):
  - `/common/items`: List tariff items
  - `/common/tariffs`: List tariff items
- [x] Rename all /browser/ endpoints to /tarmed/

[1.4.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/1.4.0)
- [x] Add filter operators to `/browser/services/` endpoint to filter by equality, starting with, ending with or contains

[1.3.2](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/1.3.2)
- [x] Add french and italian validation messages

[1.3.1](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/1.3.1)
- [x] Add cumulation validation by service groups
- [x] Add filter possibility to services by fields (chapter_code, experience_code, law_code, section_code, service_type_code) and associations (service_groups.code, service_blocks.code)
- [x] Bugfix: Return correct locale for /services/ endpoints

[1.3.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/1.3.0)
- [x] New browser endpoints:
  - `/browser/services`
  - `/browser/services/{code}`
  - `/browser/chapters`
  - `/browser/chapters/{code}`
  - `/browser/service-blocks`
  - `/browser/service-blocks/{code}`
  - `/browser/service-groups`
  - `/browser/service-groups/{code}`

[1.2.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/1.2.0)
- [x] Add posibility to force cumulation validity by chapters.

[1.1.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/1.1.0)
- [x] Add possibility to ignore all cumulation rules by condition `ignore_cumulations`

[1.0.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/1.0.0)
- [x] Performance optimization
- [x] Several optimizations of error descriptions and minor bugfixes

[0.5.1](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.5.1)
- [x] Quantity rules of service-groups are now cumulated together and validated as separate errors (5222, 5223)
- [x] Optimize error descriptions for time range validation (5021, 5022, 5023, 5026, 5222, 5223)

[0.5.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.5.0)
- [x] Add validation for inaccurate quantity rules, like (pro Aufenthalt, pro Testreihe, pro Schwangerschaft, pro Geburt, pro Eingriff, etc.) (5011, 5012, 5013, 5014, 5015, 5016, 5017, 5018, 5019, 5031, 5040, 5043)
- [x] Add possibility to activate inaccurate quantity rules by setting the conditions (inaccurate_quantity_warnings, inaccurate_quantity_warnings)

[0.4.4](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.4.4)
- [x] Add validation for invalid patient gender (3001)
- [x] Optimize error descriptions
- [x] Update documentation (remove old error codes)
- [x] Change validation result for ignored errors: Validation is now valid if all errors are ignored by configuration

[0.4.3](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.4.3)
- [x] Add validation for per patient rules (5009)

[0.4.2](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.4.2)
- [x] Add validation for per side rules (2070, 2071, 5010)

[0.4.1](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.4.1)
- [x] Add validation for per case rules (5008, 5108)

[0.4](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.4)
- [x] Add endpoint /validate-invoices for validating multiple invoices
- [x] Add quantity per year validation
- [x] Reorder quantity warnings for each position. It adds the warning at the position, where the quantity exceeds.
- [x] Add warnings for missing qualifications (6100, 6101)

[0.3](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.3)
- [x] Validate quantity rules of service groups
- [x] Validate reference service [GI-6](https://www.tarmed-browser.ch/de/generelle-interpretationen#gi-6-hauptleistung-zuschlagsleistung)
- [x] Validation report in german
- [x] Validate exceptions for service blocks according to [GI-45](http://www.tarmed-browser.ch/de/generelle-interpretationen#gi-45-leistungsblocke) (e.g. ignore LB-53 cumulations for non "Medizinische Radiologie/Radiodiagnostik")

[0.2](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.2.1)
- [x] Configuration for ignoring errors and warnings
- [x] Warnings for rules, which can't be validated (e.g. duplicate entries)
- [x] Add reference ID for consumers to refer to submitted ID instead of index when errors appear (`internal_id`)
- [x] Validate quantities over multiple sessions

