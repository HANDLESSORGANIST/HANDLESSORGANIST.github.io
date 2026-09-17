# Tezos Baker: Sources to Monitor

Build 1 (2026-09-17)

Watch list for the Tezos baker work. The chain moves under us, so nothing here
should be treated as settled: re-check Tier 1 at the start of every work session,
Tier 2 weekly or whenever a proposal is in the voting period.

## Environment note (read first)

Every host on this list is currently rejected by this session's egress policy
(the proxy answers 403 to CONNECT). Verified 2026-09-17 for
`pq-shownet-09-16.pqpark.dal.nomadic-labs.com`, `research-development.nomadic-labs.com`,
`docs.nomadic-labs.com`, `tuto-docker.pqpark.dal.nomadic-labs.com`,
`pqpark.dal.nomadic-labs.com`, `octez.tezos.com`, `tezos.gitlab.io`,
`api.tzkt.io`, and `spotlight.tezos.com`.

Practical consequence: from a Claude Code web session these pages can only be
reached indirectly (web search summaries), not fetched. To pull live values
(chain id, protocol hash, bootstrap peers) either run the fetch from a local
machine or from a baker host, or get these domains added to the environment's
allowed egress list.

## Tier 1: live network state, re-check every session

### PQPark post-quantum shownet root

- Current: https://pq-shownet-09-16.pqpark.dal.nomadic-labs.com/
- Previous: https://pq-shownet-09-10.pqpark.dal.nomadic-labs.com/

The hostname is the version number. Pattern: `pq-shownet-MM-DD.pqpark.dal.nomadic-labs.com`,
where `MM-DD` is the date that network was stood up. These are throwaway networks:
when Nomadic Labs rolls a new one the old host stops being authoritative (and
usually stops resolving), so the first check each session is "is there a newer
dated host than the one recorded above". Bump the Build number here when it moves.

Machine-readable config on the same host:

- `https://<net>.<base>/network.json` carries `default_bootstrap_peers` and
  `dal_config.bootstrap_peers`. This is the file to diff, not the rendered page.
  For the current net that is
  `https://pq-shownet-09-16.pqpark.dal.nomadic-labs.com/network.json`.

### PQPark join tutorial

- https://tuto-docker.pqpark.dal.nomadic-labs.com/ (join a PQPark network from macOS, Docker based)

This is the operational source for what the baker command line has to look like
on the current shownet: docker image tags, volume layout, the flags for pointing
`octez-baker` at a DAL node, and the consensus key rotation procedure.

## Tier 2: protocol and release sources

### Nomadic Labs (added 2026-09-17 at Rob's request)

- https://research-development.nomadic-labs.com/ (protocol proposal and previewnet announcements, e.g. the Ushuaia announcement)
- https://docs.nomadic-labs.com/nomadic-labs-knowledge-center/ (knowledge center, recommended tooling)
- https://pqpark.dal.nomadic-labs.com/ (PQPark base domain)

Nomadic Labs is the upstream for the post-quantum networks, so it is both the
announcement channel and the operator of the shownets in Tier 1. Treat an
announcement post there as the trigger to go re-read Tier 1.

### Octez (node, baker, accuser)

- https://octez.tezos.com/releases/ (current releases; the GitLab releases page is deprecated)
- https://octez.tezos.com/docs/CHANGES.html (changelog)
- https://octez.tezos.com/docs/shell/dal_bakers.html (bakers and the DAL)
- https://octez.tezos.com/docs/alpha/accounts.html (account and key types, including tz5)

### Test networks and community

- https://teztnets.xyz/ (test network registry and parameters)
- https://forum.tezosagora.org/ (Research and Development category; live testing threads)
- https://docs.tezos.com/tutorials/join-dal-baker (canonical baker + DAL setup, mainnet flavored)
- https://bakers.tezos.com/ (baking portal)
- https://bakingsheet.tezoscommons.org/ (ecosystem newsletter)
- https://spotlight.tezos.com/ (post-quantum explainers, e.g. "Why post-quantum, why now?")

## What to diff on each check

1. Is the Tier 1 shownet host still the newest dated one?
2. `network.json`: chain id, `default_bootstrap_peers`, `dal_config.bootstrap_peers`.
3. Protocol hash running on the net, and whether our binaries support it.
4. Key types the net expects for the baker (manager key vs consensus key; see below).
5. Docker image tag in the tutorial versus what we have pulled.
6. `octez-baker` flags, especially the DAL node RPC endpoint flag.
7. Anything in the Octez changelog marked as breaking for bakers.

## Baseline as of 2026-09-17

Recorded so the next check has something to diff against. Sourced from web search
summaries, not from fetching the pages (see the environment note), so verify before
relying on any of it.

- Ushuaia, the 21st protocol upgrade proposal, activated 2026-06-30. It added
  testnet-only `tz5` accounts using ML-DSA-44 (NIST FIPS 204, the standardized
  form of CRYSTALS-Dilithium), behind a feature flag on mainnet.
- Under Ushuaia, `tz5` accounts support user operations only: they cannot be
  registered as delegates and cannot be used as consensus keys.
- The PQPark shownets go further than Ushuaia. The documented setup there is a
  funded and staked `tz5` manager key plus a `tz6` consensus key that does the
  actual signing, with the operator running their own L1 node, a DAL node, and a
  baker attesting the producer's slots. Consensus aggregation uses a STARK-based
  approach borrowed from LeanEthereum for aggregating hash-based signatures.
- Consensus key rotation on PQPark: a new key shows up under `pendings` with its
  activation cycle, activating at cycle n+3 (roughly 20 to 30 minutes out). The
  baker fixes its key set at startup, so it has to be started with both keys:
  the old one signs until the activation cycle, the new one after it.
- A post-quantum previewnet was announced 2026-09-02 with public availability
  during September 2026, and bakers were asked to test through the month.
- Octez v23 introduced protocol-independent `octez-baker` and `octez-accuser`,
  which survive protocol upgrades as long as the activating protocol is
  supported. The protocol-dependent executables are deprecated as of v24 and
  are slated for removal in v25.
- Octez v24 supports protocols up to protocol environment V15, which includes
  the Tallinn proposal. v24.4 fixes a bug in v24.3 and earlier where
  `octez-node` could become unresponsive or fall behind the chain.

## Change log

- Build 1 (2026-09-17): first version. Added Nomadic Labs (research and
  development blog, knowledge center, PQPark base domain) and the PQPark
  post-quantum shownet at `pq-shownet-09-16`, plus Octez, teztnets, Agora, and
  community sources. Recorded the shownet host naming pattern and the
  `network.json` endpoint as the things to actually diff.
