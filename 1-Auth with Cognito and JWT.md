# Auth with Cognito and JWT

## Construct

```typescript
const auth = new Cognito(stack, "Auth", {
  login: ["email"],
  cdk: {
    userPool: {
      removalPolicy: RemovalPolicy.DESTROY, // Only for development
    },
  }
});

const api = new Api(stack, "Api", {
  authorizers: {
    jwt: {
      type: "user_pool",
      userPool: {
        id: auth.userPoolId,
        clientIds: [auth.userPoolClientId],
      },
    },
  },
  defaults: {
    authorizer: "jwt",
  },
  routes: {
    "GET /private": "src/private.handler", // For private routes
    "GET /public": {                       // For public routes
      handler: "src/public.handler",
      authorizer: undefined,
    },
  },
});

auth.attachPermissionsForAuthUsers(stack, [api]);
```

## Usage

### API Private Routes

```typescript
export const main: APIGatewayProxyHandlerV2WithJWTAuthorizer = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: "Hello from private route!",
      user: event.requestContext.authorizer.jwt.claims,
    }),
  };
};
```