<div align="center">
  <br/>
  <a href="https://purpleteam-labs.com" title="purpleteam">
    <img width=900px src="https://gitlab.com/purpleteam-labs/purpleteam/raw/master/assets/images/purpleteam-banner.png" alt="purpleteam logo">
  </a>
  <br/>
<br/>
<h2>purpleteam Stage Two containers</h2><br/>
  
<br/><br/>


<br/><br/><br/>
</div>

These containers are started dynamically based on Build User config (Job) input supplied to the purpleteam CLI, specifically the number of `testSession`s you define.

The following configurations are relevant if you are intending on running the purpleteam back-end in the `local` environment. In the `cloud` this is all done for you.

Clone this repository.

# Define the environment variables

## app-slave (Zap)

We use a .env file directly in the app-slave directory for testing.

**`ZAP_API_KEY`**

Make sure you have assigned a value to the `ZAP_API_KEY` environment variable. 

The `ZAP_API_KEY` can be what ever you chose, just make sure that as well as defining it for app-slave, you also add it to the app-scanner project configuration. The app-scanner project requires the Zap API Key to be configured in order to authenticate to Zap running in the Stage Two container. For the app-scanner project, this needs to be set in the following:  
`{ "slave": { "apiKey": <zap-api-key-here> } }`

**`HOST_ZAP_LOG4J_PROPERTIES_PATH`** and **`ZAP_LOG4J_PROPERTIES_PATH_MOUNT_TARGET`**

If/when you need Zap debug logs you will also need to add the environment variables for the LOG4J debug configuration.

**.env** file

If you choose to use a .env file, adding all of these environment variables would look similar to the following:

```env
ZAP_API_KEY=<zap-api-key-here>
HOST_ZAP_LOG4J_PROPERTIES_PATH=<absolute-path-to/purpleteam-s2-containers/app-slave/log4j.properties>
ZAP_LOG4J_PROPERTIES_PATH_MOUNT_TARGET=/home/zap/.ZAP/log4j.properties
```

# Debugging

## app-slave (Zap)

### Debug logging

Providing you have established the environment variables discussed above:

In order to turn on debug logging for the app-slave (Zap) containers that run in the `local` environment, uncomment the `volumes` array and the element containing `source` key with environment variable `HOST_ZAP_LOG4J_PROPERTIES_PATH`.

Details [below](#redirecting-and-viewing-container-logs) for actually viewing the logs.

### Interacting with Zap

You can interact with Zap (query the Zap UI) while your tests are running. We've found this useful in the past to check the state of Zap while debugging the app-scanner.

1. Confirm that the appslave_zap_[n] container is running with:  
   ```shell
   docker stats
   ```
2. Check which host port the appslave_zap_[n] container is bound to with:  
   ```shell
   docker container ls
   ```
   This could be any port between `8080-8091` inclusive, as defined in the docker-compose.yml.  
   The port that Zap inside the container is listening on will always be 8080
3. In your browser using FoxyProxy set-up a proxy to localhost:[zap-host-port]  
   ![FoxyProxyDetails_Zap-min](/uploads/32b06156241706d0637afb19c0f6a211/FoxyProxyDetails_Zap-min.png)
   Optional: set-up the following URL Pattern in FoxyProxy: `zap:8080/*`  
   ![FoxyProxyURLPattern_Zap-min](/uploads/7446c101571199ca04e979518fd10e38/FoxyProxyURLPattern_Zap-min.png)
   If you use the URL Pattern it will allow you to leave FoxyProxy on, selecting "_Use proxies based on their predefined patterns and priorities_". Failing that you can just select the specific proxy you have created
4. Browse to `http://zap:8080/`  
   Your requests will be proxied through your host port and responded to via the `zap` process in the container specified by host port in your FoxyProxy configuration

## selenium-standalone

### Debug logging

In order to [turn on debug logging](https://github.com/SeleniumHQ/docker-selenium#se_opts-selenium-configuration-options) for the Selenium containers that run in the `local` environment, uncomment the `environment` array and the `SE_OPTS=-debug` element for `chrome` and/or `firefox`.

Details [below](#redirecting-and-viewing-container-logs) for actually viewing the logs.

### Viewing browser in Selenium container

The following outlines what you will need to do in order to view the browser inside any of the Selenium containers run from this project.

#### docker-compose.yml set-up:

* Swap the container images with the `-debug` images. The `-debug` images may be commented out, so simply commenting out the usual image and uncommenting the image with `-debug` appended to the end should do the trick
* Make sure you can access the VNC server within the container by uncommenting the `5900` port range. By specifying a range as the external port (Ex: `5900-5901`) you will be able to VNC into more than one container at once, in the example we have allowed for opening two sessions concurrently. If you need to VNC into more than two simply widen the external port range

#### VNC Client set-up:

1. You will need a VNC client to open a connection to the VNC server within the container. We've had success with using the Remmina Remote Desktop Cleint on Linux Mint. Install Remmina-plugin-vnc via the Software Manager
2. Run your Remmina Remote Desktop Client
3. Create new entries. If you intend to VNC into a couple of Selenium containers concurrently, you could set each one up like the following:  
   
   | Key  | Value                       |
   |------|-----------------------------|
   | Name | seleniumstandalone_chrome_1 |
   | Protocol | VNC - Virtual Network Computing |
   | **In the Basic tab** |  |
   | Server | 127.0.0.1:5900 |
   | Password | secret |
   | Color depth | True color (24 bit) # This was the only one that worked for us |
   | Quality | Poor (fastest) |
   
   | Key  | Value                       |
   |------|-----------------------------|
   | Name | seleniumstandalone_chrome_2 |
   | Protocol | VNC - Virtual Network Computing |
   | **In the Basic tab** |  |
   | Server | 127.0.0.1:5901 |
   | Password | secret |
   | Color depth | True color (24 bit) # This was the only one that worked for us |
   | Quality | Poor (fastest) |

Further details on the [SeleniumHQ github](https://github.com/SeleniumHQ/docker-selenium#debugging)

#### Viewing the browser within Selenium container

Once the Selenium containers are running (Keeping `docker stats` running in a terminal is convenient for viewing this), you can confirm which Selenium container is using which external port with `docker container ls`, as Docker has no idea which ports you have assigned to which of your VNC client entries, so the name of a given VNC client entry may not necessarily match the Selenium container with the same name. For this reason it is a good idea to confirm the port mappings with `docker container ls`.

In order to correlate which Selenium container is being used for which Test Session when you have multiple Test Sessions you may need to review the running app-scanner log. You may also need to make sure that the app-scanner is configured to log level `debug` in order to see some or more of the following log messages:  

`[app.parallel] cucCli process with PID "28" has been spawned for test session with Id "lowPrivUser"`

`[app.parallel] cucCli process with PID "34" has been spawned for test session with Id "adminUser"`

`[pid-28,world] seleniumContainerName is: seleniumstandalone_chrome_1`

`[pid-34,world] seleniumContainerName is: seleniumstandalone_chrome_2`

In this example we have two Test Sessions configured in our Build User config (Job). One has an `id` of `lowPrivUser` and one has an `id` of `adminUser`. In this example the `lowPrivUser` Test Session has a process with `PID` `28` and the `adminUser` Test Session has a process with `PID` `34`.  
In the next two log messages after that we see by correlating the PIDs that the `lowPrivUser` Test Session is running a container named `seleniumstandalone_chrome_1` and `adminUser` Test Session is running a container named `seleniumstandalone_chrome_2`.  
There are no guarantees as to which Test Session will run which of the seleniumstandalone_chrome_[n] containers, so if you need to be sure then use this correlation technique.

To VNC to the Selenium containers, once you have Remmina running, simply double click on one or more of the VNC entries you created above and you should be able to see the browser being interacted with... providing the Cucumber test steps in the app-scanner are actually up to that point.  
You can of course slow your tests down, pause them, step through them with a debugger. These details are in the purpleteam wiki in the workflow page

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




