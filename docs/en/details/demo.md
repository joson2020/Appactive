# AppActive Demo

## Overall structure
![appactive_landscape](https://appactive.oss-cn-beijing.aliyuncs.com/images/AppActive-demo.png)

The overall structure of this demo is shown in the figure.

Note: The registry nacos and database mysql that the application depends on are not shown in the figure.

There are 2 units:

-center: center unit
-unit: ordinary unit

There are 4 applications in total, according to the distance (call link) of the end user from near and far:

-gateway: Multi-active gateway, which distributes user traffic
-frontend: frontend application, accept user requests, and return after requesting actual data
-product: product application, providing three services:
-Product List: General Service
-Product Details: Unit Service
-Product order: central service, relying on inventory application
-storage: Inventory application, for ordering service to deduct inventory

For the sake of simplicity, the gateway is only deployed in the center, and the rest of the applications are deployed in each of the center and unit.

The green grid in the figure represents the call link of this request.

## Quick Start

### Premise
This demo requires the following software to be installed

-docker && docker-compose

### Step

1. Enter the nginx-plugin directory of the `appactive-gateway` module and mark it as a mirror: `docker build --build-arg UNITFLAG=center -t app-active/gateway:1.0-SNAPSHOT .`
2. Enter the `appactive-demo` module, maven build to get the jar package
3. Run `./run.sh` in the `appactive-demo` module to start all applications
4. Run `baseline.sh` in the `appactive-portal` module to push the baseline
5. Bind the local host: `127.0.0.1 demo.appactive.io`, visit the browser `http://demo.appactive.io/listProduct?r_id=1999` to see the effect
6. Run `cut.sh` in the `appactive-portal` module to cut the flow. This demo supports two cutting methods: precision and range,
   The cut flow commands are: `./cut.sh In` and `./cut.sh Between`. It should be noted that the write prohibition rules of this demo are hard-coded. If the user wants to change the cut flow range, he needs to calculate the write prohibition rule and the next routing rule by himself, and then execute the cut flow.

## Rule description

After running all applications, we ran baseline.sh and actually did the following things:

-Push rules to gateway through http channel
-Push rules to other applications through file channels

The rules include

-idSource.json: Describes how to extract routing labels from http traffic
-idTransformer.json: Describe how to parse the routing mark
-idUnitMapping.json: Describe the mapping relationship between the routing mark and the unit
-machine.json: describes the attribution unit of the current machine
-mysql-product: describe the attributes of the database

### Cut flow
Mainly do the following things when cutting flow:

-Build new mapping rules and banning rules (manually)
-Push new mapping rules to gateway
-Push banning rules to other apps
-Wait for the data to tie and push the new mapping relationship rules to other applications

Note that the new mapping relationship is the target state you want to achieve, and the prohibition rule is the difference calculated based on the target state and the status quo. Currently, both of these need to be manually set and updated to the corresponding json file under `appactive-portal/rule`, and then run `./cut.sh In` or `./cut.sh Between`