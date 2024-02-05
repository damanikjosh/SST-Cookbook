# DynamoDB with ElectroDB

## Construct

```typescript
const db = new Table(stack, "db", {
  fields: {
    pk: "string",
    sk: "string",
    gsi1pk: "string",
    gsi1sk: "string",
    // add more pk/sk pairs as needed
  },
  primaryIndex: {
    partitionKey: "pk",
    sortKey: "sk",
  },
  globalIndexes: {
    gsi1: {
      partitionKey: "gsi1pk",
      sortKey: "gsi1sk",
    },
    // add more global indexes as needed
  },
  cdk: {
    table: {
      removalPolicy: RemovalPolicy.DESTROY, // Only for development
    },
  },
});

const api = new Api(stack, "Api", {
  defaults: {
    function: {
      bind: [db],
    },
  },
});
```

## Usage

### Entity Configuration

```typescript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {Entity, EntityConfiguration, EntityItem} from "electrodb";
import {Table} from "sst/node/table";

const configuration: EntityConfiguration = {
  table: Table.db.tableName, // Change db to the name of the Table construct
  client: new DynamoDBClient({})
}

// Define the entity
export const UserEntity = new Entity(
  {
    model: {
      version: "1",
      entity: "User",
      service: "profile",
    },
    attributes: {
      userId: {
        type: "string",
        required: true,
        readOnly: true,
      },
      name: {
        type: "string",
      },
      phone: {
        type: "string",
      },
      affiliation: {
        type: "string",
      },
      department: {
        type: "string",
      },
    },
    indexes: {
      primary: {
        pk: {
          field: "pk",
          composite: ["userId"],
        },
        sk: {
          field: "sk",
          composite: [],
        },
      },
    },
  },
  configuration
);

export type UserEntityItem = EntityItem<typeof UserEntity>
```

## CRUD Operations

```typescript
import { UserEntity, UserEntityItem } from "./entities";

export const createUser = async (item: UserEntityItem) => {
  return await UserEntity.put(item);
};

export const getUser = async (userId: string) => {
  return await UserEntity.get({ userId });
};

export const updateUser = async (userId: string, item: Partial<UserEntityItem>) => {
  return await UserEntity.update({ userId }, item);
};

export const deleteUser = async (userId: string) => {
  return await UserEntity.delete({ userId });
};
```