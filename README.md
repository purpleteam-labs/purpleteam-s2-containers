<div align="center">
  <br/>
  <a href="https://purpleteam-labs.com" title="purpleteam">
    <img width=900px src="https://github.com/purpleteam-labs/purpleteam/blob/main/assets/images/purpleteam-banner.png" alt="purpleteam logo">
  </a>
  <br/>
<br/>
<h2>purpleteam stage two containers</h2><br/>
Stage two containers of <a href="https://purpleteam-labs.com/" title="purpleteam"><em>PurpleTeam</em></a>
<br/><br/>

  <a href="https://purpleteam-labs.com/doc/" title="documentation">
    <img src="https://img.shields.io/badge/-documentation-blueviolet" alt="documentation">
  </a>

  <a href="https://github.com/purpleteam-labs/purpleteam-s2-containers/releases" title="latest release">
    <img src="https://img.shields.io/github/v/release/purpleteam-labs/purpleteam-s2-containers?color=%23794fb8&include_prereleases" alt="GitHub release (latest SemVer including pre-releases)">
  </a>
  <br/><br/>
</div>

These containers are started dynamically based on ([_Job_](https://purpleteam-labs.com/doc/definitions/)) input supplied to the _PurpleTeam_ CLI, specifically the number of `appScanner` resource objects you define.

The following configurations are relevant if you are intending on running the _PurpleTeam_ back-end in the `local` environment. In the `cloud` this is all done for you.

Clone this repository.

# Define the environment variables

## app-emissary (Zap)

We use a .env file directly in the app-emissary directory for testing. We have created a .env.example file in the app-emissary/ directory. Rename this to .env and set any values within appropriately.

### `ZAP_API_KEY`

Make sure you have assigned a value to the `ZAP_API_KEY` environment variable. 

The `ZAP_API_KEY` can be what ever you chose, just make sure that as well as defining it for the app-emissary, you also add it to the app-scanner project configuration. The app-scanner project requires the Zap API Key to be configured in order to authenticate to Zap running in the Stage Two container. For the app-scanner project, this needs to be set in the following way in the configuration:  
`{ "emissary": { "apiKey": <zap-api-key-here> } }`

### `HOST_DIR_APP_SCANNER`

This environment variable along with the directory it refers to should have been set-up for the App _Tester_ as part of the [_orchestrator_ set-up](https://purpleteam-labs.com/doc/local/set-up/#orchestrator). Set this environment variable to the same value. This is the directory that the App _Tester_ puts ephemeral files for the app-emissary to consume.

### `ZAP_DIR_APP_SCANNER_MOUNT_TARGET`

This environment variable refers to the target directory (from the above host directory) mounted into the app-emissary container

### `HOST_ZAP_LOG4J_PROPERTIES_PATH` and `ZAP_LOG4J_PROPERTIES_PATH_MOUNT_TARGET`

If/when you need Zap debug logs you will also need to make sure the environment variables for the LOG4J debug configuration is added to the .env file.

# Debugging

## app-emissary (Zap)

### Debug logging

Providing you have established the environment variables discussed above:

In order to turn on debug logging for the app-emissary (Zap) containers that run in the `local` environment, uncomment the `volumes` array and the following elements containing:

* `source` key with environment variable `HOST_ZAP_LOG4J_PROPERTIES_PATH`
* `target` key with environment variable `ZAP_LOG4J_PROPERTIES_PATH_MOUNT_TARGET`

The PurpleTeam-Labs team have a `debug` branch that we rebase on `main` when ever we want to apply these settings (enable debugging).

Details [below](#redirecting-and-viewing-container-logs) for actually viewing the logs.

### Interacting with Zap

You can interact with Zap (query the Zap API in your browser) while your tests are running. We've found this useful in the past to check the state of Zap while debugging the app-scanner.

1. Confirm that the appemissary_zap_[n] container is running with:  
   ```shell
   docker stats
   ```
2. Check which host port the appemissary_zap_[n] container is bound to with:  
   ```shell
   docker container ls
   ```
   This could be any port between `8080-8091` inclusive, as defined in the docker-compose.yml.  
   The port that Zap inside the container is listening on will always be 8080
3. In your browser using FoxyProxy set-up a proxy to localhost:[zap-host-port]  
   ![FoxyProxyDetails_Zap-min](https://user-images.githubusercontent.com/2862029/104170682-69998c00-5466-11eb-97b7-b4c2f3e597e9.png)  
   Optional: set-up the following URL Pattern in FoxyProxy: `zap:8080/*`  
   ![FoxyProxyURLPattern_Zap-min](https://user-images.githubusercontent.com/2862029/104170839-9e0d4800-5466-11eb-853c-6acad0ba52ea.png)  
   If you use the URL Pattern it will allow you to leave FoxyProxy on, selecting "_Use proxies based on their predefined patterns and priorities_". Failing that you can just select the specific proxy you have created
4. Browse to `http://zap:8080/`  
   Your requests will be proxied through your host port and responded to via the `zap` process in the container specified by host port in your FoxyProxy configuration

## selenium-standalone

### Debug logging

In order to [turn on debug logging](https://github.com/SeleniumHQ/docker-selenium#troubleshooting) for the Selenium containers that run in the `local` environment, edit the selenium-standalone/docker-compose.yml file, uncomment the `environment` array and the `SE_OPTS=--log-level FINE` element for `chrome` and/or `firefox`.

The PurpleTeam-Labs team use the same `debug` branch mentioned above that we rebase on `main` when ever we want to apply these settings (enable debugging).

Details [below](#redirecting-and-viewing-container-logs) for actually viewing the logs.

### Viewing browser in Selenium container

The following outlines what you will need to do in order to view the browser inside any of the Selenium containers run from this project. Details partly sourced from the [docker-selenium](https://github.com/SeleniumHQ/docker-selenium#using-a-vnc-client) Github repository.

#### docker-compose.yml set-up:

Make sure you can access the VNC server within the container by uncommenting the `5900` port range. By specifying a range as the external port (Ex: `5900-5901`) you will be able to VNC into more than one container at once, in the example we've made in the selenium-standalone/docker-compose.yml file, we have allowed for opening two sessions concurrently. If you need to VNC into more than two simply widen the external port range. The `debug` branch also takes care of this

#### VNC Client set-up:

1. You will need a VNC client to open a connection to the VNC server within the container. We've had success with using the Remmina Remote Desktop Client on Linux Mint. Install Remmina-plugin-vnc via Synaptic Package Manager or apt
2. Run your Remmina Remote Desktop Client
3. Create new entries. If you intend to VNC into a couple of Selenium containers concurrently, you could set each one up like the following:  
   
   | Key  | Value                       |
   |------|-----------------------------|
   | Name | seleniumstandalone_chrome_1 |
   | Protocol | VNC - Virtual Network Computing |
   | **In the Basic tab** |  |
   | Server | 127.0.0.1:5900 |
   | Password | secret |
   | Color depth | True color (32 bpp) # Others may also work |
   | Quality | Poor (fastest) |
   
   | Key  | Value                       |
   |------|-----------------------------|
   | Name | seleniumstandalone_chrome_2 |
   | Protocol | VNC - Virtual Network Computing |
   | **In the Basic tab** |  |
   | Server | 127.0.0.1:5901 |
   | Password | secret |
   | Color depth | True color (32 bpp) # Others may also work |
   | Quality | Poor (fastest) |

#### Viewing the browser within Selenium container

Once the Selenium containers are running (Keeping `docker stats` running in a terminal is convenient for viewing this), you can confirm which Selenium container is using which external port with `docker container ls`, as Docker has no idea which ports you have assigned to which of your VNC client entries, so the name of a given VNC client entry may not necessarily match the Selenium container with the same name. For this reason it is a good idea to confirm the port mappings with `docker container ls`.

In order to correlate which Selenium container is being used for which `appScanner` _Test Session_ when you have multiple `appScanner` _Test Sessions_ you may need to review the running app-scanner log. You may also need to make sure that the app-scanner is configured to log level `debug` in order to see some or more of the following log messages:  

`[app.parallel] cucCli process with PID "28" has been spawned for Test Session with Id "lowPrivUser"`

`[app.parallel] cucCli process with PID "34" has been spawned for Test Session with Id "adminUser"`

`[pid-28,world] seleniumContainerName is: seleniumstandalone_chrome_1`

`[pid-34,world] seleniumContainerName is: seleniumstandalone_chrome_2`

In this example we have two `appScanner` _Test Sessions_ configured in our _[Job](https://github.com/purpleteam-labs/purpleteam/blob/d08581dadf29ebd0becd1623a408335f2e72e15e/testResources/jobs/job_0.1.0-alpha.1_local)_. One has an `id` of `lowPrivUser` and one has an `id` of `adminUser`. In this example the `lowPrivUser` _Test Session_ has a process with `PID` `28` and the `adminUser` _Test Session_ has a process with `PID` `34`.  
In the next two log messages after that we see by correlating the `PID`s that the `lowPrivUser` _Test Session_ is running a container named `seleniumstandalone_chrome_1` and `adminUser` _Test Session_ is running a container named `seleniumstandalone_chrome_2`.  
There are no guarantees as to which `appScanner` _Test Session_ will run which of the seleniumstandalone_chrome_[n] containers, so if you need to be sure then use this correlation technique.

To VNC to the Selenium containers, once you have Remmina running, simply double click on one or more of the VNC entries you created above and you should be able to see the browser being interacted with... providing the Cucumber test steps in the app-scanner are actually up to that point.  
You can of course slow your tests down, pause them, step through them with a debugger. These details are in the _PurpleTeam_ [documentation](https://purpleteam-labs.com/doc/local/workflow/#app-scanner-and-sub-processes).

## Redirecting and viewing container logs

`local`ly you can keep `docker stats` running if you want so you can see which containers are running, when they are being started and stopped. This will also give you their names for the following commands.

In order to view the Stage Two container logs, tail stdout (and stderr, as Docker merges stdout and stderr) of a container:  
```shell
docker logs --follow [container-name]
```

If you would like to send those logs to a file as well:  
```shell
docker logs --follow [container-name] |tee output.log$(date '+%Y-%m-%d_%T')
```

If you would like to send those logs to a file without viewing them via your terminal:  
```shell
docker logs --follow [container-name] > output.log$(date '+%Y-%m-%d_%T')
```

<br>

Once you have cloned and configured the environment for the stage two containers, head back to the [local setup](https://purpleteam-labs.com/doc/local/set-up/) documentation to continue setting up the other _PurpleTeam_ components.

