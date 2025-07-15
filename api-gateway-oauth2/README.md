# API Gateway with OAuth2

This example includes an example that allows you to call the API Gateway using OAuth2 scopes.

## Prerequisites

- **swagger-ui-serve**, run: `npm install -g swagger-ui-cli`

## Deploy the solution

* Copy the `example.env` to `.env`, see: `cp example.env .env`.
* Deploy the solution, see: `make deploy`
* Update the `.env` file with the `CLIENT_ID` and `CLIENT_SECRET` from the console.

## Perform API Call

First, you need to request the `access_token`:

```shell
make read-token
```

Now, you are able to call the API using:

```shell
make products
```

You can now try and post to the API: 

```shell
make post-products
```

Switch the scopes to read/write:

```shell
make read-write-token
```

Perform the API call again:

```shell
make post-products
```

## Lessons Learned

### Referencing Models

API Gateway can refer to other models, but it has a very specific implementation. For example, you cannot directly refer
to the model using: `$ref: "#/components/schemas/Product"`. There are two options

#### Option 1: Use definitions

When you use a definition, the definition needs to be part of the schema that you feed into the model: 

```yaml
type: object
properties:
  timestamp:
    type: string
    description: The timestamp of the context of the API call (Model)
  requestId:
    type: string
    description: The requestId of the context of the API call (Model)
  products:
    type: array
    description: List of available products (Model)
    items:
      $ref: "#/definitions/Product"
  definitions:
    Product:
      type: object
      properties:
        name:
          type: string
        description:
          type: string
```

The side effect of this is that the "Product" definition will be converted into a schema component just for this model.
This will mean that you might end up with duplicate schema definitions. 

#### Option 2: Use canonical references

When using the canonical references, you will have one schema component definition:

```yaml
type: object
properties:
  timestamp:
    type: string
    description: The timestamp of the context of the API call (Model)
  requestId:
    type: string
    description: The requestId of the context of the API call (Model)
  products:
    type: array
    description: List of available products (Model)
    items:
      $ref: !Sub "https://apigateway.amazonaws.com/restapis/${MyApi.RestApiId}/models/${ProductModel}"
```

This will be converted into the correct referencing of models. I have used a reference towards the `ProductModel` so
that the name will always be the same as the model.

> [!IMPORTANT]
> In the `AWS::ApiGateway::DocumentationPart` schema you do need to use the `#/components/schemas/ListProducts` notation.

```json
{
  "tags": ["products"],
  "summary": "List products (Doc)",
  "description": "Returns a list of products (Doc)",
  "operationId": "listProducts",
  "responses": {
    "200": {
      "description": "Successful operation (Doc)",
      "content": {
        "application/json": {
          "schema": { "$ref": "#/components/schemas/ListProducts" }
        }
      }
    }
  }
}
```
