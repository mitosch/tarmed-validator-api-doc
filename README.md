# TARMED Validator API

The TARMED Validator API allows you to validate and access TARMED data. The main purpose of this API is to validate TARMED invoices.

## API documentation

**IMPORTANT:**

This API is currently under heavy development. A free preview version (path /v0) is currently available and will be deprecated after releasing the production version /v1).


### /v0 (preview)

* [0.1.0](https://app.swaggerhub.com/apis-docs/Mitosch/tarmed/0.1.0)

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
      "message": "00.0025 with session date 2019-04-01 not allowed for patient with date of birth 1981-03-30. patient age: 38. invalid before: 2056-03-30, invalid after: 1987-03-30. Rules of 00.0025: >= 75y (tol.: -0d), <= 6y (tol.: +0d)",
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

## Roadmap

The following features will be implemented:

### /v0 -> /v1

- [ ] Integrate reference ID for consumers to refer to submitted ID instead of index when errors appear
- [ ] Validate quantities over multiple sessions

## Todos

- [ ] Change to [GitHub Sync at SwaggerHub](https://app.swaggerhub.com/help/integrations/github-sync)
- [ ] Implement access token authentication
- [ ] Implement logging for false positives/negatives
- [ ] Decouple API from TARMED browser website (separate rails instance)
