# CosmosDB_with_Csharp-sample1-quickstart

## 1. Login in Azure CosmosDB database
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/c62911e0-7eb6-4bec-a9d0-9dad36f4f7df)

## 2. Create a new Azure CosmosDB account
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/b445eb97-b81b-4c2b-9d11-00f2e60369e1)

## 3. Press the “Create” button in the Azure CosmosDB for NoSQL
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/ed55baed-de56-4765-bc30-8c3fbd23717c)

## 4. Set the Subscription, the resource group, the account name, the location, select the capacity mode,apply the free tier discount, limit total account throughput and press the button “Review + create” 
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/89c529b5-2864-4196-a667-afb17102eb87)

## 5. Azure CosmosDB structure
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/8b5ac069-2dda-47d6-b2b9-3fe5cd5f9c8a)

## 6. Press the CosmosDB link “luiscococosmosdb” to navigate to the Azure CosmosDB account
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/80099f62-7a69-4986-a2f5-47d9fd555773)

## 7. In the Azure CosmosDB account go to the “Keys” option
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/26b8ab47-82f1-4277-a751-f430a7ecfc1b)

## 8. Set the environmental variables values: the COSMOS_ENDPOINT and the COSMOS_KEY
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/848e7a42-38d1-44a8-83e8-971f2505cdab)

## 9. Source code
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

## 10. Press in the "Data Explorer" option to access the ComosDB database
![image](https://github.com/luiscoco/CosmosDB_with_Csharp-sample1-quickstart/assets/32194879/746f9af5-75c9-4589-86a5-941ecc28e7d7)



