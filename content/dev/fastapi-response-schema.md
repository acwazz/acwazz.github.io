+++
title = "fastapi-responseschema"
date = 2024-01-03
description = "Global response wrappers for FastAPI — a lightweight library to enforce a consistent envelope on every API endpoint."

[extra]
github = "https://github.com/acwazz/fastapi-responseschema"
demo = "https://emanueleaddis.it/fastapi-responseschema/"
tags = ["Python", "FastAPI", "Pydantic", "Open Source", "Backend"]
+++


This package extends the [FastAPI](https://fastapi.tiangolo.com/) response model schema allowing you to have a common response wrapper via a `fastapi.routing.APIRoute`.

This library supports Python versions **>=3.8** and FastAPI versions **>=0.89.1**.


### Install the package
```sh
pip install fastapi-responseschema
```

If you are planning to use the pagination integration, you can install the package including [fastapi-pagination](https://github.com/uriyyo/fastapi-pagination)
```sh
pip install fastapi-responseschema[pagination]
```

### Usage

```py
from typing import Generic, TypeVar, Any, Optional, List
from pydantic import BaseModel
from fastapi import FastAPI
from fastapi_responseschema import AbstractResponseSchema, SchemaAPIRoute, wrap_app_responses


# Build your "Response Schema"
class ResponseMetadata(BaseModel):
    error: bool
    message: Optional[str]


T = TypeVar("T")


class ResponseSchema(AbstractResponseSchema[T], Generic[T]):
    data: T
    meta: ResponseMetadata

    @classmethod
    def from_exception(cls, reason, status_code, message: str = "Error", **others):
        return cls(
            data=reason,
            meta=ResponseMetadata(error=status_code >= 400, message=message)
        )

    @classmethod
    def from_api_route(
        cls, content: Any, status_code: int, description: Optional[str] = None, **others
    ):
        return cls(
            data=content,
            meta=ResponseMetadata(error=status_code >= 400, message=description)
        )


# Create an APIRoute
class Route(SchemaAPIRoute):
    response_schema = ResponseSchema

# Integrate in FastAPI app
app = FastAPI()
wrap_app_responses(app, Route)

class Item(BaseModel):
    id: int
    name: str


@app.get("/items", response_model=List[Item], description="This is a route")
def get_operation():
    return [Item(id=1, name="ciao"), Item(id=2, name="hola"), Item(id=3, name="hello")]
```

Te result of `GET /items`:
```http
HTTP/1.1 200 OK
content-length: 131
content-type: application/json

{
    "data": [
        {
            "id": 1,
            "name": "ciao"
        },
        {
            "id": 2,
            "name": "hola"
        },
        {
            "id": 3,
            "name": "hello"
        }
    ],
    "meta": {
        "error": false,
        "message": "This is a route"
    }
}
```


## Docs
You can find detailed info for this package in the [Documentation](https://acwazz.github.io/fastapi-responseschema/).