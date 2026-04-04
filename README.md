> **Update:** Check out our official Bug Report on Solen Main Repo: [Issue #1](https://github.com/Solen-Blockchain/solen/issues/1) 🐞

If you're here, you're probably trying to spin up a Solen node and realized the P2P mesh is acting a bit... let's say "unstable." Don't worry, we've been through the trenches.

This guide is for the Chads who want their node up and running while everyone else is stuck at waiting for P2P mesh.

# 1. Pre-reqs (Don't skip or you'll get rekt)

Solen is built in Rust, and it’s hungry for performance. Make sure your VPS isn't a potato.

OS: Ubuntu 22.04 (The GOAT)

Rust: 1.78+

Tools: build-essential, clang (Standard stuff, but crucial).

Get your environment ready
```
sudo apt update && sudo apt install build-essential clang netcat -y
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

# 2. The Build (Smash that Cargo)
Clone the repo and build the binaries. If you run into RocksDB issues, it's probably a path problem. Fix it like a pro:

```
git clone https://github.com/Solen-Blockchain/solen && cd solen
export C_INCLUDE_PATH=/usr/lib/gcc/x86_64-linux-gnu/11/include
cargo build --release
```

# 3. Firewall (Skill Issue Prevention)
Opening ports is basic, but skipping it is a massive L.
```
ufw allow 40333/tcp # P2P Mesh
ufw allow 19944/tcp # RPC Port
ufw allow 19955/tcp # Explorer API
```

# 4. The "Midas Touch" Fix (Fixing the P2P Mesh)
Standard DNS bootstrap addresses can be flaky on some VPS providers. If your node is stuck at no peers found, stop yelling at your monitor and use the Direct IP Strike method.

Pro-tier start command:
```
./target/release/solen-node --network testnet \
    --data-dir ./data/solen-testnet \
    --rpc-port 19944 \
    --p2p-port 40333 \
    --bootstrap /ip4/37.60.228.6/tcp/40333 \
    --bootstrap /ip4/147.182.129.231/tcp/40333
```

# 5. Keeping it Alive (Screen it!)

Running a node in a raw SSH session is for noobs. Use ```screen``` so your node keeps pumping even after you close Termius.

New Session: ```screen -S solen-node```

Detach (Go ghost): ```Ctrl + A``` then ```D```

Resume (Back in the matrix): ```screen -r solen-node```

# 6. Known Issues (Big L's you should know)
⚠️ The ASCII Faucet Disaster (Issue #1)
The current testnet faucet is a bit broken. It treats Base58 addresses like ASCII strings, sending your hard-earned tokens to a black hole.

The Alpha: Convert your address to Hex manually before hitting the faucet.

Proof: Check out the first-ever issue on the repo: GitHub Issue #1 => https://github.com/Solen-Blockchain/solen/issues/1

___________________________

Stay hungry, stay building. If this guide saved your node, drop a star on the repo. See you on the mainnet!
