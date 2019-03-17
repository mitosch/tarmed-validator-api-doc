# TARMED Validator API

The TARMED Validator API allows you to validate and access TARMED data. The main purpose of this API is to validate TARMED invoices.

Full endpoint documentation: [link-to-swagger]

## Example

The following example shows, how to validate TARMED positions.

Example request body:
```json
{
  "patient": {
    "dob": "1981-03-30",
    "gender": "m"
  },
  "services": [
    {
      "date": "2019-04-01",
      "nr": "00.0010",
      "quantity": 1,
      "session": 1
    },
    {
      "date": "2019-04-01",
      "nr": "00.0025",
      "ref_nr": "00.0010",
      "quantity": 1,
      "session": 1
    },
    {
      "date": "2019-04-01",
      "nr": "00.0030",
      "ref_nr": "00.0010",
      "quantity": 1,
      "session": 1
    },
    {
      "date": "2019-04-01",
      "nr": "00.2285",
      "quantity": 1,
      "session": 1
    },
    {
      "date": "2019-04-01",
      "nr": "00.0060",
      "quantity": 1,
      "session": 1
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
      "message": "00.0010 <-> 00.0060 failed. Rules: service 00.0060 cannot be used in conjunction with service 00.0010",
      "index": 0
    },
    {
      "code": 3011,
      "message": "00.0025 can not be used for patient with age 38. Rules of 00.0025: >= 75 (-0), <= 6 (+0)",
      "index": 1
    },
    {
      "code": 2010,
      "message": "00.0060 <-> 00.0010 failed. Rules: service 00.0010 cannot be used in conjunction with service 00.0060",
      "index": 4
    }
  ]
}
```

## Todos

- [ ] Change to [GitHub Sync at SwaggerHub](https://app.swaggerhub.com/help/integrations/github-sync)
