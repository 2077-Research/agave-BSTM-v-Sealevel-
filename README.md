This repository contains the agave fork code used for the Block-STM vs. Sealevle paper tests. Follow these instructions to reproduce the results on a machine of your choice.

# Building Agave

## **1. Install rustc, cargo and rustfmt.**

```bash
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup component add rustfmt
```
On Linux systems you may need to install libssl-dev, pkg-config, zlib1g-dev, protobuf etc.

On Ubuntu:
```bash
sudo apt-get update
sudo apt-get install libssl-dev libudev-dev pkg-config zlib1g-dev llvm clang cmake make libprotobuf-dev protobuf-compiler
```

On Fedora:
```bash
sudo dnf install openssl-devel systemd-devel pkg-config zlib-devel llvm clang cmake make protobuf-devel protobuf-compiler perl-core
```

## **2. Download the source code.**

```bash
git clone https://github.com/2077-Research/agave-BSTM-v-Sealevel-.git
cd agave
```

## **3. Build.**

```bash
./cargo build
```

# Banking Stage Benchmarking

Run the command below to run the benchmarks. Feel free to select additional options.

```bash
./cargo run --release --package solana-banking-bench --bin solana-banking-bench
```

## Usage with Commands

```bash
./cargo run --release --package solana-banking-bench --bin solana-banking-bench -- --command(s)
```

### Commands
--iterations <value> - Number of test iterations \
--num-chunks <value> - Number of packet chunks; it's an internal value with little to no significance to the runs. \
--packets-per-batch <value> - Number of packets per batch. Default 200 \
--batches-per-iteration <value> - Number of batches per iteration. Default 5. \
  Note: total number of transactions per iteration = packets-per-batch * batches-per iteration \
--skip-sanity - skip sanity check to ensure tranactions can execute in parallel \
--trace-banking - enable bank tracing i.e., logging scheduled transaction data. Can be used with apfitzge's banking trace tool and graphia to vizualize prio-graphs. \
--tpu-disable-quic - disable quic forwarding \
--block-production-method - options are central-scheduler and thread-local-multi-iterator. CS is default \
--num-banking-threads <value> - number of bank threads (must be >= 3). Default 6 (2 Vote, 4 non-vote) \

#### Write-lock-contetntion Tests -- Default Behaviour \
--write-lock-contention <value> - oprions are full, same-batch-only, and none. Default none \
--simulate-mint - Should there be "mint transactions" Mint transactions are transactions that have higher priority. \
--mint-txs-percentage <value> - percentage of mint transactions. Default 99 \

#### Account-based-contention Tests
--is-accounts - boolean value that indicates that tests should use account-based contention. \
--num-accounts <value> - number of accounts. Default 1000 \

A simple bash script can automate the data collection process for account-based contention. You can modify it to include central-scheduler data if you're interested.

```bash
#!/bin/bash
schedulers=("thread-local-multi-iterator" "central-scheduler")
threads=(6 10 14 18 26 34)
accounts=(2 10 20 30 50 80 100 1000 10000)
packets=(200 2000)
output_directory="/home/ubuntu/data/accounts"

#create data directory
cd
mkdir data

#change directory to agave
cd
cd agave

#run tests
for thread in "${threads[@]}"; do
  for account in "${accounts[@]}"; do
    for packet in "${packets[@]}"; do
      output_file_path="$output_directory/account_${account}_${thread}_${packet}.txt"
      ./cargo run --release --package solana-banking-bench --bin solana-banking-bench \
      -- --iterations 200 --skip-sanity --tpu-disable-quic \
      --block-production-method thread-local-multi-iterator \
      --packets-per-batch "${packet}"
      --block-production-method "${scheduler}" \
      --num-banking-threads "${thread}" \
      --is-accounts --num-accounts "${account}" \
      |& tee "$output_file_path"
      sleep 30s
    done
  done
done
```
