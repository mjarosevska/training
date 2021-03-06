# Tutorial 04 - Data models: Build from scratch

TODO Short introduction of what this is about, should be the same/similar to the Indico description. Something like: The goal of this tutorial...

Jump to: [Step 1](#step-1) | [Step 2](#step-2) | [Step 3](#step-3) | [Step 4](#step-4)

Any extra long description

## Step 1

Start from a clean and working instance:

```bash
$ cd 04-data-models-build-from-scratch/
$ ./init.sh
```

## Step 2

Create an authors folder and an authors JSONSchema:

```json
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "id": "http://localhost/schemas/authors/author-v1.0.0.json",
    "additionalProperties": false,
    "title": "My site v1.0.0",
    "type": "object",
    "properties": {
      "id": {
        "description": "Invenio record identifier (integer).",
        "type": "number"
      },
      "name": {
        "description": "Author name.",
        "type": "string"
      },
      "organization": {
        "description": "Organization the author belongs to.",
        "type": "string"
      }
    },
    "required": [
      "id",
      "name"
    ]
  }
```

Next, we will add the Marshmallow schema to `my_site/records/marshmallow/json.py` and add a new loader `my_site/records/loader/__init__.py` in order to validate the incoming data before storing it in the database.

```python
class AuthorMetadataSchemaV1(StrictKeysMixin):
    """Schema for the author metadata."""

    id = PersistentIdentifier()
    name = SanitizedUnicode(required=True)
    organization = SanitizedUnicode(required=True)
```

```diff
from __future__ import absolute_import, print_function

from invenio_records_rest.loaders.marshmallow import json_patch_loader, \
    marshmallow_loader

-from ..marshmallow import MetadataSchemaV1
+from ..marshmallow import MetadataSchemaV1, AuthorMetadataSchemaV1

#: JSON loader using Marshmallow for data validation.
json_v1 = marshmallow_loader(MetadataSchemaV1)
+author_v1 = marshmallow_loader(AuthorMetadataSchemaV1)

__all__ = (
    'json_v1',
+    'author_v1',
)
```

We will now create an ElasticSearch mapping to make the authors searchable:

```json
{
  "mappings": {
    "author-v1.0.0": {
      "date_detection": false,
      "numeric_detection": false,
      "properties": {
        "$schema": {
          "type": "text",
          "index": false
        },
        "id": {
          "type": "keyword"
        },
        "name": {
          "type": "keyword"
        },
        "organization": {
          "type": "keyword"
        },
        "_created": {
          "type": "date"
        },
        "_updated": {
          "type": "date"
        }
      }
    }
  }
}
```

Until now we managed to pave the way to create and search authors. Now we will create a Marshmallow schema in `my_site/records/marshmallow/json.py` for serializing and a new serializer in `my_site/records/serializers/__init__.py`:

```python
class AuthorSchemaV1(StrictKeysMixin):
    """Author schema."""

    metadata = fields.Nested(AuthorMetadataSchemaV1)
    created = fields.Str(dump_only=True)
    updated = fields.Str(dump_only=True)
    id = PersistentIdentifier()
```

```diff
"""Record serializers."""

from __future__ import absolute_import, print_function

from invenio_records_rest.serializers.json import JSONSerializer
from invenio_records_rest.serializers.response import record_responsify, \
    search_responsify

-from ..marshmallow import RecordSchemaV1
+from ..marshmallow import RecordSchemaV1, AuthorSchemaV1

# Serializers
# ===========
#: JSON serializer definition.
json_v1 = JSONSerializer(RecordSchemaV1, replace_refs=True)
+author_v1 = JSONSerializer(AuthorSchemaV1, replace_refs=True)

# Records-REST serializers
# ========================
#: JSON record serializer for individual records.
json_v1_response = record_responsify(json_v1, 'application/json')
#: JSON record serializer for search results.
json_v1_search = search_responsify(json_v1, 'application/json')
+#: JSON author serializer for individual authors.
+author_v1_response = record_responsify(author_v1, 'application/json')
+#: JSON author serializer for search results.
+author_v1_search = search_responsify(author_v1, 'application/json')

__all__ = (
    'json_v1',
    'json_v1_response',
    'json_v1_search',
+    'author_v1_response',
+    'author_v1_search',
)
```


## What did we learn

* TODO
