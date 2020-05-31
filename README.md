# TARMED Validator API

The TARMED Validator API allows you to validate and access TARMED data. The main purpose of this API is to validate TARMED invoices.

## API documentation

**IMPORTANT:**

This API is currently under development. A free preview version (path /v0) is available and will be deprecated after releasing the production version /v1).


### /v0 (preview)

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


## Example

The following example shows, how to validate TARMED positions.

Example request body `GET /validate`:
```json
{
  "locale": "de",
  "tarmed_version": "01.09",
  "config": {
    "ignore_errors": [ 5007 ]
  },
  "patient": {
    "dob": "1981-03-30",
    "gender": "m"
  },
  "physician": {
    "qualifications": [
      { "code": "0100" },
      { "code": "0200" }
    ]
  },
  "services": [
    {
      "date": "2019-04-01",
      "code": "00.0010",
      "quantity": 1,
      "session": 1,
      "internal_id": 100
    },
    {
      "date": "2019-04-01",
      "code": "00.0015",
      "ref_code": "00.0010",
      "quantity": 1,
      "session": 1,
      "internal_id": 101
    },
    {
      "date": "2019-04-01",
      "code": "00.0025",
      "ref_code": "00.0010",
      "quantity": 1,
      "session": 1,
      "internal_id": 102
    },
    {
      "date": "2019-04-01",
      "code": "00.0030",
      "ref_code": "00.0010",
      "quantity": 1,
      "session": 1,
      "internal_id": 103
    },
    {
      "date": "2019-04-01",
      "code": "00.2285",
      "quantity": 1,
      "session": 1,
      "internal_id": 104
    },
    {
      "date": "2019-04-01",
      "code": "00.0060",
      "quantity": 1,
      "session": 1,
      "internal_id": 105
    },
    {
      "date": "2019-04-01",
      "code": "00.0060",
      "quantity": 2,
      "session": 1,
      "internal_id": 105
    }
  ]
}
```

Example response body:
```json
{
  "validation": "invalid",
  "errors": [
    {
      "code": 2010,
      "message": "Kumulation ungültig (Exklusion): 00.0010 und 00.0060.",
      "index": 0,
      "internal_id": 100,
      "rules": [
        "Leistung 00.0010 kann nicht kumuliert werden mit Leistung 00.0060"
      ]
    },
    {
      "code": 6000,
      "message": "Gegebene Dignität(en) 0100, 0200 können Position 00.0015 nicht verrechnen. Eingeschränkt auf Dignität(en): 0500, 1100, 3000, 3010, 9900.",
      "index": 1,
      "internal_id": 101
    },
    {
      "code": 2020,
      "message": "Kumulation ungültig (limitierte Inklusion): 00.0015 und 00.0060",
      "index": 1,
      "internal_id": 101,
      "rules": [
        "Leistung 00.0015 kann ausschliesslich kumuliert werden mit Leistungsgruppe LG-03"
      ]
    },
    {
      "code": 2020,
      "message": "Kumulation ungültig (limitierte Inklusion): 00.0015 und 00.0060",
      "index": 1,
      "internal_id": 101,
      "rules": [
        "Leistung 00.0015 kann ausschliesslich kumuliert werden mit Leistungsgruppe LG-03"
      ]
    },
    {
      "code": 3011,
      "message": "Leistung 00.0025 mit Sitzungsdatum 2019-04-01 nicht erlaubt für Patient mit Geburtsdatum 1981-03-30. Alter: 38.",
      "index": 2,
      "internal_id": 102,
      "rules": [
        "über 75 Jahre (-0 Tage) und unter 6 Jahre (+0 Tage)",
        "Sitzungsdatum ungültig vor: 2056-03-30",
        "Sitzungsdatum ungültig nach: 1987-03-30"
      ]
    },
    {
      "code": 2010,
      "message": "Kumulation ungültig (Exklusion): 00.0060 und 00.0010.",
      "index": 5,
      "internal_id": 105,
      "rules": [
        "Leistung 00.0010 kann nicht kumuliert werden mit Leistung 00.0060"
      ]
    },
    {
      "code": 2010,
      "message": "Kumulation ungültig (Exklusion): 00.0060 und 00.0010.",
      "index": 6,
      "internal_id": 105,
      "rules": [
        "Leistung 00.0010 kann nicht kumuliert werden mit Leistung 00.0060"
      ]
    }
  ],
  "warnings": [
    {
      "code": 7000,
      "message": "Duplikat: Leistung 00.0060 mit Sitzung 1 und Datum 2019-04-01 ist mehrmals aufgeführt. Dies kann zu ungenauer Validierung führen.",
      "index": 5,
      "internal_id": 105
    },
    {
      "code": 7000,
      "message": "Duplikat: Leistung 00.0060 mit Sitzung 1 und Datum 2019-04-01 ist mehrmals aufgeführt. Dies kann zu ungenauer Validierung führen.",
      "index": 6,
      "internal_id": 105
    }
  ],
  "ignored": {
    "errors": [
      {
        "code": 5007,
        "message": "Menge 2 für Leistung 00.0060 nicht erlaubt.",
        "index": 6,
        "internal_id": 105,
        "rules": [
          "max. 1 mal pro Sitzung"
        ]
      }
    ],
    "warnings": []
  }
}
```

## Roadmap

The following features will be implemented:

### /v0 -> /v1

- [ ] Validate other service types, if possible (Pro Memoria, Zusatzleistung)
- [ ] Validation reporting in french, italian
- [ ] Notes for quantity rules which can't be validated technically (e.g. Pro Gutachten)
