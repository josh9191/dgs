## Relay Pagination
The DGS framework supports dynamic generation of schema types for cursor based pagination based on the [relay spec](https://relay.dev/graphql/connections.htm).
When a type in the graphql schema is annotated with the `@connection` directive, the framework generates the corresponding `Connection` and `Edge` types, along with the common `PageInfo`.

This avoids boilerplate code around defining related Connection and Edge types in the schema for every type that needs to be paginated. 

!!!note
    The `@connection` directive only works for DGSs that are not required to register the static schema file with an external service (since the relay types are dynamically generated).
    For example, in a federated architecture involving a gateway, some gateway implementations may or may not recognize the `@connection` directive when working with a static schema file.


## Set up 
To enable the use of `@connection` directive for generating the schema for pagination, add the following module to dependencies in your build.gradle:

```
dependencies {
    implementation 'com.netflix.graphql.dgs:graphql-dgs-pagination'
}
```

Next, add the directive on the type you want to paginate.

```graphql
type Query {
      hello: MessageConnection
}
            
type Message @connection {
    name: String
}
```

Note that the `@connection` directive is defined automatically by the framework, so there is no need to add it to your schema file.

This results in the following relay types dynamically generated and added to the schema:

```graphql
type MessageConnection {
    edges: [MessageEdge]
    pageInfo: PageInfo
}

type MessageEdge {
    node: Message
    cursor: String
}

type PageInfo {
    hasPreviousPage: Boolean!
    hasNextPage: Boolean!
    startCursor: String
    endCursor: String
}
```


You can now use the corresponding `graphql.relay` [types](https://www.javadoc.io/doc/com.graphql-java/graphql-java/16.2/graphql/relay/package-summary.html) for `Connection<T>`, `Edge<T>` and `PageInfo` to set up your datafetcher as shown here:

```java
@DgsData(parentType = "Query", field = "hello")
public Connection<Message> hello(DataFetchingEnvironment env) {
    return new SimpleListConnection<>(Collections.singletonList(new Message("This is a generated connection"))).get(env);
}
```
