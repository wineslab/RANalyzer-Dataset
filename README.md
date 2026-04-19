# RANalyze-Dataset

This dataset contains end-to-end 5G performance measurements collected through automated CI/CD/CT pipelines. Each entry captures iPerf3 downlink UDP results and OpenAirInterface (OAI) gNB logs from an over-the-air test triggered by a new OAI software release. The dataset spans 69 OAI releases and more than 9,400 automated test runs collected from July 2023 to December 2025.

This dataset accompanies the following paper:

> R. Shirkhani, R. Prasad, L. Bonati, T. Melodia, and M. Polese, "RANalyzer: Automated Continuous RAN Software Evaluation and Regression Analysis," in *Proc. of IEEE International Conference on Network Softwarization (NetSoft)*, Berlin, Germany, July 2026.

## Citation

If you use this dataset, please cite:

```bibtex
@inproceedings{ranalyzer2026,
  author    = {Shirkhani, Ravis and Prasad, Reshma and Bonati, Leonardo and Melodia, Tommaso and Polese, Michele},
  title     = {{RANalyzer}: Automated Continuous {RAN} Software Evaluation and Regression Analysis},
  booktitle = {Proc. of IEEE International Conference on Network Softwarization (NetSoft)},
  year      = {2026},
  note      = {To appear}
}
```

---

## Experimental Setup

The dataset was collected on a heterogeneous private 5G network testbed. The testbed is deployed on an OpenShift cluster and integrates open-source and commercial components for a complete end-to-end 5G network.

**Compute and RAN.** The RAN is hosted on Microway servers with AMD EPYC 7262 processors.

**Radio and UEs.** RAN workloads interface with a grid of USRP X410 software-defined radios (SDRs) for over-the-air tests in an indoor environment. All tests operate on a 60 MHz channel at a carrier frequency of 3.629 GHz (band n78). 
User equipment consists of commercial Sierra Wireless 5G modems connected to small host compute nodes.

**Core network.** Open5GS is used as the 5G core network.

**CI/CD/CT pipeline.** Data collection is driven by the 5G-CT framework [1], which implements automated pipelines that build a new tag each week, deploy, and test the end-to-end network.
Deployment and testing on the OpenShift cluster follows the configuration described in [2]. This design allows the gNB software to be continuously updated while keeping test execution repeatable across software versions.

> [1] L. Bonati et al., "5G-CT: A Framework for Continuous Testing of 5G Networks," 2024.
> [2] M. Maxenti et al., "AutoRAN," 2025.

**Tests.** Each end-to-end test runs downlink UDP traffic using iPerf3 at multiple target data rates (e.g., 10, 20, 30 Mbps, and up to 80 Mbps). At the end of each test, gNB logs, iPerf3 results, and UE metadata are stored for further analysis.


---

## Overview

- **Time span:** November 2023 – December 2025
- **Total test runs:** 9,430
- **Traffic type:** UDP downlink iPerf3 tests at varying target bitrates
- **Stack:** OpenAirInterface gNB on OpenShift Container Platform


## Dataset Structure

```
cicd-dataset-ocp-arena/
└── YYYYMMDD/                         # Date of the test run
    └── HHMMSS/                       # Time of the test run (UTC)
        ├── YYYYMMDD_HHMMSS_<id>.json
        ├── YYYYMMDD_HHMMSS_<id>_udp_band<X>Mbps_time_stats.csv
        └──  nr-gnb.log
```

The `<id>` field in filenames is a unique numeric identifier assigned to each test run. `<X>` is the target bitrate in Mbps (e.g., `10`, `20`, `50`).


## File Descriptions

### `YYYYMMDD_HHMMSS_<id>.json`
Raw iPerf3 result in JSON format, captured from the UE/client side. Contains:
- Connection metadata (host, port, protocol version, timestamp)
- Per-interval throughput, jitter, packet loss, and byte counts
- Aggregate summary statistics for the full test duration
- CPU utilization on both host and remote endpoints

### `YYYYMMDD_HHMMSS_<id>_udp_band<X>Mbps_time_stats.csv`
Time-series breakdown of the iPerf3 test, one row per reporting interval (typically 1-second intervals over a 10-second test). Columns:

| Column | Description |
|---|---|
| `start` / `end` | Interval start and end time (seconds) |
| `seconds` | Actual interval duration (s) |
| `bytes` | Bytes transferred in interval |
| `bits_per_second` | Throughput (bps) |
| `jitter_ms` | Jitter (ms) |
| `lost_packets` / `packets` | Lost and total packets |
| `lost_percent` | Packet loss (%) |
| `omitted` | Whether interval was marked omitted by iPerf3 |
| `sender` | Whether row is from sender perspective |
| `host_total/user/system` | Host CPU usage during interval |
| `remote_total/user/system` | Remote CPU usage during interval |
| `flow_type` | Traffic flow type (e.g., `udp`) |
| `target_rate_mbps` | Configured target bitrate (Mbps) |
| `date` / `time` | Date and time identifier of the test |

### `nr-gnb.log`
Full OAI gNB log for the test run. Contains multiple layer events including Random Access (RA) procedure messages, RRC setup, PDU session establishment, per-TTI signal quality metrics (RSRP, SNR, BLER, MCS), HARQ retransmission counts, and MAC/RLC byte counters.


## Dataset CSV

A processed, feature-extracted version of this dataset is provided in `.csv` (in the parent directory). Each row corresponds to one test run identified by `log_id` (format: `YYYYMMDD_HHMMSS`), and includes aggregated iPerf3 metrics, gNB signal quality and protocol event features, and git commit code features derived from the OAI repository history.
