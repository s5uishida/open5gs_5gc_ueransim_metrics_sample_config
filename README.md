# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Monitoring Metrics with Prometheus
This describes a very simple configuration that uses Open5GS and UERANSIM for monitoring the metrics with Prometheus.
The metrics are as of 2023.03.06. I think the new metrics will be added in the future.

---

<h2 id="conf_list">List of Sample Configurations</h2>

1. [One SGW-C/PGW-C, one SGW-U/PGW-U and one APN](https://github.com/s5uishida/open5gs_epc_srsran_sample_config)
2. [One SGW-C/PGW-C, Multiple SGW-Us/PGW-Us and APNs](https://github.com/s5uishida/open5gs_epc_oai_sample_config)
3. [One SMF, one UPF and one DNN](https://github.com/s5uishida/open5gs_5gc_srsran_sample_config)
4. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/open5gs_5gc_ueransim_sample_config)
5. [Select nearby UPF(PGW-U) according to the connected eNodeB](https://github.com/s5uishida/open5gs_epc_srsran_nearby_upf_sample_config)
6. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/open5gs_5gc_ueransim_nearby_upf_sample_config)
7. [Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)
8. [SCP Indirect communication Model C](https://github.com/s5uishida/open5gs_5gc_ueransim_scp_model_c_sample_config)
9. [VoLTE and SMS Configuration for docker_open5gs](https://github.com/s5uishida/docker_open5gs_volte_sms_config)
10. Monitoring Metrics with Prometheus (this article)
11. [Framed Routing](https://github.com/s5uishida/open5gs_5gc_ueransim_framed_routing_sample_config)
12. [VPP-UPF(PGW-U) with DPDK](https://github.com/s5uishida/open5gs_epc_srsran_vpp_upf_dpdk_sample_config)
13. [VPP-UPF with DPDK](https://github.com/s5uishida/open5gs_5gc_ueransim_vpp_upf_dpdk_sample_config)
---

<h2 id="misc">Miscellaneous Notes</h2>

- [Install MongoDB 6.0 and Open5GS WebUI](https://github.com/s5uishida/open5gs_install_mongodb6_webui)
- [Install MongoDB 4.4.18 on Ubuntu 20.04 for Raspberry Pi 4B](https://github.com/s5uishida/install_mongodb_on_ubuntu_for_rp4b)
- [A Note for 5G SUCI Profile A/B Scheme](https://github.com/s5uishida/note_5g_suci_profile_ab)
- [A Note for Changing Network Interface of UPF from TUN to TAP in Open5GS](https://github.com/s5uishida/change_from_tun_to_tap_in_open5gs)
---

<h2 id="toc">Table of Contents</h2>

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Additional changes in configuration files of Open5GS 5GC C-Plane and U-Plane](#changes_cp)
- [Build Open5GS for using Prometheus](#build)
- [Run Prometheus](#run_prometheus)
  - [Web Access to Prometheus Dashboard](#access_prometheus)
  - [Metrics of Open5GS AMF](#amf_metrics)
  - [Metrics of Open5GS PCF](#pcf_metrics)
  - [Metrics of Open5GS SMF](#smf_metrics)
  - [Metrics of Open5GS UPF](#upf_metrics)
- [Run Grafana](#run_grafana)
  - [Web Access to Grafana Dashboard](#access_grafana)
  - [Prometheus data source](#data_source)
  - [Example of setting visualization of metrics](#set_metric)
- [Changelog (summary)](#changelog)

---
<h2 id="overview">Overview of Open5GS 5GC Simulation Mobile Network</h2>

This is an additional setting example when monitoring the metrics with Prometheus for the following sample configuration.

- [Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)

This example configuration for monitoring metrics with Prometheus is shown in the figure below.
NFs without metrics are not drawn in the figure.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=900px></img>

The 5GC / UE / RAN used are as follows.
Also, I started Prometheus and Grafana using Docker.
- 5GC - Open5GS v2.6.1 (2023.03.10) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM

The IP address and port of the monitored NFs are as follows.
| NF | IP address | port | Prometheus job_name |
| --- | --- | --- | --- |
| AMF | 192.168.0.111 | 9090/tcp | open5gs-amfd |
| PCF | 192.168.0.111 | 9091/tcp | open5gs-pcfd |
| SMF1 | 192.168.0.112 | 9090/tcp | open5gs-smfd1 |
| SMF2 | 192.168.0.113 | 9090/tcp | open5gs-smfd2 |
| UPF1 | 192.168.0.114 | 9090/tcp | open5gs-upfd1 |
| UPF2 | 192.168.0.115 | 9090/tcp | open5gs-upfd2 |

The exposed IP address and port of Prometheus and Grafana-OSS are as follows.
| Server | IP address | port |
| --- | --- | --- |
| Prometheus |  192.168.0.111 | 9092/tcp |
| Grafana-OSS |  192.168.0.111 | 3001/tcp |

<h2 id="changes_cp">Additional changes in configuration files of Open5GS 5GC C-Plane and U-Plane</h2>

In this case, the following configuration is further changed for monitoring the metrics with Prometheus.

- [Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)

I will explain what to set additionally here.
When monitoring the metrics with Prometheus, change the following configuration files for each NF of C-Plane and U-Plane.
- `open5gs/install/etc/open5gs/amf.yaml`
```
...
metrics:
  - addr: 192.168.0.111
    port: 9090
```
- `open5gs/install/etc/open5gs/pcf.yaml`
```
...
metrics:
  - addr: 192.168.0.111
    port: 9091
```
- `open5gs/install/etc/open5gs/smf1.yaml`
```
...
metrics:
  - addr: 192.168.0.112
    port: 9090
```
- `open5gs/install/etc/open5gs/smf2.yaml`
```
...
metrics:
  - addr: 192.168.0.113
    port: 9090
```
- `open5gs/install/etc/open5gs/upf.yaml` for UPF1
```
...
metrics:
  - addr: 192.168.0.114
    port: 9090
```
- `open5gs/install/etc/open5gs/upf.yaml` for UPF2
```
...
metrics:
  - addr: 192.168.0.115
    port: 9090
```

<h2 id="build">Build Open5GS for using Prometheus</h2>

Open5GS sets Prometheus metrics by default. Please refer to the following for building Open5GS.
- https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/

Please install the following to run Prometheus on Docker.
- [docker-ce](https://docs.docker.com/install/linux/docker-ce/ubuntu)

<h2 id="run_prometheus">Run Prometheus</h2>

Please refer to the following for using Prometheus.
- https://open5gs.org/open5gs/docs/tutorial/04-metrics-prometheus/

<h3 id="access_prometheus">Web Access to Prometheus Dashboard</h3>

First, create the following `prometheus.yml`.
```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: open5gs-amfd
    static_configs:
      - targets: ["192.168.0.111:9090"]
  - job_name: open5gs-pcfd
    static_configs:
      - targets: ["192.168.0.111:9091"]
  - job_name: open5gs-smfd1
    static_configs:
      - targets: ["192.168.0.112:9090"]
  - job_name: open5gs-smfd2
    static_configs:
      - targets: ["192.168.0.113:9090"]
  - job_name: open5gs-upfd1
    static_configs:
      - targets: ["192.168.0.114:9090"]
  - job_name: open5gs-upfd2
    static_configs:
      - targets: ["192.168.0.115:9090"]
```
After starting Open5GS, run Prometheus as follows.
```
docker run -d -p 9092:9090 -v `pwd`/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```
You can access the following URL with web browser.
```
http://192.168.0.111:9092/
```

<img src="./images/prometheus-top.png" title="./images/prometheus-top.png" width=900px></img>

The list of targets is as follows.

<img src="./images/prometheus-targets.png" title="./images/prometheus-targets.png" width=900px></img>

<h3 id="amf_metrics">Metrics of Open5GS AMF</h3>

Following the Endpoint link of job_name=**open5gs-amfd**, the metrics will be displayed as follows.
```sh
# HELP gnb gNodeBs
# TYPE gnb gauge
gnb 0

# HELP fivegs_amffunction_mm_confupdate Number of UE Configuration Update commands requested by the AMF
# TYPE fivegs_amffunction_mm_confupdate counter
fivegs_amffunction_mm_confupdate 0

# HELP fivegs_amffunction_rm_reginitreq Number of initial registration requests received by the AMF
# TYPE fivegs_amffunction_rm_reginitreq counter
fivegs_amffunction_rm_reginitreq 0

# HELP fivegs_amffunction_rm_regemergreq Number of emergency registration requests received by the AMF
# TYPE fivegs_amffunction_rm_regemergreq counter
fivegs_amffunction_rm_regemergreq 0

# HELP fivegs_amffunction_mm_paging5greq Number of 5G paging procedures initiated at the AMF
# TYPE fivegs_amffunction_mm_paging5greq counter
fivegs_amffunction_mm_paging5greq 0

# HELP fivegs_amffunction_rm_regperiodreq Number of periodic registration update requests received by the AMF
# TYPE fivegs_amffunction_rm_regperiodreq counter
fivegs_amffunction_rm_regperiodreq 0

# HELP fivegs_amffunction_mm_confupdatesucc Number of UE Configuration Update complete messages received by the AMF
# TYPE fivegs_amffunction_mm_confupdatesucc counter
fivegs_amffunction_mm_confupdatesucc 0

# HELP fivegs_amffunction_rm_reginitsucc Number of successful initial registrations at the AMF
# TYPE fivegs_amffunction_rm_reginitsucc counter
fivegs_amffunction_rm_reginitsucc 0

# HELP fivegs_amffunction_amf_authreject Number of authentication rejections sent by the AMF
# TYPE fivegs_amffunction_amf_authreject counter
fivegs_amffunction_amf_authreject 0

# HELP fivegs_amffunction_rm_regmobreq Number of mobility registration update requests received by the AMF
# TYPE fivegs_amffunction_rm_regmobreq counter
fivegs_amffunction_rm_regmobreq 0

# HELP amf_session AMF Sessions
# TYPE amf_session gauge
amf_session 0

# HELP fivegs_amffunction_rm_regmobsucc Number of successful mobility registration updates at the AMF
# TYPE fivegs_amffunction_rm_regmobsucc counter
fivegs_amffunction_rm_regmobsucc 0

# HELP fivegs_amffunction_amf_authreq Number of authentication requests sent by the AMF
# TYPE fivegs_amffunction_amf_authreq counter
fivegs_amffunction_amf_authreq 0

# HELP fivegs_amffunction_rm_regemergsucc Number of successful emergency registrations at the AMF
# TYPE fivegs_amffunction_rm_regemergsucc counter
fivegs_amffunction_rm_regemergsucc 0

# HELP fivegs_amffunction_mm_paging5gsucc Number of successful 5G paging procedures initiated at the AMF
# TYPE fivegs_amffunction_mm_paging5gsucc counter
fivegs_amffunction_mm_paging5gsucc 0

# HELP ran_ue RAN UEs
# TYPE ran_ue gauge
ran_ue 0

# HELP fivegs_amffunction_rm_regperiodsucc Number of successful periodic registration update requests at the AMF
# TYPE fivegs_amffunction_rm_regperiodsucc counter
fivegs_amffunction_rm_regperiodsucc 0

# HELP fivegs_amffunction_rm_registeredsubnbr Number of registered state subscribers per AMF
# TYPE fivegs_amffunction_rm_registeredsubnbr gauge

# HELP fivegs_amffunction_rm_reginitfail Number of failed initial registrations at the AMF
# TYPE fivegs_amffunction_rm_reginitfail counter

# HELP fivegs_amffunction_rm_regmobfail Number of failed mobility registration updates at the AMF
# TYPE fivegs_amffunction_rm_regmobfail counter

# HELP fivegs_amffunction_rm_regperiodfail Number of failed periodic registration update requests at the AMF
# TYPE fivegs_amffunction_rm_regperiodfail counter

# HELP fivegs_amffunction_rm_regemergfail Number of failed emergency registrations at the AMF
# TYPE fivegs_amffunction_rm_regemergfail counter

# HELP fivegs_amffunction_amf_authfail Number of authentication failure messages received by the AMF
# TYPE fivegs_amffunction_amf_authfail counter

# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1024

# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes -1

# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total gauge
process_cpu_seconds_total 0

# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 159555584

# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 17182720

# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 61727

# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 23
```

<h3 id="pcf_metrics">Metrics of Open5GS PCF</h3>

Following the Endpoint link of job_name=**open5gs-pcfd**, the metrics will be displayed as follows.
```sh
# HELP fivegs_pcffunction_pa_policyamassoreq Number of AM policy association requests
# TYPE fivegs_pcffunction_pa_policyamassoreq counter

# HELP fivegs_pcffunction_pa_policyamassosucc Number of successful AM policy associations
# TYPE fivegs_pcffunction_pa_policyamassosucc counter

# HELP fivegs_pcffunction_pa_policysmassoreq Number of SM policy association requests
# TYPE fivegs_pcffunction_pa_policysmassoreq counter

# HELP fivegs_pcffunction_pa_policysmassosucc Number of successful SM policy associations
# TYPE fivegs_pcffunction_pa_policysmassosucc counter

# HELP fivegs_pcffunction_pa_sessionnbr Active Sessions
# TYPE fivegs_pcffunction_pa_sessionnbr gauge

# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1024

# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes -1

# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total gauge
process_cpu_seconds_total 0

# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 192069632

# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 18964480

# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 61727

# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 14
```

<h3 id="smf_metrics">Metrics of Open5GS SMF</h3>

Following the Endpoint link of job_name=**open5gs-smfd1**, the metrics will be displayed as follows.
```sh
# HELP gn_rx_createpdpcontextreq Received GTPv1C CreatePDPContextRequest messages
# TYPE gn_rx_createpdpcontextreq counter
gn_rx_createpdpcontextreq 0

# HELP gn_rx_deletepdpcontextreq Received GTPv1C DeletePDPContextRequest messages
# TYPE gn_rx_deletepdpcontextreq counter
gn_rx_deletepdpcontextreq 0

# HELP gtp1_pdpctxs_active Active GTPv1 PDP Contexts (GGSN)
# TYPE gtp1_pdpctxs_active gauge
gtp1_pdpctxs_active 0

# HELP fivegs_smffunction_sm_n4sessionreport Number of requested N4 session reports evidented by SMF
# TYPE fivegs_smffunction_sm_n4sessionreport counter
fivegs_smffunction_sm_n4sessionreport 0

# HELP ues_active Active User Equipments
# TYPE ues_active gauge
ues_active 0

# HELP gtp2_sessions_active Active GTPv2 Sessions (PGW)
# TYPE gtp2_sessions_active gauge
gtp2_sessions_active 0

# HELP gtp_node_gn_rx_parse_failed Received GTPv1C messages discarded due to parsing failure
# TYPE gtp_node_gn_rx_parse_failed counter

# HELP s5c_rx_createsession Received GTPv2C CreateSessionRequest messages
# TYPE s5c_rx_createsession counter
s5c_rx_createsession 0

# HELP s5c_rx_deletesession Received GTPv2C DeleteSessionRequest messages
# TYPE s5c_rx_deletesession counter
s5c_rx_deletesession 0

# HELP gtp_new_node_failed Unable to allocate new GTP (peer) Node
# TYPE gtp_new_node_failed counter
gtp_new_node_failed 0

# HELP s5c_rx_parse_failed Received GTPv2C messages discarded due to parsing failure
# TYPE s5c_rx_parse_failed counter
s5c_rx_parse_failed 0

# HELP fivegs_smffunction_sm_n4sessionreportsucc Number of successful N4 session reports evidented by SMF
# TYPE fivegs_smffunction_sm_n4sessionreportsucc counter
fivegs_smffunction_sm_n4sessionreportsucc 0

# HELP gtp_node_gn_rx_createpdpcontextreq Received GTPv1C CreatePDPContextRequest messages
# TYPE gtp_node_gn_rx_createpdpcontextreq counter

# HELP fivegs_smffunction_sm_n4sessionestabreq Number of requested N4 session establishments evidented by SMF
# TYPE fivegs_smffunction_sm_n4sessionestabreq counter
fivegs_smffunction_sm_n4sessionestabreq 0

# HELP bearers_active Active Bearers
# TYPE bearers_active gauge
bearers_active 0

# HELP gn_rx_parse_failed Received GTPv1C messages discarded due to parsing failure
# TYPE gn_rx_parse_failed counter
gn_rx_parse_failed 0

# HELP gtp_peers_active Active GTP peers
# TYPE gtp_peers_active gauge
gtp_peers_active 0

# HELP gtp_node_gn_rx_deletepdpcontextreq Received GTPv1C DeletePDPContextRequest messages
# TYPE gtp_node_gn_rx_deletepdpcontextreq counter

# HELP gtp_node_s5c_rx_parse_failed Received GTPv2C messages discarded due to parsing failure
# TYPE gtp_node_s5c_rx_parse_failed counter

# HELP gtp_node_s5c_rx_createsession Received GTPv2C CreateSessionRequest messages
# TYPE gtp_node_s5c_rx_createsession counter

# HELP gtp_node_s5c_rx_deletesession Received GTPv2C DeleteSessionRequest messages
# TYPE gtp_node_s5c_rx_deletesession counter

# HELP fivegs_smffunction_sm_sessionnbr Active Sessions
# TYPE fivegs_smffunction_sm_sessionnbr gauge

# HELP fivegs_smffunction_sm_qos_flow_nbr Number of QoS flows at the SMF
# TYPE fivegs_smffunction_sm_qos_flow_nbr gauge

# HELP fivegs_smffunction_sm_n4sessionestabfail Number of failed N4 session establishments evidented by SMF
# TYPE fivegs_smffunction_sm_n4sessionestabfail counter

# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1024

# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes -1

# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total gauge
process_cpu_seconds_total 0

# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 1180213248

# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 40980480

# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 61727

# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 20
```

<h3 id="upf_metrics">Metrics of Open5GS UPF</h3>

Following the Endpoint link of job_name=**open5gs-upfd1**, the metrics will be displayed as follows.
```sh
# HELP fivegs_ep_n3_gtp_indatapktn3upf Number of incoming GTP data packets on the N3 interface
# TYPE fivegs_ep_n3_gtp_indatapktn3upf counter
fivegs_ep_n3_gtp_indatapktn3upf 0

# HELP fivegs_ep_n3_gtp_outdatapktn3upf Number of outgoing GTP data packets on the N3 interface
# TYPE fivegs_ep_n3_gtp_outdatapktn3upf counter
fivegs_ep_n3_gtp_outdatapktn3upf 0

# HELP fivegs_upffunction_sm_n4sessionestabreq Number of requested N4 session establishments
# TYPE fivegs_upffunction_sm_n4sessionestabreq counter
fivegs_upffunction_sm_n4sessionestabreq 0

# HELP fivegs_upffunction_sm_n4sessionreport Number of requested N4 session reports
# TYPE fivegs_upffunction_sm_n4sessionreport counter
fivegs_upffunction_sm_n4sessionreport 0

# HELP fivegs_upffunction_sm_n4sessionreportsucc Number of successful N4 session reports
# TYPE fivegs_upffunction_sm_n4sessionreportsucc counter
fivegs_upffunction_sm_n4sessionreportsucc 0

# HELP fivegs_upffunction_upf_sessionnbr Active Sessions
# TYPE fivegs_upffunction_upf_sessionnbr gauge
fivegs_upffunction_upf_sessionnbr 0

# HELP fivegs_ep_n3_gtp_indatavolumeqosleveln3upf Data volume of incoming GTP data packets per QoS level on the N3 interface
# TYPE fivegs_ep_n3_gtp_indatavolumeqosleveln3upf counter

# HELP fivegs_ep_n3_gtp_outdatavolumeqosleveln3upf Data volume of outgoing GTP data packets per QoS level on the N3 interface
# TYPE fivegs_ep_n3_gtp_outdatavolumeqosleveln3upf counter

# HELP fivegs_upffunction_sm_n4sessionestabfail Number of failed N4 session establishments
# TYPE fivegs_upffunction_sm_n4sessionestabfail counter

# HELP fivegs_upffunction_upf_qosflows Number of QoS flows of UPF
# TYPE fivegs_upffunction_upf_qosflows gauge

# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1024

# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes -1

# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total gauge
process_cpu_seconds_total 0

# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 286445568

# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 25284608

# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 58513

# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 12
```

<h2 id="run_grafana">Run Grafana</h2>

I used the OSS version of Grafana.

<h3 id="access_grafana">Web Access to Grafana Dashboard</h3>

After starting Open5GS and Prometheus, run Grafana-OSS as follows.
```
docker run -d -p 3001:3000 grafana/grafana-oss
```
You can access the following URL with web browser.
The default username and password are both `admin`.
```
http://192.168.0.111:3001/
```

<h3 id="data_source">Prometheus data source</h3>

The data source name, URL, and access mode are as follows.
I used the defaults for other than these as is.
| Name | URL |
| --- | --- |
| Open5GS | `http://192.168.0.111:9092/` |

<img src="./images/grafana-ds.png" title="./images/grafana-ds.png" width=900px></img>

<h3 id="set_metric">Example of setting visualization of metrics</h3>

Create your first dashboard and a panel for each metric you want to visualize.
In the `Metrics browser`, select the job and its metrics to visualize from the three job_names, set panel properties, add panels, and create the dashboard.

The following is a simple example of setting the panel for the AMF metric `amf_session`.

<img src="./images/grafana-metric.png" title="./images/grafana-metric.png" width=900px></img>

Grafana allows you to freely design user-friendly dashboards by creating attractive panels for each metric and combining them.

---
I was able to confirm the very simple configuration that uses Open5GS and UERANSIM for monitoring the metrics with Prometheus.
Also, when using Open5GS as EPC, MME(`open5gs-mmed`) supports Prometheus metrics, so you can monitor these metrics by making similar settings.
I would like to thank the excellent developers and all the contributors of Open5GS, UERANSIM, Prometheus and Grafana.

<h2 id="changelog">Changelog (summary)</h2>

- [2023.03.06] Added more AMF metrics.
- [2022.12.11] Added PCF and UPF metrics.
- [2022.08.07] Initial release.
