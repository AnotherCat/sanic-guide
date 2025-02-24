# Versioning

It is standard practice in API building to add versions to your endpoints. This allows you to easily differentiate incompatible endpoints when you try and change your API down the road in a breaking manner.

Adding a version will add a `/v{version}` url prefix to your endpoints.

::: new NEW in v21.3
The version can be a `int`, `float`, or `str`. Acceptable values:

- `1`, `2`, `3`
- `1.1`, `2.25`, `3.0`
- `"1"`, `"v1"`, `"v1.1"`
:::

## Per route

---:1

You can pass a version number to the routes directly.
:--:1
```python
# /v1/text
@app.route("/text", version=1)
def handle_request(request):
    return response.text("Hello world! Version 1")

# /v2/text
@app.route("/text", version=2)
def handle_request(request):
    return response.text("Hello world! Version 2")
```
:---

## Per Blueprint

---:1

You can also pass a version number to the blueprint, which will apply to all routes in that blueprint.
:--:1
```python
bp = Blueprint("test", url_prefix="/foo", version=1)

# /v1/foo/text
@bp.route("/html")
def handle_request(request):
    return response.html("<p>Hello world!</p>")
```
:---

## Per Blueprint Group

---:1
In order to simplify the management of the versioned blueprints, you can provide a version number in the blueprint
group. The same will be inherited to all the blueprint grouped under it if the blueprints don't already override the
same information with a value specified while creating a blueprint instance.

When using blueprint groups for managing the versions, the following order is followed to apply the Version prefix to
the routes being registered.

1. Route Level configuration
2. Blueprint level configuration
3. Blueprint Group level configuration

If we find a more pointed versioning specification, we will pick that over the more generic versioning specification
provided under the Blueprint or Blueprint Group
:--:1
```python
from sanic.blueprints import Blueprint
from sanic.response import json

bp1 = Blueprint(
    name="blueprint-1",
    url_prefix="/bp1",
    version=1.25,
)
bp2 = Blueprint(
    name="blueprint-2",
    url_prefix="/bp2",
)

group = Blueprint.group(
    [bp1, bp2],
    url_prefix="/bp-group",
    version="v2",
)

# GET /v1.25/bp-group/bp1/endpoint-1
@bp1.get("/endpoint-1")
async def handle_endpoint_1_bp1(request):
    return json({"Source": "blueprint-1/endpoint-1"})

# GET /v2/bp-group/bp2/endpoint-2
@bp2.get("/endpoint-1")
async def handle_endpoint_1_bp2(request):
    return json({"Source": "blueprint-2/endpoint-1"})

# GET /v1/bp-group/bp2/endpoint-2
@bp2.get("/endpoint-2", version=1)
async def handle_endpoint_2_bp2(request):
    return json({"Source": "blueprint-2/endpoint-2"})
```
:---
