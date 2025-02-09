[![FIWARE Banner](https://fiware.github.io/tutorials.CRUD-Operations/img/fiware.png)](https://www.fiware.org/developers)
[![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.CRUD-Operations.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial teaches FIWARE users about CRUD Operations. The tutorial builds on the data created in the previous
[stock management example](https://github.com/FIWARE/tutorials.Entity-Relationships/) and introduces the concept of
[CRUD operations](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete), allowing users to manipulate the data
held within the context.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://fiware.github.io/tutorials.CRUD-Operations/).

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/7764c9cbc3cfe2d5b403)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/FIWARE/tutorials.CRUD-Operations/tree/NGSI-v2)

-   このチュートリアルは[日本語](https://github.com/FIWARE/tutorials.CRUD-Operations/blob/master/README.ja.md)でもご覧い
    ただけます。

## Contents

<details>
<summary><strong>Details</strong></summary>

-   [Data Entities](#data-entities)
    -   [Entities within a stock management system](#entities-within-a-stock-management-system)
-   [Architecture](#architecture)
-   [Prerequisites](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin to be removed](#cygwin)
-   [Start Up](#start-up)
-   [What is CRUD?](#what-is-crud)
    -   [Entity CRUD Operations](#entity-crud-operations)
    -   [Attribute CRUD Operations](#attribute-crud-operations)
    -   [Batch CRUD Operations](#batch-crud-operations)
-   [Example CRUD Operations using FIWARE](#example-crud-operations-using-fiware)
    -   [Create Operations](#create-operations)
        -   [Create a New Data Entity](#create-a-new-data-entity)
        -   [Create a New Attribute](#create-a-new-attribute)
        -   [Batch Create New Data Entities or Attributes](#batch-create-new-data-entities-or-attributes)
        -   [Batch Create/Overwrite New Data Entities](#batch-createoverwrite-new-data-entities)
    -   [Read Operations](#read-operations)
        -   [Read an Attribute from a Data Entity](#read-an-attribute-from-a-data-entity)
        -   [Read a Data Entity (key-value pairs)](#read-a-data-entity-key-value-pairs)
        -   [Read Multiple attributes values from a Data Entity](#read-multiple-attributes-values-from-a-data-entity)
        -   [List all Data Entities (verbose)](#list-all-data-entities-verbose)
        -   [List all Data Entities (key-value pairs)](#list-all-data-entities-key-value-pairs)
        -   [List Data Entity by type](#list-data-entity-by-type)
    -   [Update Operations](#update-operations)
        -   [Overwrite the value of an Attribute value](#overwrite-the-value-of-an-attribute-value)
        -   [Overwrite Multiple Attributes of a Data Entity](#overwrite-multiple-attributes-of-a-data-entity)
        -   [Batch Overwrite Attributes of Multiple Data Entities](#batch-overwrite-attributes-of-multiple-data-entities)
        -   [Batch Create/Overwrite Attributes of Multiple Data Entities](#batch-createoverwrite-attributes-of-multiple-data-entities)
        -   [Batch Replace Entity Data](#batch-replace-entity-data)
    -   [Delete Operations](#delete-operations)
        -   [Data Relationships](#data-relationships)
        -   [Delete a Data Entity](#delete-a-data-entity)
        -   [Delete an Attribute from a Data Entity](#delete-an-attribute-from-a-data-entity)
        -   [Batch Delete Multiple Data Entities](#batch-delete-multiple-data-entities)
        -   [Batch Delete Multiple Attributes from a Data Entity](#batch-delete-multiple-attributes-from-a-data-entity)
        -   [Find existing data relationships](#find-existing-data-relationships)
-   [Next Steps](#next-steps)

</details>

# Data Entities

Within the FIWARE platform, an entity represents the state of a physical or conceptual object which exists in the real world.

## Entities within a stock management system

Within our simple stock management system, we currently have four entity types. The relationships between our entities are defined as shown below:

![](https://github.com/AliIbnIbrahim/tutorials.CRUD-Operations/blob/master/images/entities.png)

-   A **Store** is a real world bricks and mortar building. Stores would have properties such as:
    -   Store name, e.g. "Marche au poisson"
    -   Address, e.g. "Zone industrielle ain sebaa 20590 Casablanca"
    -   Physical location, e.g. _52.5075 N, 13.3903 E_
-   A **Shelf** is a real world object to hold items which we wish to sell. Each shelf would have properties such as:
    -   Shelf name, e.g. "Wall Unit"
    -   Physical location, e.g. _33.6271224 N, -7.511453 E_
    -   Maximum capacity
    -   An association to the store in which the shelf is located
-   A **Product** is defined as something that we sell - it is a conceptual object. Products would have properties such as:
    -   Product name, e.g. "Jus Orange"
    -   Price, e.g. 3.99 Dirhams 
    -   Size, e.g. Small
-   An **Inventory Item** is another conceptual entity, used to associate products, stores, shelves and physical objects. It would have properties such as:
    -   An association to the product being sold
    -   An association to the store in which the product is being sold
    -   An association to the shelf where the product is being displayed
    -   Stock count, i.e. product quantity available in the warehouse
    -   Shelf count, i.e. number of items available on the shelf

As you can see, each of the entities defined above contain some properties which are liable to change. For example, product price could change, stock could be sold and the number of items on the shelves would drop.

# Architecture

This application will only make use of one FIWARE component - the [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/). Using the Orion Context Broker is sufficient for
an application to qualify as _“Powered by FIWARE”_.

Currently, the Orion Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to store the context data it manages. Therefore, the architecture will consist of two components:

-   The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
-   The underlying [MongoDB](https://www.mongodb.com/) database:
    -   Used by the Orion Context Broker to store context information such as data entities, subscriptions and
        registrations

Since the two components interact by means of HTTP requests, they can be containerized and run from exposed ports.

![](https://github.com/AliIbnIbrahim/tutorials.CRUD-Operations/blob/master/images/architecture.png)


The necessary configuration information can be seen in the services section of the associated `docker-compose.yml` file:

```yaml
orion:
    image: fiware/orion:latest
    hostname: orion
    container_name: orion
    depends_on:
        - mongo-db
    networks:
        - default
    expose:
        - "1026"
    ports:
        - "1026:1026"
    command: -dbhost mongo-db -logLevel DEBUG
```

```yaml
mongo-db:
    image: mongo:4.2
    hostname: mongo-db
    container_name: db-mongo
    expose:
        - "27017"
    ports:
        - "27017:27017"
    networks:
        - default
```

Both containers reside on the same network - the Orion Context Broker is listening on port `1026` and MongoDB is listening on the default port `271071`. For the sake of this tutorial, we have also made the two ports available from outside the network so that cUrl or Postman can access them without having to be run from inside the network. The command-line initialization should be self explanatory.

# Prerequisites

## Docker

Each tutorial runs all components using [Docker](https://www.docker.com). **Docker** is a container technology which
allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)

#### Install Docker Engine on Ubuntu
-   from this  [link](https://docs.docker.com/engine/install/ubuntu/#set-up-the-repository)
Install using the repository
Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

1. Set up the repository:  Update the apt package index and install packages to allow apt to use a repository over HTTPS:

```console
 sudo apt-get update
 sudo apt-get install ca-certificates curl gnupg lsb-release
```
2. Add Docker’s official GPG key:

```console
 sudo mkdir -p /etc/apt/keyrings
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
3. Use the following command to set up the repository:

```console
 echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
4. Install Docker Engine
Update the apt package index:

```console
sudo apt-get update
```
 
  Install latest Docker Engine, containerd, and Docker Compose:
  
```console
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

**Docker Compose** is a tool for defining and running multi-container Docker applications. A series of `*.yaml` files
are used configure the required services for the application. This means all container services can be brought up in a
single command. 

You can check your current **Docker** and **Docker Compose** versions using the following commands:

```console
docker version
docker compose version
```

Please ensure that you are using Docker version 20.10 or higher and Docker Compose 1.29 or higher and upgrade if
necessary.

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/) to provide a command-line functionality similar to a Linux distribution on Windows.

NB: Cygwin to be remove and replaced by debian/or ubuntu windows package. 

# Start Up

All services can be initialised from the command-line by running the bash script provided within the repository. Please clone the repository and create the necessary images by running the commands as shown below:

```console
git clone https://github.com/FIWARE/tutorials.CRUD-Operations.git
cd tutorials.CRUD-Operations
git checkout NGSI-v2

./services start
```

<better explain what is in the ./services script>

This command will also import seed data from the previous
[Store Finder tutorial](https://github.com/FIWARE/tutorials.Entity-Relationships) on startup.

> :information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```console
> ./services stop
> ```

# What is CRUD?

**Create**, **Read**, **Update** and **Delete** are the four basic functions of persistent storage. These operations are usually referred to using the acronym **CRUD**. Within a database each of these operations map directly to a series of commands, however their relationship with a RESTful API is slightly more complex.

The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) uses [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) to manipulate the context data. As a RESTful API,
requests to manipulate the data held within the context follow the standard conventions found when mapping HTTP verbs to CRUD operations.

## Entity CRUD Operations

For operations where the `<entity-id>` is not yet known within the context, or is unspecified, the `/v2/entities` endpoint is used.

Once an `<entity-id>` is known within the context, individual data entities can be manipulated using the
`/v2/entities/<entity-id>` endpoint.

It is recommended that entity identifiers should be URNs following the
[NGSI-LD specification](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.04.01_60/gs_cim009v010401p.pdf),
therefore each `id` is a URN which follows a standard format: `urn:ngsi-ld:<entity-type>:<entity-id>`. This helps making every `id` in the context data unique.

| HTTP Verb  |                                               `/v2/entities`                                               |                                              `/v2/entities/<entity-id>`                                              |
| ---------- | :--------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------: |
| **POST**   |                                CREATE a new entity and add to the context.                                 |                                 CREATE or UPDATE an attribute of a specified entity.                                 |
| **GET**    | READ entity data from the context. This will return data from multiple entities. The data can be filtered. | READ entity data from a specified entity. This will return data from a single entity only. The data can be filtered. |
| **PUT**    |                                                    :x:                                                     |                                                         :x:                                                          |
| **PATCH**  |                                                    :x:                                                     |                                                         :x:                                                          |
| **DELETE** |                                                    :x:                                                     |                                          DELETE an entity from the context                                           |

A complete list of entity endpoints can be found in the
[NGSI v2 Swagger Specification](https://fiware.github.io/specifications/OpenAPI/ngsiv2#/Entities)

## Attribute CRUD Operations

To perform CRUD operations on attributes, the `<entity-id>` must be known. Each attribute is effectively a key-value
pair.

There are three endpoints:

-   `/v2/entities/<entity-id>/attrs` is only used for a patch operation to update one or more exisiting attributes.
-   `/v2/entities/<entity-id>/attrs/<attribute>` is used to manipulate an attribute as a whole.
-   `/v2/entities/<entity-id>/attrs/<attribute>/value` is used to read or update the `value` of an attribute, leaving
    the `type` untouched.

| HTTP Verb   |                           `.../attrs`                           |                `.../attrs/<attribute>`                |                              `.../attrs/<attribute>/value`                               |
| ----------- | :-------------------------------------------------------------: | :---------------------------------------------------: | :--------------------------------------------------------------------------------------: |
| **POST**    |                               :x:                               |                          :x:                          |                                           :x:                                            |
| **GET**     |                               :x:                               |                          :x:                          | READ the value of an attribute from a specified entity. This will return a single field. |
| **PUT**     |                               :x:                               |                          :x:                          |              UPDATE the value of single attribute from a specified entity.               |
| **PATCH**   | UPDATE one or more existing attributes from an existing entity. |                          :x:                          |                                           :x:                                            |
| **DELETE**. |                               :x:                               | DELETE an existing attribute from an existing entity. |                                           :x:                                            |

A complete list of attribute endpoints can be found in the
[NGSI v2 Swagger Specification](https://fiware.github.io/specifications/OpenAPI/ngsiv2#/Attributes)

## Batch CRUD Operations

Additionally the Orion Context Broker has a convenience batch operation endpoint `/v2/op/update` to manipulate multiple entities in a single operation.

Batch operations are always triggered by a POST request where the payload is an object with two properties:

-   `actionType` specifies the kind of action to invoke (e.g. `delete`)
-   `entities` is an array of objects holding the list of entities to update, along with the relevant entity data used to perform the operation.

# Example CRUD Operations using FIWARE

The following examples assume that the Orion Context Broker is listening on port 1026 of `localhost`, and the initial seed data has been imported from the previous tutorial.

All examples refer to the **Product** entity as defined in the stock management system. CRUD operations will therefore relate to adding, reading, amending and deleting a product or series of products. This is a typical use case for a store regional manager, for example setting prices and deciding what products can be sold. The actual responses you receive in each case will depend on the state of the context data in your system at the time. If you find that you have already deleted an entity by mistake, you can restore the initial context by reloading the data from the command-line

```console
./import-data
```

## Create Operations

Create Operations map to HTTP POST.

-   The `/v2/entities` endpoint is used for creating new entities
-   The `/v2/entities/<entity>` endpoint is used for adding new attributes

Any newly created entity must have `id` and `type` attributes, other attributes are optional and will depend on the
system being modelled. If additional attributes are present though, each should specify both a `type` and a `value`.

The response will be **204 - No Content** if the operation is successful or **422 - Unprocessable Entity** if the
operation fails.

### Create a New Data Entity

This example adds a new **Product** entity ("Citronade" at 1.99 dirhams) to the context.

#### :one: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/entities' \
  --header 'Content-Type: application/json' \
  --data ' {
      "id":"urn:ngsi-ld:Product:010", "type":"Product",
      "name":{"type":"Text", "value":"Citronade"},
      "size":{"type":"Text", "value": "S"},
      "price":{"type":"Integer", "value": 1.99}
}'
```

New entities can be added by making a POST request to the `/v2/entities` endpoint.

The request will fail if any of the attributes already exist in the context.

#### :two: Request:

You can check to see if the new **Product** can be found in the context by making a GET request

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010?type=Product'|jq
```

### Create a New Attribute

This example adds a new `specialOffer` attribute to the existing **Product** entity with `id=urn:ngsi-ld:Product:001`.

#### :three: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs' \
  --header 'Content-Type: application/json' \
  --data '{
      "specialOffer":{"value": true}
}'
```

New attributes can be added by making a POST request to the `/v2/entities/<entity>/attrs` endpoint.

The payload should consist of a JSON object holding the attribute names and values as shown.

If no `type` is specified a default type (`Boolean`, `Text` , `Number` or `StructuredValue`) will be assigned.

Subsequent requests using the same `id` will update the value of the attribute in the context.

#### :four: Request:

You can check to see if the new **Product** attribute can be found in the context by making a GET request

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001?type=Product'
```

As you can see there is now a boolean `specialOffer` flag attached to the productuct with id n°01 **Product** entity.

### Batch Create New Data Entities or Attributes

This example uses the convenience batch processing endpoint to add two new **Product** entities and one new attribute (`offerPrice`) to the context.

#### :five: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"append_strict",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:011", "type":"Product",
      "name":{"type":"Text", "value":"Boisson au Chocolat"},
      "size":{"type":"Text", "value": "M"},
      "price":{"type":"Integer", "value": 1.99}
    },
    {
      "id":"urn:ngsi-ld:Product:012", "type":"Product",
      "name":{"type":"Text", "value":"Boisson au lait a la vanille"},
      "size":{"type":"Text", "value": "M"},
      "price":{"type":"Integer", "value": 2.90}
    },
    {
      "id":"urn:ngsi-ld:Product:001", "type":"Product",
      "offerPrice":{"type":"Integer", "value": 0.89}
    }
  ]
}'
```

The request will fail if any of the attributes already exist in the context.

Batch processing uses the `/v2/op/update` endpoint with a payload with two attributes

-   `actionType=append_strict` means that the request only succeeds if all entities / attributes are new.
-   The `entities` attribute holds an array of entities we wish to create.

Subsequent requests using the same data with the `actionType=append_strict` batch operation will result in an error
response.

### Batch Create/Overwrite New Data Entities

This example uses the convenience batch processing endpoint to add or amend two **Product** entities and one attribute
(`offerPrice`) to the context.

-   if an entity already exists, the request will update that entity's attributes.
-   if an entity does not exist, a new entity will be created.

#### :six: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"append",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:011", "type":"Product",
      "name":{"type":"Text", "value":"Boisson au Chocolat"},
      "size":{"type":"Text", "value": "M"},
      "price":{"type":"Integer", "value": 1.99}
    },
    {
      "id":"urn:ngsi-ld:Product:012", "type":"Product",
      "name":{"type":"Text", "value":"Boisson au lait a la vanille"},
      "size":{"type":"Text", "value": "M"},
      "price":{"type":"Integer", "value": 2.90}
    }
  ]
}'
```

Batch processing uses the `/v2/op/update` endpoint with a payload with two attributes:

-   `actionType=append` means we will overwrite existing entities if they exist
-   The entities attribute holds an array of entities we wish to create/overwrite.

A subsequent request containing the same data (i.e. same entities and `actionType=append`) won't change the context
state.

## Read Operations

-   The `/v2/entities` endpoint is used for listing entities
-   The `/v2/entities/<entity>` endpoint is used for retrieving the details of a single entity

### Filtering

-   The options parameter (combined with the attrs parameter) can be used to filter the returned fields
-   The q parameter can be used to filter the returned entities

### Read a Data Entity (verbose)

This example reads the full context from an existing **Product** entity with a known `id`.

#### :seven: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010?type=Product'|jq
```

#### Response:

Product `urn:ngsi-ld:Product:010` is "Lemonade" at 99 cents. The response is shown below:

```json
{
  "id": "urn:ngsi-ld:Product:010",
  "type": "Product",
  "name": {
    "type": "Text",
    "value": "Citronade",
    "metadata": {}
  },
  "price": {
    "type": "Integer",
    "value": 1.99,
    "metadata": {}
  },
  "size": {
    "type": "Text",
    "value": "S",
    "metadata": {}
  }
}
```

Context data can be retrieved by making a GET request to the `/v2/entities/<entity>` endpoint.

### Read an Attribute from a Data Entity

This example reads the value of a single attribute (`name`) from an existing **Product** entity with a known `id`.

#### :eight: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs/name/value'|jq
```

#### Response:

Product `urn:ngsi-ld:Product:001` is "Jus Orange" at 1.99 Dirham. The response is shown below:

```json
"Jus Orange"
```

Context data can be retrieved by making a GET request to the `/v2/entities/<entity>/attrs/<attribute>/value` endpoint.

### Read a Data Entity (key-value pairs)

This example reads the key-value pairs of two attributes (`name` and `price`) from the context of existing **Product**
entities with a known `id`.

#### :nine: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001?type=Product&options=keyValues&attrs=name,price'|jq
```

#### Response:

Product `urn:ngsi-ld:Product:001` is "Jus Orange" at 1.99 Dirham. The response is shown below:

```json
{
    "id": "urn:ngsi-ld:Product:001",
    "type": "Product",
    "name": "Jus Orange",
    "price": 1.99
}
```

Combine the `options=keyValues` parameter with the `attrs` parameter to retrieve key-value pairs.

### Read Multiple attributes values from a Data Entity

This example reads the value of two attributes (`name` and `price`) from the context of existing **Product** entities
with a known ID.

#### :one::zero: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001?type=Product&options=values&attrs=name,price'
```

#### Response:

Product `urn:ngsi-ld:Product:001` is "Beer" at 99 cents. The response is shown below:

```json
["Beer", 99]
```

Combine the `options=values` parameter and the `attrs` parameter to return a list of values in an array.

### List all Data Entities (verbose)

This example lists the full context of all **Product** entities.

#### :one::one: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities?type=Product'|jq
```

### Response:

On start-up the context held nine products, three more have been added by create operations so the full context will now
contain twelve products.

```json

[
  {
    "id": "urn:ngsi-ld:Product:010",
    "type": "Product",
    "name": {
      "type": "Text",
      "value": "Citronade",
      "metadata": {}
    },
    "price": {
      "type": "Integer",
      "value": 1.99,
      "metadata": {}
    },
    "size": {
      "type": "Text",
      "value": "S",
      "metadata": {}
    }
  },
  {
    "id": "urn:ngsi-ld:Product:011",
    "type": "Product",
    "name": {
      "type": "Text",
      "value": "Boisson au Chocolat",
      "metadata": {}
    },
    "price": {
      "type": "Integer",
      "value": 1.99,
      "metadata": {}
    },
    "size": {
      "type": "Text",
      "value": "M",
      "metadata": {}
    }
  },
  {
    "id": "urn:ngsi-ld:Product:012",
    "type": "Product",
    "name": {
      "type": "Text",
      "value": "Boisson au lait a la vanille",
      "metadata": {}
    },
    "price": {
      "type": "Integer",
      "value": 2.9,
      "metadata": {}
    },
    "size": {
      "type": "Text",
      "value": "M",
      "metadata": {}
    }
  },
  {
    "id": "urn:ngsi-ld:Product:001",
    "type": "Product",
    "offerPrice": {
      "type": "Integer",
      "value": 0.89,
      "metadata": {}
    }
  }
]
```

### List all Data Entities (key-value pairs)

This example lists the `name` and `price` attributes of all **Product** entities.

#### :one::two: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/?type=Product&options=keyValues&attrs=name,price'|jq
```

#### Response:

On start-up the context held nine products, three more have been added by create operations so the full context will now
contain twelve products.

```json

[
  {
    "id": "urn:ngsi-ld:Product:010",
    "type": "Product",
    "name": "Citronade",
    "price": 1.99
  },
  {
    "id": "urn:ngsi-ld:Product:011",
    "type": "Product",
    "name": "Boisson au Chocolat",
    "price": 1.99
  },
  {
    "id": "urn:ngsi-ld:Product:012",
    "type": "Product",
    "name": "Boisson au lait a la vanille",
    "price": 2.9
  },
  {
    "id": "urn:ngsi-ld:Product:001",
    "type": "Product"
  }
]

```

Full context data for a specified entity type can be retrieved by making a GET request to the `/v2/entities` endpoint and supplying the `type` parameter, combine this with the `options=keyValues` parameter and the `attrs` parameter to retrieve key-values.

### List Data Entity by type

This example lists the `id` and `type` of all **Product** entities.

#### :one::three: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/?type=Product&options=count&attrs=__NONE'|jq
```

#### Response:

On start-up the context held nine products, three more have been added by create operations so the full context will now
contain twelve products.

```json
[
  {
    "id": "urn:ngsi-ld:Product:010",
    "type": "Product"
  },
  {
    "id": "urn:ngsi-ld:Product:011",
    "type": "Product"
  },
  {
    "id": "urn:ngsi-ld:Product:012",
    "type": "Product"
  },
  {
    "id": "urn:ngsi-ld:Product:001",
    "type": "Product"
  }
]
```

Context data for a specified entity type can be retrieved by making a GET request to the `/v2/entities` endpoint and supplying the `type` parameter. Combine this with `options=count` and `attrs=__NONE` to return the `id` attributes of the given `type`.

> **Note:** The NGSIv2 specification specifies that `attrs=` has to be a "comma-separated list of attribute names whose
> data are to be included in the response". `id` and `type` are not allowed to be used as attribute names. If you
> specify a name that does not exist in attributes, such as `__NONE` to the `attrs=` parameter, No attribute will match
> and you will always retrieve only the `id` and `type` of the entity.

## Update Operations

Overwrite operations are mapped to HTTP PUT. HTTP PATCH can be used to update several attributes at once.

-   The `/v2/entities/<entity>/attrs/<attribute>/value` endpoint is used to update an attribute
-   The `/v2/entities/<entity>/attrs` endpoint is used to update multiple attributes

### Overwrite the value of an Attribute value

This example updates the value of the price attribute of the Entity with `id=urn:ngsi-ld:Product:001`

#### :one::four: Request:

```console
curl -iX PUT \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs/price/value' \
  --header 'Content-Type: text/plain' \
  --data 1.89
```

Existing attribute values can be altered by making a PUT request to the `/v2/entities/<entity>/attrs/<attribute>/value`
endpoint.

### Overwrite Multiple Attributes of a Data Entity

This example simultaneously updates the values of both the price and name attributes of the Entity with
`id=urn:ngsi-ld:Product:001`.

#### :one::five: Request:

```console
curl -iX PATCH \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs' \
  --header 'Content-Type: application/json' \
  --data ' {
      "price":{"type":"Integer", "value": 1.89},
      "name": {"type":"Text", "value": "Ale"}
}'
```

### Batch Overwrite Attributes of Multiple Data Entities

This example uses the convenience batch processing endpoint to update existing products.

#### :one::six: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"update",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:001", "type":"Product",
      "price":{"type":"Integer", "value": 1199}
    },
    {
      "id":"urn:ngsi-ld:Product:002", "type":"Product",
      "price":{"type":"Integer", "value": 1199},
      "size": {"type":"Text", "value": "L"}
    }
  ]
}'
```

Batch processing uses the `/v2/op/update` endpoint with a payload with two attributes - `actionType=append` means we
will overwrite existing entities if they exist whereas the `entities` attribute holds an array of entities we wish to
update.

### Batch Create/Overwrite Attributes of Multiple Data Entities

This example uses the convenience batch processing endpoint to update existing products.

#### :one::seven: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"append",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:001", "type":"Product",
      "price":{"type":"Integer", "value": 1199}
    },
    {
      "id":"urn:ngsi-ld:Product:002", "type":"Product",
      "price":{"type":"Integer", "value": 1199},
      "specialOffer": {"type":"Boolean", "value":  true}
    }
  ]
}'
```

Batch processing uses the `/v2/op/update` endpoint with a payload with two attributes - `actionType=append` means we
will overwrite existing entities if they exist whereas the `entities` attribute holds an array of entities we wish to
update.

### Batch Replace Entity Data

This example uses the convenience batch processing endpoint to replace entity data of existing products.

#### :one::eight: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"replace",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:010", "type":"Product",
      "price":{"type":"Integer", "value": 1199}
    }
  ]
}'
```

Batch processing uses the `/v2/op/update` endpoint with a payload with two attributes - `actionType=replace` means we
will overwrite existing entities if they exist whereas the `entities` attribute holds an array of entities whose data we
wish to replace.

## Delete Operations

Delete Operations map to HTTP DELETE.

-   The `/v2/entities/<entity>` endpoint can be used to delete an entity
-   The `/v2/entities/<entity>/attrs/<attribute>` endpoint can be used to delete an attribute

The response will be **204 - No Content** if the operation is successful or **404 - Not Found** if the operation fails.

### Data Relationships

If there are entities within the context which relate to one another, you must be careful when deleting an entity. You
will need to check that no references are left dangling once the entity has been deleted.

Organizing a cascade of deletions is beyond the scope of this tutorial, but it would be possible using a batch delete
request.

### Delete an Entity

This example deletes the entity with `id=urn:ngsi-ld:Product:001` from the context.

#### :one::nine: Request:

```console
curl -iX DELETE \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010'
```

Entities can be deleted by making a DELETE request to the `/v2/entities/<entity>` endpoint.

Subsequent requests using the same `id` will result in an error response since the entity no longer exists in the
context.

### Delete an Attribute from an Entity

This example removes the `specialOffer` attribute from the entity with `id=urn:ngsi-ld:Product:001`.

#### :two::zero: Request:

```console
curl -iX DELETE \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs/specialOffer'
```

Attributes can be deleted by making a DELETE request to the `/v2/entities/<entity>/attrs/<attribute>` endpoint.

If the attribute does not exist in the context, the result will be an error response.

### Batch Delete Multiple Entities

This example uses the convenience batch processing endpoint to delete some **Product** entities.

#### :two::one: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"delete",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:001", "type":"Product"
    },
    {
      "id":"urn:ngsi-ld:Product:002", "type":"Product"
    }
  ]
}'
```

Batch processing uses the `/v2/op/update` endpoint with a payload with two attributes - `actionType=delete` means we
will delete something from the context and the `entities` attribute holds the `id` of the entities we wish to delete.

If an entity does not exist in the context, the result will be an error response.

### Batch Delete Multiple Attributes from an Entity

This example uses the convenience batch processing endpoint to delete some attributes from a **Product** entity.

#### :two::two: Request:

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"delete",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:003", "type":"Product",
      "price":{},
      "name": {}
    }
  ]
}'
```

Batch processing uses the `/v2/op/update` endpoint with a payload with two attributes - `actionType=delete` means we
will delete something from the context and the `entities` attribute holds an array of attributes we wish to delete.

If any attribute does not exist in the context, the result will be an error response.

### Find existing data relationships

This example returns the key of all entities directly associated with the `urn:ngsi-ld:Product:001`.

#### :two::three: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/?q=refProduct==urn:ngsi-ld:Product:001&options=count&attrs=type'
```

#### Response:

```json
[
    {
        "id": "urn:ngsi-ld:InventoryItem:001",
        "type": "InventoryItem"
    }
]
```

-   If this request returns an empty array, the entity has no associates - it can be safely deleted
-   If the response lists a series of **InventoryItem** entities they should be deleted before the associated
    **Product** entity is removed from the context.

Note that we deleted **Product** `urn:ngsi-ld:Product:001` earlier, so what we see above is actually a dangling
reference, i.e. the returned **InventoryItem** references a **Product** that no longer exists.

# Next Steps

Want to learn how to add more complexity to your application by adding advanced features? You can find out by reading
the other [tutorials in this series](https://fiware-tutorials.rtfd.io)

---

## License

[MIT](LICENSE) © 2018-2023 FIWARE Foundation e.V.
