# CosmosDB_with_Csharp-sample1-quickstart

1. Login in Azure CosmosDB database

![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/c62911e0-7eb6-4bec-a9d0-9dad36f4f7df)


2. Create a new Azure CosmosDB account
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/b445eb97-b81b-4c2b-9d11-00f2e60369e1)


Source code
```csharp
using Microsoft.Azure.Cosmos;

using CosmosClient client = new(
    accountEndpoint: Environment.GetEnvironmentVariable("COSMOS_ENDPOINT")!,
    authKeyOrResourceToken: Environment.GetEnvironmentVariable("COSMOS_KEY")!
);

Database database = await client.CreateDatabaseIfNotExistsAsync(
    id: "cosmicworks"
);
Console.WriteLine($"New database:\t{database.Id}");

// Container reference with creation if it does not already exist
Container container = await database.CreateContainerIfNotExistsAsync(
    id: "products",
    partitionKeyPath: "/categoryId",
    throughput: 400
);
Console.WriteLine($"New container:\t{container.Id}");

// Create new object and upsert (create or replace) to container
Product newItem = new(
    id: "70b63682-b93a-4c77-aad2-65501347265f",
    categoryId: "61dba35b-4f02-45c5-b648-c6badc0cbd79",
    categoryName: "gear-surf-surfboards",
    name: "Yamba Surfboard",
    quantity: 12,
    sale: false
);
Product createdItem = await container.CreateItemAsync<Product>(
    item: newItem,
    partitionKey: new PartitionKey("61dba35b-4f02-45c5-b648-c6badc0cbd79")
);
Console.WriteLine($"Created item:\t{createdItem.id}\t[{createdItem.categoryName}]");

// Point read item from container using the id and partitionKey
Product readItem = await container.ReadItemAsync<Product>(
    id: "70b63682-b93a-4c77-aad2-65501347265f",
    partitionKey: new PartitionKey("61dba35b-4f02-45c5-b648-c6badc0cbd79")
);

// Create query using a SQL string and parameters
var query = new QueryDefinition(
    query: "SELECT * FROM products p WHERE p.categoryId = @categoryId"
)
    .WithParameter("@categoryId", "61dba35b-4f02-45c5-b648-c6badc0cbd79");

using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
    queryDefinition: query
);

while (feed.HasMoreResults)
{
    FeedResponse<Product> response = await feed.ReadNextAsync();
    foreach (Product item in response)
    {
        Console.WriteLine($"Found item:\t{item.name}");
    }
}
```
