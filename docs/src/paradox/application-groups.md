---
title: Application Groups
---

# Application Groups

Applications can be nested into a n-ary tree, with groups as branches and applications as leaves.
Application Groups are used to partition multiple applications into manageable sets.

<p class="text-center">
  <img src="{{ site.baseurl}}/img/hierarchy.png" width="645" height="220" alt="">
</p>


The definition could look like this:


```json
{
  "id": "/product",
  "groups": [
    {
      "id": "/product/database",
      "apps": [
         { "id": "/product/database/mongo", ... },
         { "id": "/product/database/mysql", ... }
       ]
    },{
      "id": "/product/service",
      "dependencies": ["/product/database"],
      "apps": [
         { "id": "/product/service/rails-app", ... },
         { "id": "/product/service/play-app", ... }
      ]
    }
  ]
}
```

**Note:** Deploy an application group through a `POST` to the `/v2/groups` REST API endpoint.

## Dependencies 

Applications can have dependencies. For example a Play application could require a database to run. 
If the dependencies are defined in the application specification, then Marathon keeps track of the
correct order of action for starting, stopping and upgrading the applications.

Dependencies can be expressed on the level of applications and on the level of application groups.
If a dependency is expressed on the level of groups, this dependency is inherited by all transitive groups and all transitive applications of this group.  

Dependencies can be expressed either by absolute or by relative path.

Example:
If defined on the application group service, all 3 definitions have the same meaning:

```json
{
  ...
  "dependencies": ["/product/database"],
  "dependencies": ["../database"],
  "dependencies": ["specific/../../database"],
  ...
}  
```

  <div class="alert alert-info"> 
    <strong>Note:</strong> If your app fails to start, then other applications depending on it will remain blocked indefinitely. But if you decide to suspend the broken app (ex. to try and fix it), Marathon will assume that the deployment is completed and upgrade the dependent apps to the target version. At this point, re-deploying the broken app will not restart the dependent apps because Marathon assumes that the target version is already achieved. As a work-around you could try adding a label to the dependent apps in order to have Marathon restart all tasks.
  </div>

## Group scaling

A whole group can be scaled.
The instance count of all transitive applications is changed accordingly.

```http
PUT /v2/groups/product HTTP/1.1
Content-Length: 21
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{ "scaleBy": 2 }
```

The instance count of each application is doubled after that operation.