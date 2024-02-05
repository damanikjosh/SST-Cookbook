# S3 Bucket

## Construct

```typescript
const bucket = new Bucket(stack, "Bucket", {
  bucket: {
    removalPolicy: RemovalPolicy.DESTROY, // Only for development
  },
});

const api = new Api(stack, "Api", {
  defaults: {
    function: {
      bind: [bucket],
    },
  },
});
```

## Usage

### Generate Upload URL

```typescript
export const main: APIGatewayProxyHandlerV2 = async (event) => {
  const command = new PutObjectCommand({
    Bucket: bucket.bucketName,
    Key: "file.txt",
    ACL: "public-read",
  });
  const url = await getSignedUrl(new S3Client({}), command, { expiresIn: 3600 }); // 1 hour
  return {
    statusCode: 200,
    body: JSON.stringify({
      url,
    }),
  };
};
```

### Generate Download URL

```typescript
export const main: APIGatewayProxyHandlerV2 = async (event) => {
  const command = new GetObjectCommand({
    Bucket: bucket.bucketName,
    Key: "file.txt",
  });
  const url = await getSignedUrl(new S3Client({}), command, { expiresIn: 3600 }); // 1 hour
  return {
    statusCode: 200,
    body: JSON.stringify({
      url,
    }),
  };
};
```

### Get Object Metadata

```typescript
export const main: APIGatewayProxyHandlerV2 = async (event) => {
  const command = new HeadObjectCommand({
    Bucket: bucket.bucketName,
    Key: "file.txt",
  });
  const metadata = await send(new S3Client({}), command);
  return {
    statusCode: 200,
    body: JSON.stringify({
      metadata,
    }),
  };
};
```

### List Objects

```typescript
export const main: APIGatewayProxyHandlerV2 = async (event) => {
  const command = new ListObjectsV2Command({
    Bucket: bucket.bucketName,
    Prefix: "folder/",
  });
  const objects = await send(new S3Client({}), command);
  return {
    statusCode: 200,
    body: JSON.stringify({
      objects,
    }),
  };
};
```