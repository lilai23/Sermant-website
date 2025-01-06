# Sermant Agent User Manual

Sermant Agent is the core component providing bytecode enhancement capabilities and various service governance functionalities. The content under the `sermant-agent-x.x.x/agent` directory described in the [Sermant Introduction](readme.md) outlines the modules of the Sermant Agent component. The main body of Sermant Agent offers foundational bytecode enhancement capabilities and a development framework, while also supporting essential features such as heartbeat functionality, dynamic configuration, logging, and event reporting. It currently supports both `premain` and `agentmain`startup methods.

The Sermant Agent plugin directory includes plugins that provide service governance capabilities such as tag routing, rate limiting and degradation, and dual registration. It now supports the dynamic installation and uninstallation of service governance plugins while the host service is running, provided the plugins support dynamic installation and uninstallation.

## Supported Version

Sermant Agent supports Linux and Windows, and is developed based on JDK 1.8. It is recommended to use JDK 1.8 or higher versions.

- [HuaweiJDK 1.8](https://gitee.com/openeuler/bishengjdk-8) / [OpenJDK 1.8](https://github.com/openjdk/jdk) / [OracleJDK 1.8](https://www.oracle.com/java/technologies/downloads/)

## Premain Startup: Static Attachment

To start Sermant Agent using the `premain` method via static attachment, configure the host service with the `-javaagent`option. Based on the environment set up in the [Quick Start Guide](../QuickStart.md), execute the following command to launch Sermant Agent:

```shell
# linux mac
java -javaagent:${path}/sermant-agent-x.x.x/agent/sermant-agent.jar -jar spring-provider.jar

# windows
java -javaagent:${path}\sermant-agent-x.x.x\agent\sermant-agent.jar -jar spring-provider.jar
```

Check the beginning of the `spring-provider.jar` log to see if it contains the following content:

```shell
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Loading god library into BootstrapClassLoader.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Building argument map by agent arguments.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Loading core library into SermantClassLoader.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Loading sermant agent, artifact is: default
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Load sermant done, artifact is: default
```

If the log outputs as shown above, it indicates that Sermant Agent has started successfully. Open your browser and navigate to `http://localhost:8900`, where you should see a Sermant Agent instance. The expected result is shown in the image below:

<MyImage src="/docs-img/backend_sermant_info.jpg"></MyImage>

## Agentmain Startup：Dynamic Attachment

### Preparations

- Based on the environment set up in the [Quick Start Guide](../QuickStart.md), first start the host service `spring-provider.jar`.

```shell
java -jar spring-provider.jar
```

- To start using the `agentmain` method, the `Attach API` is required. Download the [Sermant Release Package](https://github.com/sermant-io/Sermant/releases/download/v2.2.0/sermant-2.2.0.tar.gz) and extract it. Retrieve the `AgentLoader` script from the `./tools` directory.

### Agent Installation

To implement Agent installation based on the `Attach API`, please run `AgentLoader` using the following command:


```shell
# Linux、MacOS
java -cp ./:$JAVA_HOME/lib/tools.jar AgentLoader

# Windows has correctly configured the environment variables required for JAVA.
java AgentLoader
```

```shell
# Choose the command to run based on the operating system you are using. Below are the commands for Linux and MacOS:
$ java -cp ./:$JAVA_HOME/lib/tools.jar AgentLoader
Please select the Java process you wish to use with Sermant Agent:
0: xxxxx AgentLoader # xxxxx represents the process ID, which is obscured here.
1: xxxxx spring-provider.jar # xxxxx represents the process ID, which is obscured here.
2: xxxxx sermant-backend-1.2.0.jar # xxxxx represents the process ID, which is obscured here.
Please enter the Java process number you wish to use with Sermant Agent: 1 # Select the process number of spring-provider.
The process ID you selected: xxxxx # xxxxx represents the process ID, which is obscured here.
Please enter the directory where Sermant Agent is located (by default, it uses sermant-agent.jar in this directory as the entry point): ${path}/sermant-agent-x.x.x/agent # Enter the directory where Sermant Agent is located.
Please select the command to execute: 
0: INSTALL-AGENT
Command description: Install the Sermant Agent and all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file.
1: UNINSTALL-AGENT
Command description: Uninstall the Sermant Agent along with all installed plugins.
2: INSTALL-PLUGINS
Command description: Install plugins into the Sermant Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
3: UNINSTALL-PLUGINS
Command description: Uninstall plugins installed in the Sermant Agent.
4: UPDATE-PLUGINS
Command description: Update plugins installed in the Sermant Agent.
5: CHECK-ENHANCEMENT
Command description: Query the plugins installed in the Sermant Agent and the corresponding enhancement information (including enhanced classes and methods, as well as the corresponding interceptors).
6: INSTALL-EXTERNAL-AGENT
Command description: install external Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
Please enter the number of the command you want to execute: 0 # Select the command number to install the Sermant Agent here.
Please enter the parameters to pass to Sermant Agent (optional, example format: key1=value1,key2=value2): appName=default # Configure Sermant Agent parameters (can be empty)
```

After completing the input according to the instructions, you can see the following content in the `spring-provider.jar`log: 

```shell
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Loading god library into BootstrapClassLoader.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Building argument map by agent arguments.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Loading core library into SermantClassLoader.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Loading sermant agent, artifact is: default
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Load sermant done, artifact is: default
```

If the log outputs as shown above, it indicates that Sermant Agent successfully read the startup command and began the installation. Open your browser and navigate to `http://localhost:8900`, where you should see a Sermant Agent instance. The expected result is shown in the image below: 

<MyImage src="/docs-img/sermant-agent-agentmain-start.png"></MyImage>

### Agent Uninstallation

> Note: To avoid potential unpredictable exceptions when uninstalling service governance capabilities developed based on the premain startup method, Sermant Agent imposes restrictions on uninstallation. Only Sermant Agents started via the agentmain method support uninstallation; those started via the premain method do not support uninstallation.

After [dynamically attaching the Agent](#agent-installation), you can uninstall Agent. Run `AgentLoader` again and pass the parameter to issue the uninstall command for Sermant Agent using `command=UNINSTALL-AGENT`:

```shell
# Choose the command to run based on the operating system you are using. Below are the commands for Linux and MacOS:
$ java -cp ./:$JAVA_HOME/lib/tools.jar AgentLoader
Please select the Java process you wish to use with Sermant Agent:
0: xxxxx AgentLoader # xxxxx represents the process ID, which is obscured here.
1: xxxxx spring-provider.jar # xxxxx represents the process ID, which is obscured here.
2: xxxxx sermant-backend-1.2.0.jar # xxxxx represents the process ID, which is obscured here.
Please enter the Java process number you wish to use with Sermant Agent: 1 # Select the process number of spring-provider.
The process ID you selected: xxxxx # xxxxx represents the process ID, which is obscured here.
Please enter the directory where Sermant Agent is located (by default, it uses sermant-agent.jar in this directory as the entry point): ${path}/sermant-agent-x.x.x/agent # Enter the directory where Sermant Agent is located.
Please select the command to execute: 
0: INSTALL-AGENT
Command description: Install the Sermant Agent and all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file.
1: UNINSTALL-AGENT
Command description: Uninstall the Sermant Agent along with all installed plugins.
2: INSTALL-PLUGINS
Command description: Install plugins into the Sermant Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
3: UNINSTALL-PLUGINS
Command description: Uninstall plugins installed in the Sermant Agent.
4: UPDATE-PLUGINS
Command description: Update plugins installed in the Sermant Agent.
5: CHECK-ENHANCEMENT
Command description: Query the plugins installed in the Sermant Agent and the corresponding enhancement information (including enhanced classes and methods, as well as the corresponding interceptors).
6: INSTALL-EXTERNAL-AGENT
Command description: install external Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
Please enter the number of the command you want to execute: 1 # Select the command number to uninstall the Sermant Agent here.
```

After completing the input according to the instructions, you can see the following content in the `spring-provider.jar`log:

```shell
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Building argument map by agent arguments.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Sermant for artifact is running, artifact is: default
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Execute command: UNINSTALL-AGENT
```

If the log outputs as shown above, open your browser and navigate to `http://localhost:8900`. You should see that the Sermant Agent instance has been shut down (**status is gray**), indicating that Sermant Agent has been successfully uninstalled. The expected result is shown in the image below:

<MyImage src="/docs-img/sermant-agent-agentmain-uninstall-success.png"></MyImage>

> Note: This capability can be implemented in the development mode by calling the `uninstall` interface provided by [AgentCoreEntrance](https://github.com/sermant-io/Sermant/blob/develop/sermant-agentcore/sermant-agentcore-core/src/main/java/io/sermant/core/AgentCoreEntrance.java) in the sermant-agentcore-core module.

### Dynamic Plugin Installation

After [dynamically attaching the Agent](#agent-installation), you can dynamically install service governance plugins (provided the plugins support dynamic installation and uninstallation). Run `AgentLoader` again and pass the parameter to issue the dynamic plugin installation command using `command=INSTALL-PLUGINS:pluginA/pluginB`:

> Note: Multiple plugins can be installed at once, separated by '/'. `pluginA` and `pluginB` are placeholders for actual plugin names. In this example, we use the [monitor](../plugin/monitor.md) plugin.

```shell
# Choose the command to run based on the operating system you are using. Below are the commands for Linux and MacOS:
$ java -cp ./:$JAVA_HOME/lib/tools.jar AgentLoader
Please select the Java process you wish to use with Sermant Agent:
0: xxxxx AgentLoader # xxxxx represents the process ID, which is obscured here.
1: xxxxx spring-provider.jar # xxxxx represents the process ID, which is obscured here.
2: xxxxx sermant-backend-1.2.0.jar # xxxxx represents the process ID, which is obscured here.
Please enter the Java process number you wish to use with Sermant Agent: 1 # Select the process number of spring-provider.
The process ID you selected: xxxxx # xxxxx represents the process ID, which is obscured here.
Please enter the directory where Sermant Agent is located (by default, it uses sermant-agent.jar in this directory as the entry point): ${path}/sermant-agent-x.x.x/agent # Enter the directory where Sermant Agent is located.
Please select the command to execute: 
0: INSTALL-AGENT
Command description: Install the Sermant Agent and all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file.
1: UNINSTALL-AGENT
Command description: Uninstall the Sermant Agent along with all installed plugins.
2: INSTALL-PLUGINS
Command description: Install plugins into the Sermant Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
3: UNINSTALL-PLUGINS
Command description: Uninstall plugins installed in the Sermant Agent.
4: UPDATE-PLUGINS
Command description: Update plugins installed in the Sermant Agent.
5: CHECK-ENHANCEMENT
Command description: Query the plugins installed in the Sermant Agent and the corresponding enhancement information (including enhanced classes and methods, as well as the corresponding interceptors).
6: INSTALL-EXTERNAL-AGENT
Command description: install external Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
Please enter the number of the command you want to execute: 2 # Select the command number to install the plugins of Sermant Agent here.
Please enter the name of the plugin you want to operate on, separated by / for multiple plugins: monitor 
# Enter the name of the plugin to be installed here. This example uses the monitor plugin for demonstration.
Please enter the parameters to pass to Sermant Agent (optional, example format: key1=value1,key2=value2):  # Configure Sermant Agent parameters (can be empty)
```

After completing the input according to the instructions, you can see the following content in the `spring-provider.jar`log:

```shell
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Building argument map by agent arguments.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Sermant for artifact is running, artifact is: default
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Execute command: INSTALL-PLUGINS:monitor # This example demonstrates using the monitor plugin.
```

If the log outputs as shown above, it indicates that the plugin was installed successfully. Open your browser and navigate to `http://localhost:8900`, where you should see that the plugin has been successfully installed. The plugins list will display the currently installed plugins, as shown in the image below:

**Before dynamically installing plugin**

<MyImage src="/docs-img/sermant-agent-dynamic-install-plugin-before.png"></MyImage>

**After dynamically installing plugin**

<MyImage src="/docs-img/sermant-agent-dynamic-install-plugin-success.png"></MyImage>

> Note: This capability can be implemented in the development mode by calling the `install(Set pluginNames)` method provided by [PluginManager](https://github.com/sermant-io/Sermant/blob/develop/sermant-agentcore/sermant-agentcore-core/src/main/java/io/sermant/core/plugin/PluginManager.java) in the sermant-agentcore-core module.

### Repeated Plugin Installation
The introduction of this capability is mainly due to the need for dynamically extending the effective scope of plugins in certain scenarios, primarily expanding the enhanced classes and methods, while ensuring that the already effective parts remain unaffected. In such cases, uninstalling and reinstalling plugins after adjusting configurations cannot address the issue.For example, in fault injection scenarios, different faults may require bytecode enhancement for different classes, and various fault scenarios need to be gradually injected according to the test plan's orchestration. In such cases, we cannot complete this task by uninstalling and reinstalling the plugin. Instead, we must install the fault injection plugin multiple times to solve this problem. This requires the capability of repeated plugin installation. Repeated plugin installation will reuse static resources, and Sermant internally isolates the repeatedly installed plugins through plugin management.

### How to Implement Repeated Plugin Installation?

To install plugins repeatedly, you need to add a unique identifier to the plugin name using the `#` symbol when performing dynamic plugin installation. For example:
```shell
command=INSTALL-PLUGINS:pluginA#FIRST
```
In this way, the plugin can be installed repeatedly.

> Note: When uninstalling a plugin, if you want to uninstall a plugin that was installed with an identifier, you need to specify the plugin name with the identifier in the uninstall command as well.


### Dynamic Plugin Uninstallation

After [dynamically attaching the Agent](#agent-installation) and [dynamically installing the plugin](#dynamic-plugin-installation), you can dynamically uninstall service governance plugins (provided the plugins support dynamic installation and uninstallation). Run `AgentLoader` again and pass the parameter to issue the dynamic plugin uninstallation command using `command=UNINSTALL-PLUGINS:pluginA/pluginB`:

> Note: Multiple plugins can be uninstalled at once, separated by '/'. `pluginA` and `pluginB` are placeholders for actual plugin names. In this example, we use the [monitor](../plugin/monitor.md) plugin.

```shell
# Choose the command to run based on the operating system you are using. Below are the commands for Linux and MacOS:
$ java -cp ./:$JAVA_HOME/lib/tools.jar AgentLoader
Please select the Java process you wish to use with Sermant Agent:
0: xxxxx AgentLoader # xxxxx represents the process ID, which is obscured here.
1: xxxxx spring-provider.jar # xxxxx represents the process ID, which is obscured here.
2: xxxxx sermant-backend-1.2.0.jar # xxxxx represents the process ID, which is obscured here.
Please enter the Java process number you wish to use with Sermant Agent: 1 # Select the process number of spring-provider.
The process ID you selected: xxxxx # xxxxx represents the process ID, which is obscured here.
Please enter the directory where Sermant Agent is located (by default, it uses sermant-agent.jar in this directory as the entry point): ${path}/sermant-agent-x.x.x/agent # Enter the directory where Sermant Agent is located.
Please select the command to execute: 
0: INSTALL-AGENT
Command description: Install the Sermant Agent and all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file.
1: UNINSTALL-AGENT
Command description: Uninstall the Sermant Agent along with all installed plugins.
2: INSTALL-PLUGINS
Command description: Install plugins into the Sermant Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
3: UNINSTALL-PLUGINS
Command description: Uninstall plugins installed in the Sermant Agent.
4: UPDATE-PLUGINS
Command description: Update plugins installed in the Sermant Agent.
5: CHECK-ENHANCEMENT
Command description: Query the plugins installed in the Sermant Agent and the corresponding enhancement information (including enhanced classes and methods, as well as the corresponding interceptors).
6: INSTALL-EXTERNAL-AGENT
Command description: install external Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
Please enter the number of the command you want to execute: 3 # Select the command number to uninstall the plugins of Sermant Agent here.
Please enter the name of the plugin you want to operate on, separated by / for multiple plugins: monitor 
# Enter the name of the plugin to be uninstalled here. This example uses the monitor plugin for demonstration.
```

After completing the input according to the instructions, you can see the following content in the `spring-provider.jar`log:

```shell
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Building argument map by agent arguments.
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Sermant for artifact is running, artifact is: default
[xxxx-xx-xxTxx:xx:xx.xxx] [INFO] Execute command: UNINSTALL-PLUGINS:monitor #This example demonstrates using "monitor" plugin.
# The log will show how many bytecode-enhanced classes were restored during the uninstallation.
[Byte Buddy] REDEFINE BATCH #0 [1 of 1 type(s)]
[Byte Buddy] REDEFINE COMPLETE 1 batch(es) containing 1 types [0 failed batch(es)]
[Byte Buddy] REDEFINE COMPLETE 1 batch(es) containing 0 types [0 failed batch(es)]
```

If the log outputs as shown above, open your browser and navigate to `http://localhost:8900`. You should see that the plugin has been successfully uninstalled. The plugins list will display the currently installed plugins, as shown in the image below:

**Before dynamically installing plugin**

<MyImage src="/docs-img/sermant-agent-dynamic-uninstall-plugin-before.png"></MyImage>

**After dynamically installing plugin**

<MyImage src="/docs-img/sermant-agent-dynamic-uninstall-plugin-success.png"></MyImage>

> Note: This capability can be implemented in development mode by calling the `uninstall(Set pluginNames)` method provided by [PluginManager](https://github.com/sermant-io/Sermant/blob/develop/sermant-agentcore/sermant-agentcore-core/src/main/java/io/sermant/core/plugin/PluginManager.java) in the sermant-agentcore-core module.

### One-Click Agent and Plugin Attachment

The [Sermant Dynamic Installation and Uninstallation Script](https://github.com/sermant-io/Sermant/blob/develop/scripts/attach.c) is a C language script based on the Java Attach API. It can attach Sermant Agent to JVM processes on either **virtual machines** or **containers**.

> Note: This script is only for use on **Linux systems**.

#### Parameter Configuration

- `-path=`：required parameter, followed by the **absolute path** of sermant-agent.jar.

- `-pid=`：required parameter, followed by the PID of the host microservice, which can be found using commands like `ps -ef`.

- `-command=`：required parameter, followed by the command to attach Sermant Agent. Supported commands can be found in the [Sermant Command Instructions](#sermant-command-instructions).

- `-nspid=`：when the host microservice is running in a container, this parameter is required and should be followed by the host microservice's nspid, which can be found using the `cat /proc/{pid}/status` command. If the host microservice is not running in a container, do not include this parameter.

#### Script Usage Steps

##### Step 1. Compile `jvm_attach.c`

```bash
gcc attach.c -o attach
```

> Note: Please ensure that `gcc` is already installed.

##### Step 2. Execute the Attach Script

```bash
./attach -path={sermant-path}/sermant-agent.jar -pid={pid} -command={COMMAND}
```

The script execution will display as follows:

```shell
[root@b6b9af8e5610 root]# ./attach -path=/home/sermant-agent-1.0.0/agent/sermant-agent.jar -pid=494 -command=INSTALL-PLUGINS:database-write-prohibition
[INFO]: PATH: /home/sermant-agent-1.0.0/agent/sermant-agent.jar
[INFO]: PID: 494
[INFO]: COMMAND: INSTALL-PLUGINS:database-write-prohibition
[INFO]: Connected to remote JVM of pid 494
[INFO]: ret code is 0, Attach success!
```

## Enhancement Information Query
After Sermant has been successfully started by any method, you can run `AgentLoader` and pass the parameter to issue the query enhancement information command using `command=CHECK_ENHANCEMENT`:

> Note: The enhancement information query will be printed to the log at the **INFO level**. If you use this feature, please configure the log level in advance. For modification instructions, see [Log Configuration](../developer-guide/log-func.md#configuration)

```shell
# Choose the command to run based on the operating system you are using. Below are the commands for Linux and MacOS:
$ java -cp ./:$JAVA_HOME/lib/tools.jar AgentLoader
Please select the Java process you wish to use with Sermant Agent:
0: xxxxx AgentLoader # xxxxx represents the process ID, which is obscured here.
1: xxxxx spring-provider.jar # xxxxx represents the process ID, which is obscured here.
2: xxxxx sermant-backend-1.2.0.jar # xxxxx represents the process ID, which is obscured here.
Please enter the Java process number you wish to use with Sermant Agent: 1 # Select the process number of spring-provider.
The process ID you selected: xxxxx # xxxxx represents the process ID, which is obscured here.
Please enter the directory where Sermant Agent is located (by default, it uses sermant-agent.jar in this directory as the entry point): ${path}/sermant-agent-x.x.x/agent # Enter the directory where Sermant Agent is located.
Please select the command to execute: 
0: INSTALL-AGENT
Command description: Install the Sermant Agent and all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file.
1: UNINSTALL-AGENT
Command description: Uninstall the Sermant Agent along with all installed plugins.
2: INSTALL-PLUGINS
Command description: Install plugins into the Sermant Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
3: UNINSTALL-PLUGINS
Command description: Uninstall plugins installed in the Sermant Agent.
4: UPDATE-PLUGINS
Command description: Update plugins installed in the Sermant Agent.
5: CHECK-ENHANCEMENT
Command description: Query the plugins installed in the Sermant Agent and the corresponding enhancement information (including enhanced classes and methods, as well as the corresponding interceptors).
6: INSTALL-EXTERNAL-AGENT
Command description: install external Agent. If the Sermant Agent is not installed, it will be automatically installed (along with all plugins listed under dynamicPlugins.active in the plugins.yaml configuration file).
Please enter the number of the command you want to execute: 5 # Select the command number to query enhancement information.
```

After completing the input according to the instructions, you can see the following content in the Sermant log:
```shell
xxxx-xx-xx xx:xx:xx.xxx [INFO] [io.sermant.core.command.CheckEnhancementsCommandExecutor] [execute:42] [Attach Listener] ---------- PLUGINS ----------
xxxx-xx-xx xx:xx:xx.xxx [INFO] [io.sermant.core.command.CheckEnhancementsCommandExecutor] [execute:44] [Attach Listener] test-plugin-A:1.0.0
xxxx-xx-xx xx:xx:xx.xxx [INFO] [io.sermant.core.command.CheckEnhancementsCommandExecutor] [execute:44] [Attach Listener] test-plugin-B:1.0.0
xxxx-xx-xx xx:xx:xx.xxx [INFO] [io.sermant.core.command.CheckEnhancementsCommandExecutor] [execute:46] [Attach Listener] ---------- ENHANCEMENT ----------
xxxx-xx-xx xx:xx:xx.xxx [INFO] [io.sermant.core.command.CheckEnhancementsCommandExecutor] [execute:58] [Attach Listener] test-plugin-A:1.0.0
xxxx-xx-xx xx:xx:xx.xxx [INFO] [io.sermant.core.command.CheckEnhancementsCommandExecutor] [execute:65] [Attach Listener] xxxxx.xxxx.TestClassA#testFunctionA(boolean,java.lang.String,java.lang.String,java.lang.String)@sun.misc.Launcher$AppClassLoader@5c647e05 [xxxx.xxxx.TestInterceptorA]
xxxx-xx-xx xx:xx:xx.xxx [INFO] [io.sermant.core.command.CheckEnhancementsCommandExecutor] [execute:65] [Attach Listener] xxxxx.xxxx.TestClassB#testFunctionB(boolean,java.lang.String,java.lang.String,java.lang.String)@sun.misc.Launcher$AppClassLoader@5c647e05 [xxxx.xxxx.TestInterceptorB,xxxx.xxxx.TestInterceptorC]
```

The format of the printed content is as follows:
```shell
---------- PLUGINS ----------
\\ List of installed plugins, formatted as: PluginName: PluginVersion
test-plugin-A:1.0.0
test-plugin-B:1.0.0
---------- ENHANCEMENT ----------
\\ Plugins that successfully completed enhancement processing, formatted as: PluginName: PluginVersion
test-plugin-A:1.0.0
\\ Information on plugins that successfully completed enhancement processing:
\\ Formatted as: EnhancedFullQualifiedClassName#EnhancedMethodName(ParameterType)@ClassLoaderInformation [InterceptorList]
xxxxx.xxxx.TestClassA#testFunctionA(boolean,java.lang.String,java.lang.String,java.lang.String)@sun.misc.Launcher$AppClassLoader@5c647e05 [xxxx.xxxx.TestInterceptorA]
xxxxx.xxxx.TestClassB#testFunctionB(boolean,java.lang.String,java.lang.String,java.lang.String)@sun.misc.Launcher$AppClassLoader@5c647e05 [xxxx.xxxx.TestInterceptorB,xxxx.xxxx.TestInterceptorC]
```

## Sermant Command Instructions

Sermant can achieve hot-plugging capabilities by running `AgentLoader` and passing in the following commands. Additionally, after Sermant Agent has been successfully started by any method, you can run `AgentLoader` and pass in commands to query enhancement information. The specific commands are as follows:

| Command Type                  | Command Example                                              |
| ----------------------------- | ------------------------------------------------------------ |
| Agent Installation            | If the command is empty, it defaults to Agent installation.  |
| Agent Uninstallation          | command=UNINSTALL-AGENT                                      |
| Plugin Installation           | command=INSTALL-PLUGINS:${PluginName}                        |
| Plugin Uninstallation         | command=UNINSTALL-PLUGINS:${PluginName}                      |
| Repeated Plugin Installation  | command=INSTALL-PLUGINS:${PluginName}#${CustomPluginIdentifier} |
| Enhancement Information Query | command=CHECK_ENHANCEMENT                                    |
| Inatall External Agent        | command=INSTALL-EXTERNAL-AGENT:${Eeternal Agent Name}        |

## Sermant Core Services

### Dynamic Configuration Service

The Sermant Dynamic Configuration Service allows Sermant to pull configurations from the dynamic configuration center, enabling rich service governance capabilities. For a detailed introduction and usage of the Sermant Dynamic Configuration Service, please refer to the [Dynamic Configuration Center User Manual](./configuration-center.md).

### xDS Service

The Sermant xDS Service enables microservices to connect to Istio in Kubernetes scenarios. Sermant communicates directly with the Istio control plane based on the xDS protocol to retrieve configuration information for service discovery, routing, load balancing, and more, thereby replacing Envoy as the data plane for service governance capabilities in Istio. For a detailed introduction and usage of the Sermant xDS Service, please refer to [Proxyless Service Mesh Based onSermant + Istio](./sermant-xds.md).

### Metric Service

The Sermant Metric Service allows users to collect and display core metrics and custom metrics from Sermant using monitoring tools like Prometheus. Through these metrics, users can gain real-time insights into the health of services and promptly identify potential issues.

#### Prerequisites

Before using the Sermant Metric Service, ensure that the following conditions are met:

- Sermant is correctly integrated into the application.
- The Prometheus monitoring tool is installed and running.

#### Configuring the Sermant Metric Service

##### Enabling the Metric Service

1. Open the Sermant configuration file located at `agent/config/config.properties`.
2. Set the following configuration items to enable the Metric Service:

    ```properties
    # Enable HTTP service
    agent.service.httpserver.enable=true
    # Enable Metric service
    agent.service.metric.enable=true
    ```

##### Configuring Prometheus

1. Open the Prometheus configuration file `prometheus.yml`.
2. Add the following configuration to scrape the metrics endpoint exposed by Sermant:

    ```yaml
    scrape_configs:
    - job_name: 'sermant-metrics'
        metrics_path: '/sermant/metrics'
        static_configs:
        - targets: ['<Sermant-Host>:47128']
    ```

    > Replace `<Sermant-Host>` with the actual host address where Sermant is running.

#### Using the Sermant Metric Service

##### Creating Custom Metrics

If you need to create custom metrics in your plugin, please refer to: [Metric Function](../developer-guide//metric-func.md).

#### Viewing Metrics

1. Start the Prometheus service.
2. Access the Prometheus web interface, typically at `http://<Prometheus-Host>:9090`.
3. In the Prometheus query interface, enter the metric name, such as `custom_counter_total`, to view the corresponding metric data.

## Configuration Specifications

The properties configuration files of the Sermant project and the YAML configuration files of each plugin support the following parameter configuration methods. For example, in the configuration file, `gateway.nettyIp=127.0.0.1`:

1. Directly modify the configuration file by changing `gateway.nettyIp=127.0.0.1` in the file.
2. Configure it via the -D parameter when starting the microservice, i.e., `-Dgateway.nettyIp=127.0.0.1`.
3. Configure it via an environment variable by adding `gateway.nettyIp=127.0.0.1` to the environment variables.
4. Configure it via Sermant Agent startup parameters, i.e., `-javaagent:sermant-agent.jar=gateway.nettyIp=127.0.0.1`.

The four methods listed above, arranged in order of priority from highest to lowest, are: 4 > 3 > 2 > 1.

Among these, the values for the last three parameter configuration methods support multiple formats. For example, for `gateway.nettyIp=127.0.0.1` in the configuration file, the following formats are all recognizable:

```properties
gateway.nettyIp=127.0.0.1
gateway_nettyIp=127.0.0.1
gateway-nettyIp=127.0.0.1
GATEWAY.NETTYIP=127.0.0.1
GATEWAY_NETTYIP=127.0.0.1
GATEWAY-NETTYIP=127.0.0.1
gateway.nettyip=127.0.0.1
gateway_nettyip=127.0.0.1
gateway-nettyip=127.0.0.1
gateway.netty.ip=127.0.0.1
gateway_netty_ip=127.0.0.1
gateway-netty-ip=127.0.0.1
GATEWAY.NETTY.IP=127.0.0.1
GATEWAY_NETTY_IP=127.0.0.1
GATEWAY-NETTY-IP=127.0.0.1
```

The Sermant Agent will sequentially check each configuration value from top to bottom to see if it is configured via startup parameters, environment variables, or -D parameters.

> **Note:** When modifying configurations through container scenarios using `env`, replace point (.) with underscore (_).
>
> Reason: Some OS images cannot recognize `env` variables with point.

For example：To modify the configuration `gateway.nettyIp=127.0.0.1` through the pod's `env`, use:

``` yaml
  env:
  - name: "gateway_nettyIp"
    value: "127.0.0.2"
```
