# TARMED Validator API

The TARMED Validator API allows you to validate and access TARMED data. The main purpose of this API is to validate TARMED invoices.

## Documentation

The full documentation is available on SwaggerHub: [TARMED Validator API](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed)

**NOTE**: The final, unlimited API (/v1) will be available soon (Q2/2020). Until release the free preview version (/v0) is available and will be limited by daily usage.

## Configuration

The validation can be configured with several attributes of the `config` object to ignore errors, use inaccurate quantity validations and more:

### Ignore Errors

By ignoring errors it is possible to force the validation to be valid. Add the error codes as integers to the `ignore_errors` array. Ignored errors will be added to the validation response in the `ignored` object.

Example:

```json
{
  "config": {
    "ignore_errors": [ 2051, 2057 ]
  }
}
```

For a full reference of error codes, see below.

### Conditions

Set special conditions for validation. E.g. use inacurrate quantity validations, which can't be validated by the given data because it is not listed on the invoice.

Example:

```json
{
  "config": {
    "conditions": [ "inaccurate_quantity_warnings" ]
  }
}
```

Possible condition options are:

| Option | Description |
| --- | --- |
| `inaccurate_quantity_warnings` | Validate inaccurate quantity rules (pro Aufenthalt, pro Testreihe, pro Schwangerschaft, pro Geburt, pro Eingriff, etc.) which can't be validated by the given data as warnings. |
| `inaccurate_quantity_errors` | Validate inaccurate quantity rules (pro Aufenthalt, pro Testreihe, pro Schwangerschaft, pro Geburt, pro Eingriff, etc.) which can't be validated by the given data as errors. inaccurate_quantity_errors overwrites inaccurate_quantity_warnings if both conditions are given. |

## Validation Requests

The following endpoints are available:

| Methods | Path | Description |
| --- | --- | --- |
| `PUT` | `/validate` | Validates a list of services (TARMED Tarifs) against each other. |
| `PUT` | `/validate-invoices` | Validates multiple invoices with lists of services. <br> With this endpoint it is possible to validate all rules with the full patient history. |


## Validation Response

The response object of a validation is structured as follows:

```json
{
  "validation": "<valid|invalid>",
  "errors": [
    {
      "code": 1000,
      "message": "<human readable message>",
      "index": 0,
      "internal_id": 123,
      "rules": [
        "<human readable TARMED rule>"
      ]
    }
  ],
  "warnings": [
    {
      "code": 1000,
      "message": "<human readable message>",
      "index": 0,
      "internal_id": 123,
      "rules": [
        "<human readable TARMED rule>"
      ]
    }
  ],
  "ignored": {
    "errors": [],
    "warnings": []
  }
```

Description about the `error` object:
* `code`: Error code of the validation. See below for a full list.
* `message`: Human readable message of the validation error.
* `index`: Optional; if the error is in relation to a position, the index of the corresponding service is returned.
* `internal_id`: Optional; if the error is in relation to a position and an `internal_id` was provided in the request, it is returned. This is useful to easily display the error in a UI.
* `rules`: Optional; if the corresponding condition has TARMED rules, these are returned in an array of strings as human readable texts.


### Error & Warning Codes

The following list shows what the TARMED Validator API is capabable of and which errors are beeing detected:

| Code | Description | Special Condition |
| --- | --- | --- |
| `2000` | TARMED version not found | |
| `2001` | TARMED code not found | |
| `2010` | Cumulation invalid: exclusion | |
| `2020` | Cumulation invalid: limited inclusion | |
| `2051` | Missing reference (addition) | |
| `2052` | Referenced parent not listed | |
| `2053` | Referenced parent not allowed | |
| `2057` | Reference error: service type reference (R) | |
| `2070` | Side error: side of service not defined | |
| `2071` | Side error: invalid side given | |
| `3001` | Patient error: invalid gender | |
| `3002` | Patient error: invalid date of birth | |
| `3010` | Patient error: service not allowed for gender | |
| `3011` | Patient error: service not allowed for patient's age | |
| `4002` | Service attribute error: invalid date | |
| `5007` | Quantity error: quantity per session invalid | |
| `5008` | Quantity error: quantity per case invalid | |
| `5009` | Quantity error: quantity per patient invalid | |
| `5010` | Quantity error: quantity per side invalid | |
| `5011` | Quantity error: quantity per inhabitation (Aufenthalt) invalid | (1) |
| `5012` | Quantity error: quantity per test serie (Testreihe) invalid | (1) |
| `5013` | Quantity error: quantity per pregnancy (Schwangerschaft) invalid | (1) |
| `5014` | Quantity error: quantity per birth (Geburt) invalid | (1) |
| `5015` | Quantity error: quantity per radiation phase (Bestrahlungsphase) invalid | (1) |
| `5016` | Quantity error: quantity per transmittal (Einsendung) invalid | (1) |
| `5017` | Quantity error: quantity per autopsy (Autopsie) invalid | (1) |
| `5018` | Quantity error: quantity per expertise (Gutachten) invalid | (1) |
| `5019` | Quantity error: quantity per intervention (Eingriff) invalid | (1) |
| `5021` | Quantity error: quantity per day invalid | |
| `5022` | Quantity error: quantity per week invalid | |
| `5023` | Quantity error: quantity per month invalid | |
| `5026` | Quantity error: quantity per year invalid | |
| `5031` | Quantity error: quantity per radiation phase and volume (Bestrahlungsphase und Bestrahlungsvolumen) invalid | (1) |
| `5040` | Quantity error: quantity per joint region (Gelenkregion) invalid | (1) |
| `5043` | Quantity error: quantity per examination (Untersuchung) invalid | (1) |
| `5108` | Quantity warning: case not set for service with matching rule | |
| `5222` | Quantity error: quantity of service group per week invalid | |
| `5223` | Quantity error: quantity of service group per month invalid | |
| `6000` | Qualification error: service not allowed due to missing qualification | |
| `6004` | Qualification error: qualification not found | |
| `6005` | Qualification error: malformed physician, qualification format | |
| `6100` | Qualification warning: qualification not set. | |
| `6101` | Qualification warning: qualification not set. Service has special cumulation exceptions. | |
| `7000` | Warning: Duplicate entry in session found | |

Special conditions:

(1): inaccurrate validation, see config.conditions for details

## Full Example Validation Example

The following example shows, how to validate TARMED positions of a single invoice:

Example request body `GET /validate`:
```json
{
  "locale": "de",
  "tarmed_version": "01.09",
  "config": {
    "ignore_errors": [ 5007 ],
    "conditions": [ "inaccurate_quantity_warnings" ]
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

## Access TARMED Data

The followig requests can be used to access TARMED data.

### `GET` /services/values/{code}

Get values of a TARMED service by code.

Example request:
```
GET /services/values/00.0010?tarmed_version=01.09
```

Response:
```json
{
    "code": "00.0010",
    "chapter_code": "00.01.01",
    "service_type_code": "H",
    "side": false,
    "sex": null,
    "anesthesia_risk_code": null,
    "law_code": "01",
    "experience_code": "FMH05",
    "visit_content": "N",
    "section_code": "0001",
    "rate_factor_doctor": 10.42,
    "rate_factor_assistance": 0.0,
    "rate_factor_technical": 8.19,
    "assistance_quantity": 0,
    "treatment_min": 5,
    "prepost_min": 0,
    "indication_min": 0,
    "additional_min": 0,
    "room_min": 5,
    "change_min": 0,
    "surcharge_factor_doctor": 1.0,
    "surcharge_factor_technical": 1.0,
    "surcharge_factor_doctor_pract": 0.93,
    "valid_from": "2018-01-01",
    "valid_until": "2999-12-31",
    "change_date": "2017-03-12"
}
```

### `GET` /services/texts/{code}

Get texts like name, medical and technical interpretation of a TARMED service by code.

Example request:
```
GET /services/texts/00.0010?tarmed_version=01.09
```

Response:
```json
{
    "code": "00.0010",
    "locale": "D",
    "title": "Konsultation, erste 5 Min. (Grundkonsultation)",
    "medical_interpretation": "Beinhaltet alle ärztlichen Leistungen, die der Facharzt in seiner Praxis oder der Arzt bei ambulanten Patienten im Spital ohne oder mit einfachen Hilfsmitteln (etwa Inhalt 'Besuchskoffer') am Patienten hinsichtlich der Beschwerden und Erscheinungen erbringt, derentwegen dieser zum Facharzt kommt, bzw. gebracht wird und hinsichtlich der Beschwerden und Erscheinungen, die während der gleichen Behandlungsdauer auftreten.\n\nBeinhaltet Begrüssung, Verabschiedung, nicht besonders tarifierte Besprechungen und Untersuchungen, nicht besonders tarifierte Verrichtungen (z.B.: bestimmte Injektionen, Verbände usw.), Begleitung zu und Übergabe (inkl. Anordnungen) an Hilfspersonal betreffend Administration, technische und kurative Leistungen, Medikamentenabgabe (in Notfallsituation u/o als Starterabgabe), auf Konsultation bezogene unmittelbar vorgängige/anschliessende Akteneinsicht/Akteneinträge.",
    "technical_interpretation": null,
    "valid_from": "2001-01-01",
    "valid_until": "2999-12-31",
    "change_date": "2001-11-08"
}
```

### `GET` /services/texts?q=query

Find TARMED services by code or title. Returns an array of TARMED text objects with name, medical and technical interpretation. Max. 30 items returned.

Example request:
```
GET /services/texts?q=infusion
```

Response:
```json
[
    {
        "code": "00.0750",
        "locale": "D",
        "title": "Injektion/Infusion durch nichtärztliches Personal",
        "medical_interpretation": "Nur verrechenbar, falls die Injektion oder die Infusion nicht im Rahmen einer ärztlichen Beratung erfolgt.\n\nInkl. Infusionswechsel.",
        "technical_interpretation": null,
        "valid_from": "2001-01-01",
        "valid_until": "2999-12-31",
        "change_date": "2001-11-08"
    },
    {
        "code": "00.0930",
        "locale": "D",
        "title": "Wechsel einer Infusion, venös, durch den Facharzt (Bestandteil von 'Allgemeine Grundleistungen')",
        "medical_interpretation": null,
        "technical_interpretation": null,
        "valid_from": "2001-01-01",
        "valid_until": "2999-12-31",
        "change_date": "2001-11-08"
    },
    {
        "code": "00.0940",
        "locale": "D",
        "title": "Wechsel einer Infusion, arteriell, durch den Facharzt (Bestandteil von 'Allgemeine Grundleistungen')",
        "medical_interpretation": null,
        "technical_interpretation": null,
        "valid_from": "2001-01-01",
        "valid_until": "2999-12-31",
        "change_date": "2001-11-08"
    }
]
```

## Roadmap

The following features will be implemented in the future:

- [ ] Special validation conditions by configuration for allowing all general services of LG-18 (Allgemeine Grundleistungen)
- [ ] Special validation condition for allowing all cumulations over service blocks (unlike the general rule defines it)
- [ ] Additional TARMED data requests for: requesting all rules for a single service, requesting service groups/blocks of services, requesting services of a group/block, etc.
- [ ] Validation reporting in french, italian
- [x] Special validation conditions by configuration for inaccurate quantity rules
- [x] Validate other service types, if possible (Pro Memoria, Zusatzleistung)
- [x] Notes for quantity rules which can't be validated technically (e.g. Pro Gutachten)
