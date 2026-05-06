# Setup & Run Guide for Teammates

## Prerequisites

- **OMNeT++ 6.3.0** installed
- **INET Framework 4.6** compiled
- Bash shell

## Quick Start

### 1. Build the Project

```bash
cd /path/to/AdvancedNetworksProject
make clean
make
```

### 2. Run Simulations via GUI (QtEnv)

```bash
# From within OMNeT++ IDE or via terminal:
./AdvancedNetworksProject
```

Then select a configuration from the dropdown and click **Run**.

### 3. Run Simulations Headless (Cmdenv)

Set your INET library path first. Find your INET installation:

```bash
# Example (adjust path to your INET location):
export INET_PATH=/path/to/inet4.6
export LD_LIBRARY_PATH=$INET_PATH/src:$LD_LIBRARY_PATH

# Then run a config:
opp_run -u Cmdenv -c RIPng -n .:"$INET_PATH/src" -l INET omnetpp.ini
```

**Common INET paths:**
- Linux: `~/omnetpp-6.3.0/inet4.6/src` or `/opt/inet4.6/src`
- macOS: `/Users/username/omnetpp-6.3.0/inet4.6/src`
- Windows: `C:\omnetpp-6.3.0\inet4.6\src`

### 4. Verify Installation

Test RIPng (baseline routing):

```bash
export LD_LIBRARY_PATH=/path/to/inet4.6/src:$LD_LIBRARY_PATH
opp_run -u Cmdenv -c RIPng -n .:/path/to/inet4.6/src -l INET omnetpp.ini 2>&1 | tail -3
```

Expected output:
```
** Event #3491   t=400   Elapsed: 0.040291s (0m 00s)  100% completed
<!> Simulation time limit reached -- at t=400s, event #3491
End.
```

Test OSPFv3 (link-state routing with XML config):

```bash
opp_run -u Cmdenv -c OSPFv3 -n .:/path/to/inet4.6/src -l INET omnetpp.ini 2>&1 | tail -3
```

Expected output:
```
** Event #26646   t=400   Elapsed: 0.108857s (0m 00s)  100% completed
<!> Simulation time limit reached -- at t=400s, event #26646
End.
```

## Available Configurations

All configurations defined in `omnetpp.ini`:

| Part | Config | Purpose |
|------|--------|---------|
| I | `RIPng` | Baseline distance-vector routing |
| I | `OSPFv3` | Link-state routing with XML splitter config |
| II | `TCP_Reno_Baseline` | TCP congestion analysis (Reno) |
| II | `TCP_Cubic_Compare` | TCP congestion analysis (CUBIC) |
| II | `UDP_Streaming` | UDP traffic generation |
| III | `QoS_DiffServ_WRR` | DiffServ with weighted round-robin |
| III | `QoS_IntServ_RSVP` | IntServ with RSVP (strict priority) |
| IV | `IPv6_Advanced_QoS` | Advanced QoS with DSCP marking |

## Troubleshooting

### Error: "Cannot load library"
- Check `LD_LIBRARY_PATH` points to correct INET location
- Verify INET was compiled for your system (x86_64 vs arm64, debug vs release)

### Error: "findInterfaceByName returned nullptr"
- **This is FIXED** in current version
- If you still see this, delete `ospfv3_splitter_config.xml` and run `git pull` to get the latest fix
- Ensure interface names in XML match network topology (eth0, eth1, eth2 per router)

### Simulation doesn't start
- Check `src/BasicNetwork.ned` is in `src/` directory
- Verify `omnetpp.ini` and `ospfv3_splitter_config.xml` are in project root
- Ensure NED files are compiled: `make clean && make`

### Slow Simulation
- OSPFv3 is slower than RIPng due to more routing events (26k+ vs 3k events)
- This is expected; wait for completion or use GUI to track progress

## Project Structure

```
AdvancedNetworksProject/
├── src/
│   └── BasicNetwork.ned          # Network topology (6 routers, 2 clients, 2 servers)
├── omnetpp.ini                   # 16 simulation configurations
├── ospfv3_splitter_config.xml    # OSPFv3 per-router OSPF process definitions
├── Tests.md                      # Verification results
└── docs/                         # (not tracked in git, ignored)
    ├── 01-omnetpp-inet.md
    ├── 02-project-goal.md
    ├── 03-topology-and-addressing.md
    ├── 04-routing-and-control-plane.md
    ├── 05-transport-qos-and-advanced-topics.md
    ├── 06-configuration-reference.md
    └── 07-experiments-metrics-and-reading-results.md
```

## Support

If you encounter issues:
1. Check this guide for common solutions
2. Verify INET library path matches your installation
3. Run `make clean && make` to rebuild
4. Check `omnetpp.ini` is in the project root directory
5. Ensure `ospfv3_splitter_config.xml` has correct XML syntax
