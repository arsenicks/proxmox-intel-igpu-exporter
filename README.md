# intel-gpu-exporter

> [!IMPORTANT]
> Forked from onedr0p/intel-gpu-exporter, no credit to me, I made minimal modification to accept args and add a systemd service example.
> This is intended to run on proxmox node but should run anywhere as far as the requirements are there.

Get metrics from Intel iGPUs, this is intended to run on a proxmox host node as a systemd service. Pretty usefull if you host VM that use passthrough for HW acceleration for transcoding like Plex, Jellyfin, Ombi or even Frigate for object detection. This script will provide different statistics of your iGPU that you can import in prometheus to use with grafana or anything, really, use your imagination.

## Basic

Runs on port 9100 by default, default refresh is 10000ms. Use intel_gpu_top output as json and parse it to feed the correct format for prometheus scraping.

Tested on Proxmox VE 8.4.1

## Requirements
- Python 3 (3.11 already installed on pve 8.4.1)
- Python prometheus_client v0.21.1
- intel-gpu-tools v1.27.x installed

## Installation
As root:
```bash
# Install dependency
apt-get install python3-prometheus-client intel-gpu-tools
# Get python exporter and put it in /usr/local/bin/intel-gpu-exporter.py on your filesystem
curl -L -o /usr/local/bin/intel-gpu-exporter.py https://raw.githubusercontent.com/arsenicks/proxmox-intel-igpu-exporter/refs/heads/main/intel-gpu-exporter.py
# Get systemd service file and put it in /etc/systemd/system/intel_igpu_exporter.service on your filesystem
curl -L -o /etc/systemd/system/intel_igpu_exporter.service https://raw.githubusercontent.com/arsenicks/proxmox-intel-igpu-exporter/refs/heads/main/intel_igpu_exporter.service
# Reload systemd and start the service
systemctl daemon-reload && systemctl start intel_igpu_exporter.service
# Check if service is up
systemctl status intel_igpu_exporter.service
```
You can also check using a web browser if the web server is available and if the data is refreshing. 
Simply go to http://ip_pve_node:9100/metrics port 9100 is used by default, if you want to change it read the next section.


## Exporter configuration
If you need to configure the default port(9100) or the default refresh time(10s) you'll have to modify the systemd service file. 
To apply your change  refresh systemd and restart the service using `systemctl daemon-reload && systemctl restart intel_igpu_exporter.service`

## Prometheus config
Here's an example job configuration to add to your prometheus config file.

Verify that you are able to access the exporter web page first, and your prometheus need to be able to access your pve node on the configured port..
```
  - job_name: pve_igpu_metrics # adjust job_name to fit your setup
    honor_timestamps: true
    scrape_interval: 15s
    scrape_timeout: 10s
    scheme: http
    follow_redirects: true
    static_configs:
    - targets:
      - 10.1.1.5:9100 # ip:port of your pve node
```

## Metrics

```bash
# HELP igpu_engines_blitter_0_busy Blitter 0 busy utilisation %
# TYPE igpu_engines_blitter_0_busy gauge
igpu_engines_blitter_0_busy 0.0
# HELP igpu_engines_blitter_0_sema Blitter 0 sema utilisation %
# TYPE igpu_engines_blitter_0_sema gauge
igpu_engines_blitter_0_sema 0.0
# HELP igpu_engines_blitter_0_wait Blitter 0 wait utilisation %
# TYPE igpu_engines_blitter_0_wait gauge
igpu_engines_blitter_0_wait 0.0
# HELP igpu_engines_render_3d_0_busy Render 3D 0 busy utilisation %
# TYPE igpu_engines_render_3d_0_busy gauge
igpu_engines_render_3d_0_busy 0.0
# HELP igpu_engines_render_3d_0_sema Render 3D 0 sema utilisation %
# TYPE igpu_engines_render_3d_0_sema gauge
igpu_engines_render_3d_0_sema 0.0
# HELP igpu_engines_render_3d_0_wait Render 3D 0 wait utilisation %
# TYPE igpu_engines_render_3d_0_wait gauge
igpu_engines_render_3d_0_wait 0.0
# HELP igpu_engines_video_0_busy Video 0 busy utilisation %
# TYPE igpu_engines_video_0_busy gauge
igpu_engines_video_0_busy 0.0
# HELP igpu_engines_video_0_sema Video 0 sema utilisation %
# TYPE igpu_engines_video_0_sema gauge
igpu_engines_video_0_sema 0.0
# HELP igpu_engines_video_0_wait Video 0 wait utilisation %
# TYPE igpu_engines_video_0_wait gauge
igpu_engines_video_0_wait 0.0
# HELP igpu_engines_video_enhance_0_busy Video Enhance 0 busy utilisation %
# TYPE igpu_engines_video_enhance_0_busy gauge
igpu_engines_video_enhance_0_busy 0.0
# HELP igpu_engines_video_enhance_0_sema Video Enhance 0 sema utilisation %
# TYPE igpu_engines_video_enhance_0_sema gauge
igpu_engines_video_enhance_0_sema 0.0
# HELP igpu_engines_video_enhance_0_wait Video Enhance 0 wait utilisation %
# TYPE igpu_engines_video_enhance_0_wait gauge
igpu_engines_video_enhance_0_wait 0.0
# HELP igpu_frequency_actual Frequency actual MHz
# TYPE igpu_frequency_actual gauge
igpu_frequency_actual 0.0
# HELP igpu_frequency_requested Frequency requested MHz
# TYPE igpu_frequency_requested gauge
igpu_frequency_requested 0.0
# HELP igpu_imc_bandwidth_reads IMC reads MiB/s
# TYPE igpu_imc_bandwidth_reads gauge
igpu_imc_bandwidth_reads 733.353818
# HELP igpu_imc_bandwidth_writes IMC writes MiB/s
# TYPE igpu_imc_bandwidth_writes gauge
igpu_imc_bandwidth_writes 166.044782
# HELP igpu_interrupts Interrupts/s
# TYPE igpu_interrupts gauge
igpu_interrupts 0.0
# HELP igpu_period Period ms
# TYPE igpu_period gauge
igpu_period 5000.241296
# HELP igpu_power_gpu GPU power W
# TYPE igpu_power_gpu gauge
igpu_power_gpu 0.0
# HELP igpu_power_package Package power W
# TYPE igpu_power_package gauge
igpu_power_package 5.480595
# HELP igpu_rc6 RC6 %
# TYPE igpu_rc6 gauge
igpu_rc6 99.999993
```