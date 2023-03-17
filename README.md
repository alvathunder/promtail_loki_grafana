## Promtail - Loki - Grafana

#### Start containers: 
```docker compose up -d```
Keep in mind that the Vector service is only needed if you're using a ARM-based CPU. I did this setup on a Raspberry Pi.

#### Make sure that all containers are running: 
 ```docker ps```
 
#### Check if all services are working by checking if metrics are being collected. Head over to your browser and type:
```<host-ip/localhost>:3100/metrics```
Port 3000 is the default Loki port.

#### Get started with Grafana. Head over you your browser and type:
```<host-ip/localhost>:3000```
Port 3000 is the default Grafana port.

#### You should now see a login page asking for username and password. 
 username: ```admin``` password: ```admin```

#### Configure Grafana and Loki
- Head over to the configuration wheel on Grafana to the left.
- Click "Add data source"
- Scroll down and choose "Loki"
- Change the URL of your endpoint under "HTTP" and "URL" to: ```loki:3100/``` and save. 

#### Query the logs
- Head over to the Grafana explore page by clicking the compass
- Query logs form your local machines ```varlogs``` by typing:
```{job="varlogs"}``` which was in our promtail configuration
- You can also serach for a specific term within the ```varlogs```like this: ```{job="varlogs"} |= `daemon` ```

#### To collect docker logs on a non-ARM system.
- The job for collecting docker logs is already defined in the promtail-config but to use this we have to first install a docker driver so that we can push the logs to Loki. Use the following command : ```docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions```
- Make sure the installation was successful by typing: ```docker plugin```
- After that you're going to have to configure the ```daemon.json```file. Just type: ```sudo nano /etc/docker/daemon.json```. This file might not exist on your system, which is fine because the command will create one if needed. 
- Paste the following in the ```daemon.json```file. 
``` 
{
    "log-driver": "loki",
    "log-opts": {
        "loki-url": "http://localhost:3100/loki/api/v1/push",
        "loki-batch-size": "400"
    }
} 
```
- After that just restart the docker daemon by typing: ```sudo systemctl restart docker```
- Keep in mind that only the newly created containers from the host will then send data to Loki. This means that you might have to recreate containers if you wish to collects logs from them.
- Head back to your Grafana and and the query page and type ```{container_name="<name>"}```
- You can ofcourse search for specific terms within the logs with the help of the same syntax as before. 

#### Collect docker logs on ARM-based systems
The Vector service will collect logs through the docker API and doesn't need the driver.
- Make sure that the Vector service is running.
- Create the following file ```/etc/vector/vector.toml``` and paste the content of the ```vector-config.toml``` that exists within the ```promtail-loki-grafana``` directory. 