# Ocelot Api-Gateway Implementation
### Agenda :
1. Simple Ocelot api-gateway front of services  (proxy)
2. Identity Server between Ocelot and Services
3. Perform Rate Limmiting
4. Perform Load Balancing
5. Perform Caching

## 1. Simple Ocelot api-gateway front of services  (proxy)
The goal is to impelement the basic api gateway that acts as proxy to services as shown. This includs basic configuration of ocelot as api gateway.

![Activitydiagram1](https://user-images.githubusercontent.com/105317212/210461837-08b811af-3423-45e0-9341-11e45dc506e0.png)

### 1.1 Package
> Install-Package Ocelot

### 1.2 IOC
```
builder.Services.AddOcelot();
```

### 1.3 Middleware
```
await app.UseOcelot();
```

### 1.4 Create Ocelot json file
The Ocelot.json contains the api routs map to proxy and their confuguration. we may have to create two json files of **Ocelot.Development.json** and **Ocelot.Local.json** which indicates the configuration for certain enviroment. To distinguish these enviroment the ocelot configuration json file is injected to IOC as 
```
builder.Host.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddJsonFile($"Ocelot.{hostingContext.HostingEnvironment.EnvironmentName}.json", true, true);
});
```
so the service could indicates which ocelot json configuration to use. these enviroment may be localhost,docker, production env etc.

### 1.5 Map the Api rout map to api gateway

<table>

<tr> <th>Method</th> <th>Service Address</th> <th>Gateway</th> </tr>
<tr> <td>GET</td> <td>https://localhost:7000/api/ADummy</td> <td>https://localhost:9000/dummyofa</td> </tr> 
<tr> <td>POST</td> <td>https://localhost:7000/api/ADummy</td> <td>https://localhost:9000/dummyofa</td> </tr>
<tr> <td>GET</td> <td>https://localhost:8000/api/BDummy</td> <td>https://localhost:9000/dummyofb</td> </tr> 
<tr> <td>POST</td> <td>https://localhost:8000/api/BDummy</td> <td>https://localhost:9000/dummyofb</td> </tr> 
</table>


