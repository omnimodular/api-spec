# Omnimodular API schema

## Field description

Required fields:

- `version`
  - Fixed string "v3".

- `kind`
  - One of: `invoice, purchase-order, delivery-receipt, match-report`.

- `site`
  - Customer namespace identifier.

- `id`
  - The identifier must be unique for the same `kind` and `site`.

- `stage`
  - One of: `input, output, final`.

Optional fields:

- `documents`
  - array of `{ id, kind }` specifying what documents this refers to. Only present for `match-report`.
- `headers`
  - array of `{ name, value }` objects.
- `rows`
  - array of Row objects, which have a `fields` consisting of an array of `{ name, value }` objects.
- `items`
  - array of Item objects, which have a `fields` consisting of an array of `{ name, value }` objects.
- `flow`
  - array of Step objects, which have a `fields` consisting of an array of `{ name, value }` objects.
- `deviations`
  - array of Deviation objects, described in a separate document.
- `itempairs`
  - array if Itempair objects, described in a separate document.
- `attachments`
  - array of `{ name, value }` objects. The value field must be base64 encoded.
- `text`
  - base64-encoded string of OCR-parsed data;
  - and is not expected to need a token -> attachment / page logic.
- `labels`
  - array of restricted strings.
- `metrics`
  - array of `{name, value}` objects;
  ```json
  {
    "name": "accuracy",
    "value": 0.85
  }
  ```
- `images`
  - an optional array of `{name, value}` objects (holding files);
  - the width of the images *should* be 900-1000 pixels.  The value
    field must be base64-encoded.
  ```json
  {
    "name": "Attachment X - page Y of Z.png",
    "value": "..."
  }
  ```

## Identifier restrictions

The `site` and `id` fields must conform to the following rules:

- Must start with a letter or number, and can only contain letters, numbers, and the dash (-) character.
- The first and last letters must be alphanumeric.
- All letters must be lowercase.
- The dash (-) character cannot be the first or last character. Consecutive dash characters are not permitted.
- Must be from 1 through 50 characters long.
- The name has been limited to give some headroom for pre- and post fixes, up to 13 characters.

Note that the field itself can be shorter than the minimum of 3 characters so be sure to validate or prefix with a sufficiently long prefix. 
2 characters should be enough, e.g., `d-`.

Reference: https://docs.microsoft.com/en-us/rest/api/storageservices/naming-queues-and-metadata#queue-names

## Name-value pairs

The name-value pairs conform to the following format.

* Values can be null or omitted.

* For metrics: value is a float or a list of floats.

* For other top-level fields, value is a string or a list of strings.
  - This imples that numbers and dates should be converted to strings.
  - Amounts should always be specified with two decimal points, such as '715.30'.
  - Dates should be encoded with ISO-8601.

* The name-value tuple may also contain a field called `meta` which contains additional information, such as prediction certainties.

## Validation

https://regex101.com/

https://extendsclass.com/json-schema-validator.html

Regexp:

```
^(?=.{1,50}$)[a-z0-9](?:-?[a-z0-9]+)*$
```

or simplified if length is enforced separately:

```
^[a-z0-9](?:-?[a-z0-9]+)*$
```

Examples:

```
x
xy
foo
foobar
foo-bar
o-k
0actuallyok
cc80eb8a-35e0-11ed-90a9-38d547aac12a
00000000-35e0-11ed-90a9-000000000000
12345678901234567890123456789012345678901234567890
```

```
bad--bader
-bad
alsobad-
fooBar
123456789012345678901234567890123456789012345678901
000000000011111111112222222222333333333344444444455
```
