# Sermant Backend User Manual

Sermant Backend includes Sermant data processing back-end module and front-end information display module, aiming to provide runtime observability capabilities for Sermant. Currently, it mainly includes Sermant Agent heartbeat information, reception and display of reported events, webhook push, configuration management and other functions.

Sermant Backend works with Sermant Agent. Sermant Agent is mounted as a data sender after the host application is started. It can regularly send heartbeat data of the current Sermant Agent (service name, host name, instance ID, version number, IP, timestamp, mounted plugin information), event data ( Sermant Agent start and stop, core service start and stop, bytecode enhancement, log data, etc.) As a data receiver, Backend can receive and process data such as heartbeats and events sent by Sermant Agent, push emergency events to webhook, and display them visually on the front end, providing the ability to observe the running status of Sermant Agent.

Sermant Backend is used in conjunction with the configuration center to manage all configuration items in the configuration center, and can view, add, modify, and delete configuration items.

> Note: Sermant Backend is a **non-essential component** and users can deploy it on demand.

## Parameter Configuration

### Sermant Agent Parameter Configuration

The instance status management and event management capabilities provided by Sermant Backend rely on the data reported by Sermant Agent. Therefore, when using Sermant Backend, you need to first configure the relevant parameters of Sermant Agent for connecting to Sermant Backend and enabling data reporting in the configuration file `agent/config/config.Modify` the following configuration in properties:

|**Parameter key** | **Description** | **Default value** | **Required**|
| ------------------ | ------------------------------------ | ---------- | ------------ |
|agent.service.heartbeat.enable  | Heartbeat service switch | false | False|
|agent.service.gateway.enable   | Gateway service switch | false | Required when enabling heartbeat service or event reporting|
| gateway.nettyIp        | Specify the IP address of Netty server                    | 127.0.0.1  | False           |
| gateway.nettyPort        | Specify the port of Netty server                    | 6888  | False           |
|event.enable | Event reporting switch | false | False|
|event.offerWarnLog | Whether to report warning level logs | false | False|
|event.offerErrorLog | Whether to report error level logs | false | False|
|event.sendInterval | Event sending interval (ms) | 30000 | Required when enabling event reporting|
|event.offerInterval | Reporting interval for the same event | 300000 | Required when enabling event reporting|

### Sermant Backend parameter configuration

Sermant Backend parameters can be modified through the `sermant-backend/src/main/resources/application.properties` configuration file before compilation and packaging. It also supports configuration through the -D parameter or environment variable before starting the jar package.

#### Basic parameters

| **Parameter Key**  | **Description**                                              | **Default Value** | **Required** |
| ------------------ | ------------------------------------------------------------ | ----------------- | ------------ |
| server.port        | The occupied port of the Sermant Backend                             | 8900              | False        |
| netty.port         | Message receiving port of net                                | 127.0.0.1         | False        |
| netty.wait.time    | Read wait time of Netty, in second                           | 60                | False        |
| max.effective.time | Time, in milliseconds, to determine whether the application is alive or not | 60000             | False        |
| max.cache.time     | Valid time in the cache for the application heartbeat, in millisecond | 600000            | False        |


#### Instance status parameters

| **Parameter Key**  | **Description**                                              | **Default Value** | **Required** |
| ------------------ | ------------------------------------------------------------ | ----------------- | ------------ |
| max.effective.time | Time, in milliseconds, to determine whether the application is alive or not | 60000             | False        |
| max.cache.time     | Valid time in the cache for the application heartbeat, in millisecond | 600000            | False        |


#### Event management parameters

| **Parameter Key**  | **Description**                                              | **Default Value** | **Required** |
| ------------------ | ------------------------------------------------------------ | ----------------- | ------------ |
| database.type | Event storage type, currently supports redis database and memory | MEMORY | False |
| database.address | redis database address | 127.0.0.1:6379 | False |
| database.user | redis database user name | default | False |
| database.password | redis database password | null | False |
| database.event.expire | Event expiration time, unit: days | 7 | False |
| webhook.eventpush.level | The webhook event push level supports three levels: EMERGENCY, IMPORTANT, and NORMAL; supports both Feishu and DingTalk webhooks | EMERGENCY | False |

#### Configure management parameters

| **Parameter Key**  | **Description**                                              | **Default Value** | **Required** |
| ------------------ | ------------------------------------------------------------ | ----------------- | ------------ |
| dynamic.config.enable | Configuration management switch | true | False |
| dynamic.config.namespace | Default namespace (used when connecting to Nacos configuration center) | default | Must be used when connecting to Nacos configuration center |
| dynamic.config.timeout | Timeout for requesting the configuration center | 30000 | Must be turned on when the configuration management switch is turned on |
| dynamic.config.serverAddress | The connection address of the configuration center | 127.0.0.1:30110 | Must be turned on when the configuration management switch is turned on |
| dynamic.config.dynamicConfigType | Type of configuration center, supports ZOOKEEPER, NACOS, KIE | KIE | Must be turned on when the switch of configuration management is turned on |
| dynamic.config.connectTimeout |Timeout for connecting to the configuration center | 3000 | Must be turned on when the configuration management switch is turned on |
| dynamic.config.enableAuth | Whether to enable authorization authentication, support Nacos and Zookeeper | false | Must be turned on when the configuration management switch is turned on |
| dynamic.config.userName |Username (plain text) used during authorization authentication | null | Required when enabling authorization authentication |
| dynamic.config.password |Password used for authorization authentication (ciphertext encrypted by AES) | null | Required when enabling authorization authentication |
| dynamic.config.secretKey |The key used when the password is encrypted using AES| null | Required when enabling authorization authentication |

## Versions Supported

Sermant Backend is developed using JDK 1.8, so the running environment requires JDK 1.8 or above.

- [HuaweiJDK 1.8](https://gitee.com/openeuler/bishengjdk-8) / [OpenJDK 1.8](https://github.com/openjdk/jdk) / [OracleJDK 1.8](https://www.oracle.com/java/technologies/downloads/)

## Operation and result verification

### 1 Preparation

- [Download](https://github.com/sermant-io/Sermant/releases/download/v2.0.0/sermant-2.0.0.tar.gz) Sermant
  Release package (current version recommends version 2.0.0)
- [Download](https://github.com/sermant-io/Sermant-examples/releases/download/v1.4.0/sermant-examples-springboot-registry-demo-1.4.0.tar.gz) Demo binary product Archive
- [Download](https://zookeeper.apache.org/releases.html#download)ZooKeeper (dynamic configuration center & registration center), and start

### 2 Modify Sermant Agent parameter configuration

Modify the `${path}/sermant-agent/agent/config/config.properties` file to the following content:

```shell
#Heartbeat service switch
agent.service.heartbeat.enable=true
# Gateway service switch
agent.service.gateway.enable=true
#Event switch
event.enable=true
# Report warn log switch
event.offerWarnLog=true
# Report error log switch
event.offerErrorLog=true
```

> **Note:**  ${path} is the path where sermant is located


### 3 Deploy application

#### 3.1 Deploy Sermant Backend

```shell
# windwos
java -Dwebhook.eventpush.level=NORMAL -Ddynamic.config.enable=true -Ddynamic.config.serverAddress=127.0.0.1:2181 -Ddynamic.config.dynamicConfigType=ZOOKEEPER -jar ${path}\sermant-agent\server\sermant\sermant-backend-x.x.x.jar

#linux mac
java -Dwebhook.eventpush.level=NORMAL -Ddynamic.config.enable=true -Ddynamic.config.serverAddress=127.0.0.1:2181 -Ddynamic.config.dynamicConfigType=ZOOKEEPER -jar ${path}/sermant-agent/server/sermant/sermant-backend-x.x.x.jar
```

> **Note:** ${path} is the actual installation path of sermant, and x.x.x represents a certain version number of sermant.

#### 3.2 Configure application

Unzip the Demo binary product compressed package to get service-a.jar.

```shell
# windwos
java -Dserver.port=8989 -javaagent:${path}\sermant-agent\agent\sermant-agent.jar=appName=default -jar service-a.jar

#linux mac
java -Dserver.port=8989 -javaagent:${path}/sermant-agent/agent/sermant-agent.jar=appName=default -jar service-a.jar
```server.port=8989 -javaagent:${path}/sermant-agent/agent/sermant-agent.jar=appName=default -jar service-a.jar
````

> **Note:** ${path} is the actual installation path of sermant

### 4 Instance status verification

Access the address `http://127.0.0.1:8900/` through the browser to view the front-end display page. If the heartbeat information of the Sermant Agent instance is displayed as shown below, the heartbeat verification is successful.

<MyImage src="/docs-img/backend/en/backend-instance.png"></MyImage>

### 5 Event Management Verification

By clicking the Observe button in the event management tab, you can view the event information reported by the Sermant Agent. If the event information reported by the Sermant Agent instance is displayed on the page as shown below, it is verified that the event was reported successfully.

<MyImage src="/docs-img/backend/en/backend-event.png"></MyImage>

#### 5.1 Verification event query

**5.1.1 Report time query**
  
  On the **Event Management -> Monitoring** page, set the query event time range at the red box in the figure below, and click the query button to query.

  <MyImage src="/docs-img/backend/en/backend-event-query-time.png"></MyImage>

**5.1.2 Service name query**
  
  On the **Event Management -> Monitoring** page, set the position of the red box in the figure below to query by service name, enter the service name to be queried (single or multiple service name queries are supported), and click the query button to query

  <MyImage src="/docs-img/backend/en/backend-event-query-service-1.png"></MyImage>

  <MyImage src="/docs-img/backend/en/backend-event-query-service-2.png"></MyImage>

**5.1.3 ip query**
  
  On the **Event Management -> Monitoring** page, set the position of the red box in the figure below to query by ip, enter the IP address to be queried (single or multiple ip queries are supported), and click the query button to query

  <MyImage src="/docs-img/backend/en/backend-event-query-ip-1.png"></MyImage>

  <MyImage src="/docs-img/backend/en/backend-event-query-ip-2.png"></MyImage>

**5.1.4 Level Query**
  
  On the **Event Management -> Monitoring** page, select the event level that needs to be queried in the red box in the figure below. Multiple selections are supported. After selecting, click Filter to query.

  <MyImage src="/docs-img/backend/en/backend-event-query-level.png "></MyImage>

**5.1.5 Type Query**
  
  On the **Event Management -> Monitoring** page, select the event type that needs to be queried at the red box in the figure below. Multiple selections are supported. After selecting, click Filter to query.

  <MyImage src="/docs-img/backend/en/backend-event-query-type.png"></MyImage>

**5.1.6 Detailed information display**
  
  On the **Event Management -> Monitoring** page, click on the red box in the image below to view event details

  <MyImage src="/docs-img/backend/en/backend-event-detail.png"></MyImage>

**5.1.7 Events automatically refreshed**
  
  On the **Event Management -> Monitoring** page, click the red box auto-refresh button in the picture below to enable automatic event refresh.(After turning it on, the latest events will be automatically obtained regularly. Click the button again to close it, or it will automatically close when viewing the event list.)

  <MyImage src="/docs-img/backend/en/backend-event-auto.png"></MyImage>

#### 5.2 Verify webhook event notification

- Access the address `http://127.0.0.1:8900/` through the browser
- Click on the menu bar **Event Management -> Configuration** to enter the webhook configuration interface, as shown in the figure below:

<MyImage src="/docs-img/backend/en/backend-event-manager.png"></MyImage>

<MyImage src="/docs-img/backend/en/backend-webhook.png"></MyImage>

- Turn on webhook, as shown in the figure below:

<MyImage src="/docs-img/backend/en/backend-webhook-enable.png"></MyImage>

- Click the edit button of webhook and set the webhook address, as shown in the figure below:

<MyImage src="/docs-img/backend/en/backend-webhook-url.png"></MyImage>

- Click the test connection button of the webhook to receive the test event notification in the corresponding webhook
  - Feishu test event push is shown in the figure below:

    <MyImage src="/docs-img/backend/backend-webhook-feishu.png"></MyImage>

  - DingTalk test event push is shown in the figure below:

    <MyImage src="/docs-img/backend/backend-webhook-dingding.png"></MyImage>

### 6 Configuration management verification

- Access the address `http://127.0.0.1:8900/` through the browser
- Click **Configuration Management** on the menu bar to enter the configuration management interface to view the configuration of the configuration center. The configuration management page will display the configuration of the routing plugin by default, as shown in the following figure:

<MyImage src="/docs-img/backend/en/backend-config-manager.png"></MyImage>

#### 6.1 Verification configuration added

- On the configuration management page, click the **Create** button to jump to the configuration details page to add new configurations, as shown in the figure below:

<MyImage src="/docs-img/backend/en/backend-config-add.png"></MyImage>

- After entering the necessary configuration information on the configuration details page, click the Submit button to add the configuration, as shown in the figure below:

<MyImage src="/docs-img/backend/en/backend-config-add-info.png"></MyImage>

- After the configuration is successfully added, the configuration item can be queried on the configuration management page.

#### 6.2 Verify configuration query

- On the configuration management page, you can select different plugin types to query different plugin configurations, or enter different query conditions (such as service name, application name, etc.) to query plugin configurations that meet the conditions. The following is the result of querying the configuration item of the flow control plugin and the service name is default (added in the previous step):

<MyImage src="/docs-img/backend/en/backend-config-query.png"></MyImage>

#### 6.3 Verify configuration modifications

- On the configuration management page, you can click the **View** button to jump to the configuration details page to modify the configuration content. As shown below:

<MyImage src="/docs-img/backend/en/backend-config-modify-1.png"></MyImage>

- The configuration content can be modified on the configuration details page. After modification, click the submit button to submit the modified content to the configuration center.

<MyImage src="/docs-img/backend/en/backend-config-modify-2.png"></MyImage>

- Enter the configuration details page again to view the modified configuration content.

<MyImage src="/docs-img/backend/en/backend-config-modify-3.png"></MyImage>

#### 6.4 Verify configuration deletion

- You can delete configurations on the configuration management page. Clicking the **Delete** button will prompt you whether to delete the configuration. As shown below:

<MyImage src="/docs-img/backend/en/backend-config-delete-1.png"></MyImage>

- After selecting **Yes**, the configuration item will be deleted, as shown in the figure below:

<MyImage src="/docs-img/backend/en/backend-config-delete-2.png"></MyImage>