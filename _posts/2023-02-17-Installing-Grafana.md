---
layout: post
title: Installling A basic Grafana Setup With Prometheues.
subtitle: Playing with Monitoring.
date: 2023-02-29 18:30:00 +0000
tags:
- monitoring
- Grafana

---

## Setting Up Prometheus, Node Exporter, and Grafana for System Monitoring


In this guide, we'll walk through the steps to set up Prometheus, Node Exporter, and Grafana for monitoring your system. This setup will allow you to collect and visualize metrics from your server, providing insights into system performance and health.

## Prerequisites

- A Linux server
- Basic command-line knowledge

## Step 1: Install Node Exporter

Node Exporter is a Prometheus exporter for hardware and OS metrics exposed by *nix kernels.

### Download and Start Node Exporter

1. **Download Node Exporter**:

    ```sh
    wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
    ```

2. **Extract the tarball**:

    ```sh
    tar xvfz node_exporter-1.6.0.linux-amd64.tar.gz
    cd node_exporter-1.6.0.linux-amd64
    ```

3. **Start Node Exporter**:

    ```sh
    ./node_exporter &
    ```

4. **Verify Node Exporter is running**:

    ```sh
    curl http://localhost:9100/metrics
    ```

## Step 2: Install Prometheus

Prometheus is a powerful monitoring system and time-series database.

### Download and Start Prometheus

1. **Download Prometheus**:

    ```sh
    wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
    ```

2. **Extract the tarball**:

    ```sh
    tar xvfz prometheus-2.43.0.linux-amd64.tar.gz
    cd prometheus-2.43.0.linux-amd64
    ```

3. **Create the Prometheus configuration file**:

    ```sh
    sudo nano /etc/prometheus/prometheus.yml
    ```

    Add the following content:

    ```yaml
    global:
      scrape_interval: 15s

    scrape_configs:
      - job_name: "myserver"
        static_configs:
          - targets: ["localhost:9100"]
    ```

4. **Start Prometheus**:

    ```sh
    ./prometheus --config.file=/etc/prometheus/prometheus.yml
    ```

5. **Verify Prometheus is running**:

    ```sh
    curl -I http://localhost:9090
    ```

## Step 3: Create a Systemd Service for Prometheus

1. **Create a Prometheus user**:

    ```sh
    sudo useradd --no-create-home --shell /bin/false prometheus
    ```

2. **Create necessary directories**:

    ```sh
    sudo mkdir /etc/prometheus
    sudo mkdir /var/lib/prometheus
    sudo chown prometheus:prometheus /etc/prometheus
    sudo chown prometheus:prometheus /var/lib/prometheus
    ```

3. **Move configuration and binaries**:

    ```sh
    sudo cp prometheus /usr/local/bin/
    sudo cp promtool /usr/local/bin/
    sudo cp -r consoles /etc/prometheus
    sudo cp -r console_libraries /etc/prometheus
    sudo cp prometheus.yml /etc/prometheus
    sudo chown prometheus:prometheus /usr/local/bin/prometheus
    sudo chown prometheus:prometheus /usr/local/bin/promtool
    sudo chown -R prometheus:prometheus /etc/prometheus/consoles
    sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
    sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
    ```

4. **Create a systemd service file**:

    ```sh
    sudo nano /etc/systemd/system/prometheus.service
    ```

    Add the following content:

    ```sh
    [Unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    ExecStart=/usr/local/bin/prometheus \
      --config.file=/etc/prometheus/prometheus.yml \
      --storage.tsdb.path=/var/lib/prometheus \
      --web.console.templates=/etc/prometheus/consoles \
      --web.console.libraries=/etc/prometheus/console_libraries
    ExecReload=/bin/kill -HUP $MAINPID
    TimeoutStopSec=20s
    SendSIGKILL=no
    LimitNOFILE=8192

    [Install]
    WantedBy=multi-user.target
    ```

5. **Reload systemd and start Prometheus**:

    ```sh
    sudo systemctl daemon-reload
    sudo systemctl start prometheus
    sudo systemctl enable prometheus
    ```

6. **Verify the Prometheus service**:

    ```sh
    sudo systemctl status prometheus
    ```

## Step 4: Install and Configure Grafana

Grafana is a powerful visualization and dashboarding tool.

### Install Grafana

1. **Download Grafana**:

    ```sh
    wget https://dl.grafana.com/oss/release/grafana_8.4.4_amd64.deb
    ```

2. **Install the package**:

    ```sh
    sudo dpkg -i grafana_8.4.4_amd64.deb
    ```

3. **Start Grafana**:

    ```sh
    sudo systemctl start grafana-server
    sudo systemctl enable grafana-server
    ```

4. **Verify Grafana is running**:

    Open your web browser and navigate to `http://localhost:3000`. The default login credentials are:
    - **Username:** admin
    - **Password:** admin

### Add Prometheus as a Data Source in Grafana

1. **Log in to Grafana**: Open `http://localhost:3000` in your web browser and log in.

2. **Navigate to Data Sources**:
    - Click on the gear icon (⚙) on the left sidebar to open the configuration menu.
    - Click on "Data Sources".

3. **Add a New Data Source**:
    - Click the "Add data source" button.
    - Select "Prometheus" from the list of data sources.

4. **Configure the Prometheus Data Source**:
    - **Name**: Enter a name, e.g., "Prometheus".
    - **URL**: Enter `http://localhost:9090`.
    - **Access**: Set to "Server" (default).
    - Click "Save & Test" to verify the connection.

### Create a Dashboard in Grafana

1. **Create a New Dashboard**:
    - Click the "+" icon on the left sidebar.
    - Select "Dashboard".
    - Click "Add new panel".

2. **Configure the Panel**:
    - In the "Query" section, select "Prometheus" as the data source.
    - Enter a Prometheus query, e.g., `node_cpu_seconds_total`.
    - Customize the visualization settings.

3. **Save the Dashboard**:
    - Click "Apply" to save the panel.
    - Click the "Save" icon at the top right to save the dashboard.

## Conclusion

By following these steps, you've set up a robust monitoring solution using Prometheus, Node Exporter, and Grafana. You can now visualize and analyze your system's performance and health metrics effectively. Happy monitoring!

