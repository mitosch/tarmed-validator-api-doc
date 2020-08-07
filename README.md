# TARMED Validator API

The TARMED Validator API allows you to validate and access TARMED data. The main purpose of this API is to validate TARMED invoices.

## Table of Contents

1. [Documentation](#documentation)
1. [Configuration](#configuration)
    1. [Ignore Errors](#ignore-errors)
    1. [Conditions](#conditions)
1. [Global Parameters](#global-parameters)
1. [TARMED Validation API](#tarmed-validation-api)
    1. [Validate Single Invoice](#validate-single-invoice)
    1. [Validate Multiple Invoices](#validate-multiple-invoices)
    1. [Error & Warning Codes](#error--warning-codes)
1. [TARMED Browser API](#tarmed-browser-api)
    1. [Chapters](#chapters)
    1. [Services](#services)
    1. [Service Blocks](#service-blocks)
    1. [Service Groups](#service-groups)
1. [Examples](#examples)
    1. [Example Validation](#example-validation)
    1. [List Services with Includes](#list-2-services-including-service-text-and-service-groups)

## Documentation

A detailled documentation of all endpoints is available on SwaggerHub: [TARMED Validator API](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed)

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
    "conditions": {
      "inaccurate_quantity_rules": "warning",
      "ignore_cumulations": true,
      "force_cumulation_validity": {
        "chapters": ["00.01", "00.03"]
      }
    }
  }
}
```

Possible condition options are:

| Condition Option | Description | Type | Default |
| --- | --- | --- | --- |
| `inaccurate_quantity_rules` | Validate inaccurate quantity rules (pro Aufenthalt, pro Testreihe, pro Schwangerschaft, pro Geburt, pro Eingriff, etc.) which can't be validated by the given data:<br>`"warning"`: return validation result as warning<br>`"error"`: return validation result as error<br>`null`: option is not set, quantity rules are ignored. | `string` | `null` |
| `ignore_cumulations` | Ignores all cumulation erros/warnings and returns possible errors/warnings in the `ignored` object. Useful to focus validation of TARMED invoices to quantity, patient and reference rules:<br>`true`: cumulation errors/warnings are returned in the `ignored` object<br>`false`: cumulation errors/warnings are not ignored for validation | `boolean` | `false` |
| `force_cumulation_validity` | Forces all services of the given types to be valid, even if these are defined to not to be included by cumulation exclusion. Quantity rules are still validated. The given types are arrays with strings. Available options:<br>`chapters: ["00.01", "..."]`: Services of these chapters will always be valid. | `object` | `{}` |

## Global Parameters

The following request parameters can be used for any request:

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| locale | query | Returns the validation errors or the TARMED data in the given language:<br>`de`: german (defaut)<br>`fr`: french<br>`it` - italian<br> | No | `string` |
| tarmed_version | query | Selects the TARMED tarif version:<br>`01.09`: (default)<br>`01.08.01_BR`<br>List of all versions: [https://www.tarmed-browser.ch/de/versionen](https://www.tarmed-browser.ch/de/versionen)  | No | `string` |

## TARMED Validation API

### Validate Single Invoice

**`PUT /validate`**

Validate multiple TARMED services in a single requests. Useful for validating single invoices.

**Request Body**

Example:

```json
{
  "config": {
    "ignore_errors": [
      [
        5007,
        6000
      ]
    ],
    "conditions": {
      "inaccurate_quantity_rules": "warning",
      "force_cumulation_validity": {
        "chapters": [
          "00.01",
          "00.03"
        ]
      }
    }
  },
  "patient": {
    "dob": "1981-03-30",
    "gender": "m"
  },
  "physician": {
    "qualifications": [
      {
        "code": "0100"
      }
    ]
  },
  "services": [
    {
      "code": "00.0010",
      "ref_code": null,
      "quantity": 1,
      "session": 1,
      "date": "2020-08-07",
      "internal_id": "1-123"
    },
    {
      "code": "00.0015",
      "ref_code": "00.0010",
      "quantity": 1,
      "session": 1,
      "date": "2020-08-07",
      "internal_id": "1-124"
    },
    {
      "code": "00.0020",
      "ref_code": "00.0010",
      "quantity": 2,
      "session": 1,
      "date": "2020-08-07",
      "internal_id": "1-125"
    }
  ]
}
```

**Response**

The structured response about the submitted invoice looks like:

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

### Validate Multiple Invoices

**`PUT /validate-invoices`**

Validate multiple invoices with multiple TARMED services in a single requests. Useful for validating multiple invoices (as presented to a patient). With this endpoint it is possible to validate all rules with the full patient history.

**Request Body**

Example:

```json
{
  "config": {
    "ignore_errors": [
      [
        5007,
        6000
      ]
    ],
    "conditions": {
      "inaccurate_quantity_rules": "warning",
      "force_cumulation_validity": {
        "chapters": [
          "00.01",
          "00.03"
        ]
      }
    }
  },
  "patient": {
    "dob": "1981-03-30",
    "gender": "m"
  },
  "physician": {
    "qualifications": [
      {
        "code": "0100"
      }
    ]
  },
  "invoices": [
    {
      "internal_id": 100,
      "case_nr": "C-001",
      "services": [
        {
          "code": "00.0010",
          "ref_code": null,
          "quantity": 1,
          "session": 1,
          "date": "2020-08-07",
          "internal_id": "1-123"
        },
        {
          "code": "00.0015",
          "ref_code": "00.0010",
          "quantity": 1,
          "session": 1,
          "date": "2020-08-07",
          "internal_id": "1-124"
        },
        {
          "code": "00.0020",
          "ref_code": "00.0010",
          "quantity": 2,
          "session": 1,
          "date": "2020-08-07",
          "internal_id": "1-125"
        }
      ]
    },
    {
      "internal_id": 201,
      "case_nr": "C-002",
      "services": [
        {
          "code": "00.0010",
          "ref_code": null,
          "quantity": 1,
          "session": 1,
          "date": "2020-09-01",
          "internal_id": "2-123"
        },
        {
          "code": "00.0015",
          "ref_code": "00.0010",
          "quantity": 1,
          "session": 1,
          "date": "2020-09-01",
          "internal_id": "2-124"
        },
        {
          "code": "00.0020",
          "ref_code": "00.0010",
          "quantity": 2,
          "session": 1,
          "date": "2020-09-01",
          "internal_id": "2-125"
        }
      ]
    }
  ]
}
```

**Response**

The response is identical with the request for [validating single invoices](#validate-single-invoice).

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

## TARMED Browser API

### Chapters

#### List TARMED chapters

**`GET /browser/chapters`**

List TARMED chapters. Maximum of 100 chapters are returned. Pagination (limit, offset) and includes (chapter interpretations, services) can be used.

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| filter[level] | query | Show only chapters of specific hierarchical level. `00` is level 0, `00.01` level 1, etc.  | No | string |
| filter[parent] | query | Filter chapters by a specific parent chapter | No | string |
| page[limit] | query | limit number of returned records (1-100) | No | integer |
| page[offset] | query | offset of returned records | No | integer |
| include | query | include associations of chapters: chapter interpretations, services | No | list of strings, comma separated |

#### Get TARMED chapter

**`GET /browser/chapters/{code}`**

Get TARMED chapter. Includes (chapter interpretations, services) can be used.

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| code | path | chapter code | Yes | string  |
| include | query | include associations of chapters: chapter interpretations, services | No | list of strings, comma separated |

### Services

#### List TARMED services

**`GET /browser/services`**

List TARMED services. Maximum of 100 services is returned. Pagination (limit, offset) and includes (service text, service groups, service blocks, chapter, service type, section, law, anesthesia risk) can be used.

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| page[limit] | query | limit number of returned records (1-100) | No | integer |
| page[offset] | query | offset of returned records | No | integer |
| include | query | include associations like service texts, service groups/blocks, chapters, and more | No | list of strings, comma separated |

#### Get TARMED service

**`GET /browser/services/{code}`**

Get TARMED service. Includes (service text, service groups, service blocks, chapter, service type, section, law, anesthesia risk) can be used.

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| code | path | TARMED service code | Yes | string |

### Service Blocks

#### List TARMED service blocks

**`GET /browser/service-blocks`**

List all TARMED service blocks.

#### Get TARMED service block

**`GET /browser/service-blocks/{code}`**

Get TARMED service block.

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| code | path | service block code | Yes | string |

### Service Groups

#### List TARMED service groups

**`GET /browser/service-groups`**

List all TARMED service groups.

#### Get TARMED service group

**`GET /browser/service-groups/{code}`**

Get TARMED service group.

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| code | path | service group code | Yes | string |

## Examples

### Example Validation

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

### List 2 Services including Service Text and Service Groups

List the first 2 services and include the service text and the service groups of the services.

**Request**

**`GET /browser/services?page[limit]=2&include=service_text,service_groups`**

**Response**
```json
[
  {
    "code": "00.0010",
    "additional_min": 0,
    "anesthesia_risk_code": null,
    "assistance_quantity": 0,
    "change_min": 0,
    "chapter_code": "00.01.01",
    "experience_code": "FMH05",
    "indication_min": 0,
    "law_code": "01",
    "prepost_min": 0,
    "rate_factor_assistance": 0,
    "rate_factor_doctor": 10.42,
    "rate_factor_technical": 8.19,
    "room_min": 5,
    "section_code": "0001",
    "service_type_code": "H",
    "sex": null,
    "side": false,
    "surcharge_factor_doctor": 1,
    "surcharge_factor_doctor_pract": 0.93,
    "surcharge_factor_technical": 1,
    "treatment_min": 5,
    "visit_content": "N",
    "valid_from": "2018-01-01",
    "valid_until": "2999-12-31",
    "change_date": "2017-03-12",
    "service_text": {
      "code": "00.0010",
      "title": "Konsultation, erste 5 Min. (Grundkonsultation)",
      "locale": "D",
      "medical_interpretation": "Beinhaltet alle ärztlichen Leistungen, die der Facharzt in seiner Praxis oder der Arzt bei ambulanten Patienten im Spital ohne oder mit einfachen Hilfsmitteln (etwa Inhalt 'Besuchskoffer') am Patienten hinsichtlich der Beschwerden und Erscheinungen erbringt, derentwegen dieser zum Facharzt kommt, bzw. gebracht wird und hinsichtlich der Beschwerden und Erscheinungen, die während der gleichen Behandlungsdauer auftreten.\n\nBeinhaltet Begrüssung, Verabschiedung, nicht besonders tarifierte Besprechungen und Untersuchungen, nicht besonders tarifierte Verrichtungen (z.B.: bestimmte Injektionen, Verbände usw.), Begleitung zu und Übergabe (inkl. Anordnungen) an Hilfspersonal betreffend Administration, technische und kurative Leistungen, Medikamentenabgabe (in Notfallsituation u/o als Starterabgabe), auf Konsultation bezogene unmittelbar vorgängige/anschliessende Akteneinsicht/Akteneinträge.",
      "technical_interpretation": null,
      "valid_from": "2001-01-01",
      "valid_until": "2999-12-31",
      "change_date": "2001-11-08"
    },
    "service_groups": [
      {
        "code": "03",
        "title": "Tarifpositionen bei denen der Zuschlag für hausärztliche Leistungen in der Arztpraxis (00.0015) abgerechnet werden kann.",
        "locale": "D",
        "show": true,
        "text": "",
        "valid_from": "2018-01-01",
        "valid_until": "2999-12-31",
        "change_date": "2017-09-01"
      },
      {
        "code": "18",
        "title": "Allgemeine Grundleistungen",
        "locale": "D",
        "show": true,
        "text": "",
        "valid_from": "2001-01-01",
        "valid_until": "2999-12-31",
        "change_date": "2001-11-08"
      },
      {
        "code": "58",
        "title": "Allgemeine Grundleistungen nicht kumulierbar mit Konsilium",
        "locale": "D",
        "show": true,
        "text": "",
        "valid_from": "2001-01-01",
        "valid_until": "2999-12-31",
        "change_date": "2001-11-08"
      }
    ]
  },
  {
    "code": "00.0015",
    "additional_min": null,
    "anesthesia_risk_code": null,
    "assistance_quantity": 0,
    "change_min": 0,
    "chapter_code": "00.01.01",
    "experience_code": "FMH05",
    "indication_min": 0,
    "law_code": "01",
    "prepost_min": 0,
    "rate_factor_assistance": 0,
    "rate_factor_doctor": 10.88,
    "rate_factor_technical": 0,
    "room_min": 0,
    "section_code": "0001",
    "service_type_code": "Z",
    "sex": null,
    "side": false,
    "surcharge_factor_doctor": 1,
    "surcharge_factor_doctor_pract": 0.93,
    "surcharge_factor_technical": 1,
    "treatment_min": 0,
    "visit_content": "N",
    "valid_from": "2018-01-01",
    "valid_until": "2999-12-31",
    "change_date": "2017-03-12",
    "service_text": {
      "code": "00.0015",
      "title": "+ Zuschlag für hausärztliche Leistungen in der Arztpraxis",
      "locale": "D",
      "medical_interpretation": "Darf nur im Zusammenhang mit der Erbringung von hausärztlichen Leistungen abgerechnet werden und wenn dem Patienten am selben Tag keine spezialärztlichen Leistungen durch den gleichen Leistungserbringer verrechnet werden.\n\nDarf nicht von ambulanten Diensten von Spitälern abgerechnet werden.",
      "technical_interpretation": null,
      "valid_from": "2014-10-01",
      "valid_until": "2999-12-31",
      "change_date": "2014-07-01"
    },
    "service_groups": [
      {
        "code": "03",
        "title": "Tarifpositionen bei denen der Zuschlag für hausärztliche Leistungen in der Arztpraxis (00.0015) abgerechnet werden kann.",
        "locale": "D",
        "show": true,
        "text": "",
        "valid_from": "2018-01-01",
        "valid_until": "2999-12-31",
        "change_date": "2017-09-01"
      }
    ]
  }
]
```
