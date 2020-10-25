<div align="center">
  <br/>
  <a href="https://purpleteam-labs.com" title="purpleteam">
    <img width=900px src="https://gitlab.com/purpleteam-labs/purpleteam/raw/master/assets/images/purpleteam-banner.png" alt="purpleteam logo">
  </a>
  <br/>
<br/>
<h2>purpleteam stage two containers</h2><br/>
  Currently in heavy development
<br/><br/>


<br/><br/><br/>
</div>

These containers are started dynamically based on Build User config (Job) input, specifically the number of `testSession`s.

# Debugging

## app-slave





## selenium-standalone

In order to [turn on debug logging](https://github.com/SeleniumHQ/docker-selenium#se_opts-selenium-configuration-options) for the Selenium containers that run in the `local` environment, uncomment the `environment` array and the `SE_OPTS=-debug` element for `chrome` and/or `firefox`




## Redirecting and viewing container logs

`local`ly you can keep `docker stats` running if you want so you can see which containers are running, when they are being started and stopped. This will also give you their names for the following commands.

In order to view the stage two container logs, tail stdout (and stderr, as Docker merges stdout and stderr) of a container:  
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




