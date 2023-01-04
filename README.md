# Ocelot Api-Gateway Implementation (Incomplete)
### Agenda :
1. Simple Ocelot api-gateway front of services  (proxy)
2. Request Agrrigation
3. Perform Rate Limmiting
4. Perform Caching
5. Identity Server between Ocelot and Services
6. Load Balancing
7. Service Discovery


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

### 1.5 Map the Api routs to api gateway

<table>

<tr> <th>Method</th> <th>Service Address</th> <th>Gateway</th> </tr>
<tr> <td>GET</td> <td>https://localhost:7000/api/ADummy</td> <td>https://localhost:9000/dummyofa</td> </tr> 
<tr> <td>POST</td> <td>https://localhost:7000/api/ADummy</td> <td>https://localhost:9000/dummyofa</td> </tr>
<tr> <td>GET</td> <td>https://localhost:8000/api/BDummy</td> <td>https://localhost:9000/dummyofb</td> </tr> 
<tr> <td>POST</td> <td>https://localhost:8000/api/BDummy</td> <td>https://localhost:9000/dummyofb</td> </tr> 
</table>

the above mapping can be configured in **Ocelot.json** as 
```
{
  "Routes": [


    {
      "DownstreamPathTemplate": "/api/ADummy",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 7000
        }
      ],
      "UpstreamPathTemplate": "/dummyofa",
      "UpstreamHttpMethod": [ "GET", "POST" ]
    },


    {
      "DownstreamPathTemplate": "/api/BDummy",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 8000
        }
      ],
      "UpstreamPathTemplate": "/dummyofb",
      "UpstreamHttpMethod": [ "GET" ]
    }





  ],

  "GlobalConfiguration": {
    "BaseUrl": "https://localhost:9000"
  }

}
```

## 2. Request Agrrigation
Some times the response of client request includes data from multple services. In such case there are two options. We may create an Aggrigator service which recieves a request and calls other services for response and then wrap them to certain data model and response back which seems costly.another way is to use Api gateway request aggrigation which api gateway itself dispatch requests to services and merge their responses data but not wraping with data model here. 

```
{
  "Routes": [


    {
      "DownstreamPathTemplate": "/api/ADummy",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 7000
        }
      ],
      "UpstreamPathTemplate": "/dummyofa",
      "UpstreamHttpMethod": [ "GET", "POST" ],
      "Key": "serviceA"
    },


    {
      "DownstreamPathTemplate": "/api/BDummy",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 8000
        }
      ],
      "UpstreamPathTemplate": "/dummyofb",
      "UpstreamHttpMethod": [ "GET" ],
      "Key": "serviceB"
    }


  ],

  "Aggregates": [
    {
      "RouteKeys": [
        "serviceA",
        "serviceB"
      ],
      "UpstreamPathTemplate": "/aggreagterequest"
    }
  ],

  "GlobalConfiguration": {
    "BaseUrl": "https://localhost:9000"
  }

}
```
The result is an object with properties named as the key defined for each rout in Ocelot.json rout configuration. The result shown as 
```
{
"serviceA": [
{
"aSomeTitle": "70Q06D2IWA",
"aSomeValue": 3
},
{
"aSomeTitle": "U40VBU3Y55",
"aSomeValue": 3
},
{
"aSomeTitle": "AUQB6XYHTQ",
"aSomeValue": 4
}

],
"serviceB": [
{
"bSomeTitle": "QLZ72H6",
"bSomeValue": 2
},
{
"bSomeTitle": "ZGPJ9JC",
"bSomeValue": 2
},
{
"bSomeTitle": "KCDLQKH",
"bSomeValue": 2
]
}
```

## 3 Perform Rate Limmiting

ClientWhitelist - This is an array that contains the whitelist of the client. It means that the client in this array will not be affected by the rate limiting.

EnableRateLimiting - This value specifies enable endpoint rate limiting.

Period - This value specifies the period that the limit applies to, such as 1s, 5m, 1h,1d and so on. If you make more requests in the period than the limit allows then you need to wait for PeriodTimespan to elapse before you make another request.

PeriodTimespan - This value specifies that we can retry after a certain number of seconds.

Limit - This value specifies the maximum number of requests that a client can make in a defined period.

```
{
  "Routes": [


    {
      "DownstreamPathTemplate": "/api/ADummy",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 7000
        }
      ],
      "UpstreamPathTemplate": "/dummyofa",
      "UpstreamHttpMethod": [ "GET", "POST" ],
      "RateLimitOptions": {
        "ClientWhitelist": [],
        "EnableRateLimiting": true,
        "Period": "2s",
        "PeriodTimespan": 1,
        "Limit": 1
      },
      "Key": "serviceA"
    },


    {
      "DownstreamPathTemplate": "/api/BDummy",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 8000
        }
      ],
      "UpstreamPathTemplate": "/dummyofb",
      "UpstreamHttpMethod": [ "GET" ],
      "RateLimitOptions": {
        "ClientWhitelist": [],
        "EnableRateLimiting": true,
        "Period": "2s",
        "PeriodTimespan": 1,
        "Limit": 1
      },
      "Key": "serviceB"
    }


  ],

  "Aggregates": [
    {
      "RouteKeys": [
        "serviceA",
        "serviceB"
      ],
      "UpstreamPathTemplate": "/aggreagterequest"
    }
  ],

  "GlobalConfiguration": {
    "BaseUrl": "https://localhost:9000"
  }

}
```
