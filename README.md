# gRPC Service Account JWT

This project demonstrates how to create a gRPC service that authenticates with a Google Cloud service account using a JWT.

## Prerequisites

- A Google Cloud project
- A service account with the `roles/iam.serviceAccounts.actAs` role
- A JWT library for your programming language

## Setup

1. Create a service account in your Google Cloud project.
2. Grant the service account the `roles/iam.serviceAccounts.actAs` role.
3. Download the service account's private key as a JSON file.
4. Create a new gRPC service project.
5. Add the following code to your service's `main.go` file:

```go
package main

import (
"context"
"fmt"
"log"
"net"

"github.com/grpc-ecosystem/go-grpc-middleware/auth"
"github.com/grpc-ecosystem/go-grpc-middleware/logging/zap"
"github.com/grpc-ecosystem/go-grpc-middleware/recovery"
"github.com/grpc-ecosystem/go-grpc-middleware/validator"
"go.opencensus.io/plugin/ocgrpc"
"go.uber.org/zap"
"google.golang.org/grpc"
"google.golang.org/grpc/codes"
"google.golang.org/grpc/credentials"
"google.golang.org/grpc/credentials/oauth"
"google.golang.org/grpc/status"
)

func main() {
// Create a new gRPC server.
srv := grpc.NewServer(
grpc.UnaryInterceptor(
grpc_auth.UnaryServerInterceptor(
grpc_auth.WithClaimsAuthorizer(func(ctx context.Context, claims map[string]interface{}) error {
// Check if the claims are valid.
if claims["email"] != "service-account@project-id.iam.gserviceaccount.com" {
return status.Error(codes.Unauthenticated, "invalid claims")
}

// The claims are valid, so allow the request.
return nil
}),
),
),
grpc.UnaryInterceptor(grpc_zap.UnaryServerInterceptor(zap.NewExample())),
grpc.UnaryInterceptor(grpc_recovery.UnaryServerInterceptor()),
grpc.UnaryInterceptor(grpc_validator.UnaryServerInterceptor()),
grpc.StatsHandler(&ocgrpc.ServerHandler{}),
)

// Create a new JWT credential.
cred, err := oauth.NewServiceAccountFromFile("service-account.json")
if err != nil {
log.Fatalf("failed to create JWT credential: %v", err)
}

// Create a new gRPC connection.
conn, err := grpc.Dial("localhost:50051", grpc.WithTransportCredentials(credentials.NewTLS(nil)), grpc.WithPerRPCCredentials(cred))
if err != nil {
log.Fatalf("failed to create gRPC connection: %v", err)
}
defer conn.Close()

// Create a new gRPC client.
client := NewGreeterClient(conn)

// Call the gRPC service.
resp, err := client.SayHello(context.Background(), &HelloRequest{Name: "World"})
if err != nil {
log.Fatalf("failed to call gRPC service: %v", err)
}

// Print the response.
fmt.Println(resp.Message)
}
```

6. Run the service.
7. Call the service using the gRPC client.

## Troubleshooting

If you encounter any problems, please consult the following resources:

- [gRPC documentation](https://grpc.io/docs/)
- [JWT documentation](https://jwt.io/)
- [Google Cloud documentation](https://cloud.google.com/)