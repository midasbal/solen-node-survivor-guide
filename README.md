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

# 7. Bonus Alpha: First Blood (Deploy Your First Contract)

So your node is running and you want to leave a permanent mark on the Solen Testnet? 

Let's deploy the standard Counter contract. But wait.. the official faucet is bugged (Issue #1), so how do you get gas money?

Welcome to the trenches. Here is the manual workaround to fund your wallet and deploy your first WASM contract.

/////// Step 1: The Faucet Hex Hack ///////


Since the faucet backend reads strings as ASCII, we bypass the frontend and send our raw 32-byte Hex address directly via API.

First, convert your Base58 address to Hex (use Python or any online converter). Then, hit the faucet API directly:

```
curl -i -X POST https://testnet-faucet.solenchain.io/drip \
  -H "Content-Type: application/json" \
  -d "{\"account\": \"YOUR_64_CHAR_HEX_ADDRESS_HERE\"}"
```
Check your balance: ```./target/release/solen --network testnet balance <your_key_name>``` (You should see 150000000000).

/////// Step 2: Build the Contract ///////


Compile the example Rust contract to WASM:

```
rustup target add wasm32-unknown-unknown
cd examples/contracts/counter
cargo build --target wasm32-unknown-unknown --release
cd ../../..
```

/////// Step 3: Deploy & Interact ///////


Fire it onto the network:

```
./target/release/solen --network testnet deploy <your_key_name> examples/contracts/counter/target/wasm32-unknown-unknown/release/solen_example_counter.wasm
```

Save the ```Contract ID``` it gives you!

Now, increment the counter (costs gas):

```
./target/release/solen --network testnet call <your_key_name> <CONTRACT_ID> increment
```

Read the counter value for free (RPC View Call):

```
curl -s -X POST https://testnet-rpc.solenchain.io \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"solen_callView","params":["<CONTRACT_ID>","get"],"id":1}'
```

Look at the ```"return_data"``` in the output. ```0100...``` means 1, ```0200...``` means 2 (Little-Endian hex). Congrats, you are now a Solen smart contract developer!

___________________________

Stay hungry, stay building. If this guide saved your node, drop a star on the repo. See you on the mainnet!
