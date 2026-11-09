# Datasets

The experiments use three public intrusion detection datasets. They are not included in this repository because of their size and licences. Download them from their original sources, prepare them as described below, and place the files in this folder:

```
data/
├── NF-ToN-IoT_final.csv
├── cic_ids_2017.parquet
└── CIC-ToN-IoT.csv
```

Each notebook reads only the columns it needs. Rows without a label are dropped, classes with fewer than 1,000 flows are removed, and the largest classes are capped with a fixed random seed (42), as listed in Table 1 of the paper. The expected counts below let you check that your files match the ones used in the paper.

## NF-ToN-IoT

- **Source:** NetFlow-based datasets of the University of Queensland, [https://staff.itee.uq.edu.au/marius/NIDS_datasets/](https://staff.itee.uq.edu.au/marius/NIDS_datasets/).
- **File:** the NF-ToN-IoT CSV file, saved as `NF-ToN-IoT_final.csv`.
- **Columns used:** `IPV4_SRC_ADDR`, `IPV4_DST_ADDR`, `L4_SRC_PORT`, `L4_DST_PORT`, `PROTOCOL`, `L7_PROTO`, `IN_BYTES`, `OUT_BYTES`, `IN_PKTS`, `OUT_PKTS`, `TCP_FLAGS`, `FLOW_DURATION_MILLISECONDS` and the label column `Attack`.
- **Class caps:** `injection` and `ddos` are capped at 300,000 flows.
- **Expected after loading:** 1,184,248 flows in 9 classes (injection 300,000, ddos 300,000, Benign 270,279, password 156,299, xss 99,944, scanning 21,467, dos 17,717, backdoor 17,247, mitm 1,295).

## CIC-IDS-2017

- **Source:** Canadian Institute for Cybersecurity, [https://www.unb.ca/cic/datasets/ids-2017.html](https://www.unb.ca/cic/datasets/ids-2017.html), folder `GeneratedLabelledFlows/TrafficLabelling`.
- **File:** the eight daily CSV files merged into one file, `cic_ids_2017.parquet`.
- **Columns used:** `Src IP`, `Dst IP`, `Src Port`, `Dst Port`, `Protocol`, `Flow Duration`, `Total Fwd Packets`, `Total Backward Packets`, `Total Length of Fwd Packets`, `Total Length of Bwd Packets`, the `Flow IAT`, `Fwd IAT` and `Bwd IAT` statistics, and the label column `Attack`.
- **Class caps:** `BENIGN` is capped at 500,000 flows.
- **Expected after loading:** 1,056,926 flows in 11 classes (BENIGN 500,000, DoS Hulk 231,073, PortScan 158,930, DDoS 128,027, DoS GoldenEye 10,293, FTP-Patator 7,938, SSH-Patator 5,897, DoS slowloris 5,796, DoS Slowhttptest 5,499, Bot 1,966, Web Attack - Brute Force 1,507).

The original CSV files use column names with leading spaces and a different naming for the address and label columns. The following script merges them into the expected file:

```python
import glob
import pandas as pd

RENAME = {"Source IP": "Src IP", "Destination IP": "Dst IP", "Source Port": "Src Port",
          "Destination Port": "Dst Port", "Label": "Attack"}

parts = []
for path in sorted(glob.glob("TrafficLabelling/*.csv")):
    df = pd.read_csv(path, encoding="latin-1", low_memory=False)
    df.columns = df.columns.str.strip()
    parts.append(df.rename(columns=RENAME))

df = pd.concat(parts, ignore_index=True)
df = df.loc[:, ~df.columns.duplicated()]
df["Attack"] = df["Attack"].astype(str).str.replace(r"^Web Attack \W+ ", "Web Attack - ", regex=True)
df.to_parquet("data/cic_ids_2017.parquet", index=False)
```

## CIC-ToN-IoT

- **Source:** CICFlowMeter version of ToN-IoT, University of Queensland, [https://staff.itee.uq.edu.au/marius/NIDS_datasets/](https://staff.itee.uq.edu.au/marius/NIDS_datasets/).
- **File:** the CIC-ToN-IoT CSV file, saved as `CIC-ToN-IoT.csv`.
- **Columns used:** `Src IP`, `Dst IP`, `Src Port`, `Dst Port`, `Protocol`, `Flow Duration`, `Tot Fwd Pkts`, `Tot Bwd Pkts`, `TotLen Fwd Pkts`, `TotLen Bwd Pkts`, the `Flow IAT`, `Fwd IAT` and `Bwd IAT` statistics, and the label column `Attack`.
- **Class caps:** `Benign` at 500,000, `xss` and `password` at 300,000, and `injection` at 250,000 flows.
- **Expected after loading:** 1,418,448 flows in 7 classes (Benign 500,000, password 300,000, xss 300,000, injection 250,000, scanning 36,205, backdoor 27,145, ransomware 5,098).
