<div align="center">
  <br/>
  <a href="https://purpleteam-labs.com" title="purpleteam">
    <img width=900px src="https://gitlab.com/purpleteam-labs/purpleteam/raw/master/assets/images/purpleteam-banner.png" alt="purpleteam logo">
  </a>
  <br/>
<br/>
<h2>purpleteam Stage Two containers</h2><br/>
  Currently in heavy development
<br/><br/>


<br/><br/><br/>
</div>

These containers are started dynamically based on Build User config (Job) input, specifically the number of `testSession`s.

# Debugging

## app-slave

### Debug logging




Details [below]() for actually viewing the logs.

## selenium-standalone

### Debug logging

In order to [turn on debug logging](https://github.com/SeleniumHQ/docker-selenium#se_opts-selenium-configuration-options) for the Selenium containers that run in the `local` environment, uncomment the `environment` array and the `SE_OPTS=-debug` element for `chrome` and/or `firefox`.

Details [below]() for actually viewing the logs.

### Viewing browser in Selenium container

In order to view the browser inside the Selenium container, there are a few things you will need to do.

#### docker-compose.yml set-up:

* Swap the container images with the `-debug` images. The `-debug` images may be commented out, so simply commenting out the usual image and uncommenting the image with `-debug` appended to the end should do the trick
* Make sure you can access the VNC server within the container by uncommenting the `5900` port range. By specifying a range as the external port (Ex: `5900-5901`) you will be able to VNC into more than one container at once, in the example we have allowed for opening two sessions concurrently. If you need to VNC into more than two simply widen the external port range

#### VNC Client set-up:

You will need a VNC client to open a connection to the VNC server within the container. We've had success with using the Remmina Remote Desktop Cleint on Linux Mint. Install the Remmina-plugin-vnc via the Software Manager.

Run your Remmina Remote Desktop Client

Create new entries. If you intend to VNC into a couple of Selenium containers concurrently, you could set each one up like the following:

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

Once the Selenium containers are running (Keeping `docker stats` running in a terminal is convenient for this), you can confirm which Selenium container is using which external port with `docker container ls`, as Docker has no idea which ports you have assigned to which of your VNC client entries, so the name of a given VNC client entry may not necessarily match the Selenium container with the same name. For this reason it is a good idea to confirm the port mappings with `docker container ls`.

In order to correlate which Selenium container is being used for which Test Session you may need to review the running app-scanner log. You may also need to make sure that the app-scanner is configured to log level `debug` in order to see some or more of the following log messages:  

`[app.parallel] cucCli process with PID "28" has been spawned for test session with Id "lowPrivUser"`

`[app.parallel] cucCli process with PID "34" has been spawned for test session with Id "adminUser"`

`[pid-28,world] seleniumContainerName is: seleniumstandalone_chrome_1`

`[pid-34,world] seleniumContainerName is: seleniumstandalone_chrome_2`

In this example we have two Test Sessions configured in our Build User config (Job). One has an `id` of `lowPrivUser` and one has an `id` of `adminUser`. In this example the `lowPrivUser` Test Session has a process with `PID` `28` and the `adminUser` Test Session has a process with `PID` `34`.  
In the next two log messages after that we see by correlating the PIDs that the `lowPrivUser` Test Session is running a container named `seleniumstandalone_chrome_1` and `adminUser` Test Session is running a container named `seleniumstandalone_chrome_2`.  
There are no guarantees as to which Test Session will run which of the seleniumstandalone_chrome_[n] containers, so if you need to be sure then use this correlation technique  

To VNC to the Selenium containers, once you have Remmina running, simply double click on one or more of the VNC entries you created above and you should be able to see the browser being interacted with... providing the Cucumber test steps in the app-scanner are actually up to that point.

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




