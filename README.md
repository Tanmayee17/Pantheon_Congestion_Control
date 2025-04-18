# Pantheon_Congestion_Control

## Overview

This project evaluates and compares three TCP congestion control algorithms — **Cubic**, **BBR**, and **Vegas** — using the *Pantheon* framework and *Mahimahi* network emulation tool. Two different network environments were tested:

- **High Bandwidth, Low Latency**: 50 Mbps / 10 ms
- **Low Bandwidth, High Latency**: 1 Mbps / 200 ms

These scenarios help compare performance trade-offs across delay-based, loss-based, and model-based TCP schemes.

---

## Setup Instructions

### 1. Install Python 2.7

```bash
cd /tmp
wget https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz
tar -xf Python-2.7.18.tgz
cd Python-2.7.18
./configure --prefix=/opt/python2.7
make -j$(nproc)
sudo make install
sudo ln -s /opt/python2.7/bin/python2.7 /usr/bin/python
```

---

### 2. Install pip for Python 2.7 and required packages

```bash
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
python get-pip.py
pip2 install --user pyyaml==5.1.2
export PATH=$PATH:~/.local/bin
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
source ~/.bashrc
```

---

### 3. Install Mahimahi (Ravinet version)

```bash
cd ~/pantheon/third_party
git clone https://github.com/ravinet/mahimahi.git
cd mahimahi
sudo apt-get install -y libxcb-xinerama0 libpangocairo-1.0-0 \
libprotobuf-dev protobuf-compiler libssl-dev libhttp-parser-dev \
libalglib-dev libcap-dev libnl-3-dev libnl-genl-3-dev libnl-route-3-dev \
apache2 apache2-dev libxcb-present0 libxcb-present-dev \
libpango1.0-dev libcairo2-dev libgtk2.0-dev libx11-dev

./autogen.sh
./configure
make
sudo make install
```

---

## Clone Pantheon and Install Dependencies

```bash
cd ~/pantheon
python src/wrappers/cubic.py deps
python src/wrappers/bbr.py deps
python src/wrappers/vegas.py deps
python src/experiments/setup.py --install-deps --schemes "cubic bbr vegas"
python src/experiments/setup.py --setup --schemes "cubic bbr vegas"
```

---

## Running Experiments

### 1. Create Trace Files

```bash
mkdir -p traces
echo 50 > traces/50mbps.trace
echo 1 > traces/1mbps.trace
```

---

### 2. Run Tests

#### High Bandwidth, Low Latency

```bash
python src/experiments/test.py local \
  --schemes "vegas cubic bbr" \
  --run-times 1 \
  --runtime 60 \
  --data-dir results_50mbps_10ms \
  --uplink-trace traces/50mbps.trace \
  --downlink-trace traces/50mbps.trace \
  --prepend-mm-cmds "mm-delay 5"
```

#### Low Bandwidth, High Latency

```bash
python src/experiments/test.py local \
  --schemes "vegas cubic bbr" \
  --run-times 1 \
  --runtime 60 \
  --data-dir results_1mbps_200ms \
  --uplink-trace traces/1mbps.trace \
  --downlink-trace traces/1mbps.trace \
  --prepend-mm-cmds "mm-delay 100"
```

---

### 3. Generate Graphs & Stats

```bash
python src/analysis/analyze.py --data-dir results_50mbps_10ms
python src/analysis/analyze.py --data-dir results_1mbps_200ms
```

---

## Output

Each run will generate:

- `*_datalink_throughput_run1.png` → Throughput vs Time
- `*_datalink_delay_run1.png` → Delay vs Time
- `*_stats_run1.log` → Contains average RTT, 95th percentile delay, loss rate, and throughput
- `pantheon_summary_mean.pdf` & `.svg` → Final summary of all protocols
  
---

## Reproducibility

To replicate the results:

1. Clone this repo and follow setup steps.
2. Create trace files using the instructions above.
3. Run both experiments (50Mbps/10ms and 1Mbps/200ms).
4. Generate results using the analyze scripts.

---

## Protocol Descriptions

- **Cubic**: Loss-based, Linux default. Optimized for throughput.
- **BBR**: Model-based, targets both high throughput and low latency.
- **Vegas**: Delay-based, tries to minimize RTT at the expense of throughput.

---


