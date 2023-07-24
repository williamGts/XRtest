# Exporter & Prometheus & Grafana setup

# Exporters(On Edge)

- Nvidia Gpu Exporter(Default port: 9835)
    - run the `nvidia_gpu_exporter.exe`
- ping_exporter
    
    [prometheus的网络ping监控exporter_姚__的博客-CSDN博客](https://blog.csdn.net/qq_32969313/article/details/124878153)
    
    - run command: `main.exe -port 8888 -pingaddr yourIP -count 4`
    - the latency: `ping_avg_time` → `avg_over_time(ping_avg_time{job="?"}[5m])`
- windows_exporter
    - download and run the `windows_exporter.exe`
    - visit localhost:9182
    
    ### PromQL commands:
    
    - throughput:
        
        <aside>
        💡 `rate(windows_net_bytes_received_total{job="?"}[5m])+rate(windows_net_bytes_sent_total{job="?"}[5m])`
        
        </aside>
        
    - packet loss:
        
        <aside>
        💡 `rate(windows_net_packets_outbound_errors_total{job="?"}[5m]) + rate(windows_net_packets_received_errors_total{job="?"}[5m])/ rate(windows_net_packets_total[5m])`
        
        </aside>
        

# Prometheus(On linux)

- `docker run -p 9090:9090 prom/prometheus`
    
    ### modify the config & the target & getting IP
    
    - open the prometheus.yml
    - get the address of the exporters
    - follow the format to add the exporters:
    - to check the internet throughput
    
    <aside>
    💡 We need the ip address of the edge and the port number to have the exporter connected.
    
    </aside>
    
    ```yaml
      - job_name: "prometheus"
        static_configs:
          - targets: ["localhost:9090"]
          
      - job_name: NVIDIA
        static_configs:
        
          - targets: ['192.168.4.167:9835'] #we need the ip address of the edge and the port number to have the exporter.
    ```
    
- back to the [localhost:9090](http://localhost:9090) we should be able to see the exporter is up
    
    ![Screenshot 2023-07-19 at 1.11.54 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/845865e2-1621-40d9-9c10-ec56b24ce9ff/Screenshot_2023-07-19_at_1.11.54_PM.png)
    
- add different items? → modify the prometheus config.
- i can modify the legend part to change the displayed name

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
      
  - job_name: NVIDIA
    static_configs:
    
      - targets: ['192.168.4.167:9835']

  - job_name: PING
    static_configs:
    
      - targets: ['192.168.4.167:8888']

  - job_name: WIN
    static_configs:
    
      - targets: ['192.168.4.167:9182']

  - job_name: iPhone
    static_configs:
    
      - targets: ['192.168.4.167:9182']
```

---

# Grafana

- Open Docker Desktop
- In the Command line type command: `docker run -d --name=grafana -p 3000:3000 grafana/grafana`
    - A container and image will appear in docker.
- visit `“localhost:3000”`
    - type `admin`  for account and password.
    
    ### Add data source
    
    1. click the three stripe button → Connections →  Add new connection → Prometheus → create a prometheus data source
    2. prometheus server url: [`http://host.docker.internal:9090/`](http://host.docker.internal:9090/)  
    3. click `save & test`
    
    ### Export JSON
    
    - Share → JSON
    
    ### Import Json
    
    - on top of the page → import dashboard → upload the JSON file → select desired data source.
    
    ### JOB filter
    
    - add a label at the end ex: `{env="xxx",instance="[http://xxxx.com](http://xxxx.com/)",job="xxxx-job"}`

---

# Some key points

- Download the exporters and run on the edge.
- run prometheus on the central server by using docker.
- the job name of the prometheus file has to be fixed, and we need to modify the targets part.
- the JSON file is fixed
- has to be in the same wifi setting.
