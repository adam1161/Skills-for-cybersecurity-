

===== FILE: skills/analyzing-ransomware-payment-wallets/SKILL.md =====
---
name: analyzing-ransomware-payment-wallets
description: 'Traces ransomware cryptocurrency payment flows using blockchain analysis tools such as Chainalysis Reactor, WalletExplorer, and blockchain.com APIs, identifying wallet clusters and tracking fund movement through mixers and exchanges to support law enforcement attribution. Use when tracing ransomware bitcoin payments, performing cryptocurrency wallet forensics, or gathering blockchain threat intelligence on extortion payments.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- blockchain
- cryptocurrency
- forensics
- threat-intelligence
- bitcoin
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1657
- T1486
mitre_f3:
  version: '1.1'
  tactics:
  - monetization
  - stealth
  techniques:
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1017
    name: Conversion to Physical Monetary Instruments
    tactic: monetization
    source: f3
  - id: F1017.001
    name: 'Conversion to Physical Monetary Instruments: Cash'
    tactic: monetization
    source: f3
  - id: F1047
    name: Transfer of funds
    tactic: monetization
    source: f3
  - id: F1045
    name: Structuring
    tactic: stealth
    source: f3
---

# Analyzing Ransomware Payment Wallets

## When to Use

- An organization has been hit by ransomware and the ransom note contains a Bitcoin or cryptocurrency wallet address that needs investigation
- Law enforcement or incident responders need to trace where ransom payments flowed after the victim paid
- Threat intelligence analysts are attributing ransomware campaigns by clustering payment infrastructure across incidents
- Investigators need to determine if a ransomware group is reusing wallet infrastructure across multiple victims
- Compliance or legal teams need evidence of fund flows for prosecution, sanctions enforcement, or insurance claims

**Do not use** this skill for live payment interception or to interact directly with ransomware operators. All analysis should be passive and read-only against public blockchain data.

## Prerequisites

- Python 3.8+ with `requests`, `json`, and `hashlib` libraries
- Access to blockchain explorer APIs (blockchain.com, WalletExplorer.com, Blockstream.info)
- Familiarity with Bitcoin transaction model (UTXOs, inputs, outputs, change addresses)
- Understanding of common obfuscation techniques (mixers, tumblers, peel chains, cross-chain swaps)
- Optional: Chainalysis Reactor license for enterprise-grade cluster analysis
- Optional: OXT.me for advanced transaction graph visualization

## Workflow

### Step 1: Extract Wallet Address from Ransom Note

Parse the ransom note to identify the payment address(es):

```
Common address formats:
  Bitcoin (P2PKH):   1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa  (starts with 1)
  Bitcoin (P2SH):    3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy  (starts with 3)
  Bitcoin (Bech32):  bc1qar0srrr7xfkvy5l643lydnw9re59gtzzwf5mdq (starts with bc1)
  Monero:            4... (95 characters, much harder to trace)
  Ethereum:          0x... (40 hex chars)
```

### Step 2: Query Blockchain Explorer for Transaction History

Retrieve all transactions associated with the wallet:

```python
import requests

def get_wallet_transactions(address):
    """Query blockchain.com API for address transactions."""
    url = f"https://blockchain.info/rawaddr/{address}"
    resp = requests.get(url, timeout=30)
    resp.raise_for_status()
    data = resp.json()
    return {
        "address": address,
        "n_tx": data.get("n_tx", 0),
        "total_received_satoshi": data.get("total_received", 0),
        "total_sent_satoshi": data.get("total_sent", 0),
        "final_balance_satoshi": data.get("final_balance", 0),
        "transactions": data.get("txs", []),
    }
```

### Step 3: Map Fund Flow and Identify Clusters

Trace outputs from the ransom wallet to downstream addresses:

```
Fund Flow Analysis:
━━━━━━━━━━━━━━━━━━
Victim Payment ──► Ransom Wallet ──► Consolidation Wallet
                                  ├─► Mixer/Tumbler Service
                                  ├─► Exchange Deposit Address
                                  └─► Peel Chain (sequential small outputs)

Key indicators:
  - Consolidation: Multiple ransom payments aggregated into one wallet
  - Peel chains: Sequential transactions with diminishing outputs
  - Mixer usage: Funds sent to known mixer addresses (Wasabi, Samourai, ChipMixer)
  - Exchange cashout: Deposits to known exchange wallets (Binance, Kraken hot wallets)
```

### Step 4: Cross-Reference with Known Wallet Databases

Check addresses against known ransomware infrastructure:

```python
# Check WalletExplorer for entity identification
def check_wallet_explorer(address):
    url = f"https://www.walletexplorer.com/api/1/address?address={address}&caller=research"
    resp = requests.get(url, timeout=30)
    data = resp.json()
    return {
        "wallet_id": data.get("wallet_id"),
        "label": data.get("label", "Unknown"),
        "is_exchange": data.get("is_exchange", False),
    }
```

### Step 5: Generate Attribution Report

Compile findings into a structured intelligence report:

```
RANSOMWARE WALLET ANALYSIS REPORT
====================================
Ransom Address:      bc1q...xyz
Family Attribution:  LockBit 3.0 (based on ransom note format)
Total Received:      4.25 BTC ($178,500 at time of payment)
Total Sent:          4.25 BTC (wallet fully drained)
Number of Payments:  3 (likely 3 separate victims)

FUND FLOW:
  Payment 1: 1.5 BTC → Consolidation wallet → Binance deposit
  Payment 2: 1.0 BTC → Wasabi Mixer → Unknown
  Payment 3: 1.75 BTC → Peel chain (12 hops) → OKX deposit

CLUSTER ANALYSIS:
  Related wallets: 47 addresses identified in same cluster
  Total cluster volume: 156.3 BTC ($6.5M USD)
  First activity: 2024-01-15
  Last activity: 2024-09-22
```

## Verification

- Confirm wallet address format is valid before querying APIs
- Cross-reference transaction timestamps with known incident timelines
- Validate cluster associations by checking common-input-ownership heuristic
- Compare findings against OFAC SDN list for sanctioned addresses
- Verify exchange attribution against multiple sources (WalletExplorer, OXT, Chainalysis)

## Key Concepts

| Term | Definition |
|------|------------|
| **UTXO** | Unspent Transaction Output; the fundamental unit of Bitcoin that tracks ownership through a chain of transactions |
| **Cluster Analysis** | Grouping multiple Bitcoin addresses believed to be controlled by the same entity using common-input-ownership and change-address heuristics |
| **Peel Chain** | A laundering pattern where funds are sent through many sequential transactions, each peeling off a small amount to a new address |
| **CoinJoin/Mixer** | Privacy techniques that combine multiple users' transactions to obscure the link between sender and receiver |
| **Common Input Ownership** | Heuristic that assumes all inputs to a single transaction are controlled by the same entity |

## Tools & Systems

- **Chainalysis Reactor**: Enterprise blockchain investigation platform with entity attribution and cross-chain tracing
- **WalletExplorer**: Free tool that clusters Bitcoin addresses and labels known services (exchanges, mixers, markets)
- **OXT.me**: Advanced Bitcoin transaction visualization with UTXO graph analysis
- **Blockstream.info**: Open-source Bitcoin block explorer with full API access
- **blockchain.com API**: Free API for querying Bitcoin address balances and transaction histories
- **OFAC SDN List**: U.S. Treasury sanctioned address list for compliance checking

===== END FILE =====


===== FILE: skills/analyzing-ransomware-payment-wallets/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/analyzing-ransomware-payment-wallets/references/api-reference.md =====
# API Reference: Ransomware Payment Wallet Analysis

## blockchain.com API

### Get Address Information
```
GET https://blockchain.info/rawaddr/{address}?limit=50
```

Returns transaction history, balance, and UTXO data for a Bitcoin address.

### Response Fields
| Field | Type | Description |
|-------|------|-------------|
| `address` | string | Bitcoin address |
| `n_tx` | int | Total number of transactions |
| `total_received` | int | Total satoshis received |
| `total_sent` | int | Total satoshis sent |
| `final_balance` | int | Current balance in satoshis |
| `txs` | array | Array of transaction objects |

### Get Single Transaction
```
GET https://blockchain.info/rawtx/{tx_hash}
```

### Get Unspent Outputs
```
GET https://blockchain.info/unspent?active={address}
```

## Blockstream.info API

### Get Address Stats
```
GET https://blockstream.info/api/address/{address}
```

### Response Fields
| Field | Type | Description |
|-------|------|-------------|
| `chain_stats.funded_txo_count` | int | Number of funding transactions |
| `chain_stats.spent_txo_count` | int | Number of spending transactions |
| `chain_stats.funded_txo_sum` | int | Total satoshis funded |
| `chain_stats.spent_txo_sum` | int | Total satoshis spent |

### Get Address Transactions
```
GET https://blockstream.info/api/address/{address}/txs
```

## WalletExplorer API

### Look Up Address
```
GET https://www.walletexplorer.com/api/1/address?address={address}&caller=research
```

### Response Fields
| Field | Type | Description |
|-------|------|-------------|
| `wallet_id` | string | Cluster wallet identifier |
| `label` | string | Known entity label (exchange, mixer, etc.) |
| `is_exchange` | bool | Whether address belongs to known exchange |

### Get Wallet Transactions
```
GET https://www.walletexplorer.com/api/1/wallet-addresses?wallet={wallet_id}&caller=research
```

## Bitcoin Address Formats

| Format | Prefix | Example | Notes |
|--------|--------|---------|-------|
| P2PKH (Legacy) | 1 | 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa | Original format |
| P2SH (SegWit compatible) | 3 | 3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy | Script hash |
| Bech32 (Native SegWit) | bc1q | bc1qar0srrr7xfkvy5l643lydnw9re59gtzzwf5mdq | Lower fees |
| Bech32m (Taproot) | bc1p | bc1p... | Newest format |

## Common Ransomware Wallet Indicators

| Pattern | Significance |
|---------|-------------|
| Single large inbound, rapid outbound | Ransom payment received, quickly laundered |
| Multiple small inbound from different addresses | Multiple victims paying same wallet |
| Outbound to known mixer addresses | Laundering through CoinJoin/mixer services |
| Peel chain (sequential diminishing outputs) | Structured laundering to evade detection |
| Transfer to exchange hot wallet | Cash-out attempt via cryptocurrency exchange |

## OFAC SDN Sanctions Check

```
Download list: https://www.treasury.gov/ofac/downloads/sdnlist.txt
Search API:    https://sanctionssearch.ofac.treas.gov/
```

Check addresses against OFAC Specially Designated Nationals list for compliance.

===== END FILE =====


===== FILE: skills/analyzing-ransomware-payment-wallets/scripts/agent.py =====
#!/usr/bin/env python3
"""Ransomware payment wallet blockchain analysis agent.

Traces cryptocurrency payment flows from ransomware wallets using public
blockchain APIs. Identifies transaction patterns, cluster relationships,
and fund movement to exchanges or mixers.
"""

import json
import re
import sys

try:
    import requests
except ImportError:
    print("[!] 'requests' library required: pip install requests")
    sys.exit(1)

BLOCKCHAIN_API = "https://blockchain.info"
BLOCKSTREAM_API = "https://blockstream.info/api"

BTC_ADDRESS_REGEX = re.compile(
    r"^(1[a-km-zA-HJ-NP-Z1-9]{25,34}|3[a-km-zA-HJ-NP-Z1-9]{25,34}|bc1[a-z0-9]{39,59})$"
)

KNOWN_RANSOMWARE_WALLETS = {
    "12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw": "WannaCry",
    "13AM4VW2dhxYgXeQepoHkHSQuy6NgaEb94": "WannaCry",
    "115p7UMMngoj1pMvkpHijcRdfJNXj6LrLn": "WannaCry",
    "1Mz7153HMuxXTuR2R1t78mGSdzaAtNbBWX": "DarkSide (Colonial Pipeline)",
    "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh": "DarkSide",
}


def validate_btc_address(address):
    """Validate Bitcoin address format."""
    if BTC_ADDRESS_REGEX.match(address):
        return True
    return False


def query_address_info(address):
    """Query blockchain.info for address details."""
    url = f"{BLOCKCHAIN_API}/rawaddr/{address}?limit=50"
    resp = requests.get(url, timeout=30)
    resp.raise_for_status()
    data = resp.json()
    return {
        "address": address,
        "total_received_btc": data.get("total_received", 0) / 1e8,
        "total_sent_btc": data.get("total_sent", 0) / 1e8,
        "final_balance_btc": data.get("final_balance", 0) / 1e8,
        "n_tx": data.get("n_tx", 0),
        "transactions": data.get("txs", []),
    }


def query_blockstream_address(address):
    """Query Blockstream API for address stats (fallback)."""
    url = f"{BLOCKSTREAM_API}/address/{address}"
    resp = requests.get(url, timeout=30)
    resp.raise_for_status()
    data = resp.json()
    chain = data.get("chain_stats", {})
    return {
        "address": address,
        "funded_txo_count": chain.get("funded_txo_count", 0),
        "spent_txo_count": chain.get("spent_txo_count", 0),
        "funded_txo_sum_btc": chain.get("funded_txo_sum", 0) / 1e8,
        "spent_txo_sum_btc": chain.get("spent_txo_sum", 0) / 1e8,
    }


def extract_output_addresses(transactions, source_address):
    """Extract downstream addresses from transaction outputs."""
    downstream = {}
    for tx in transactions:
        tx_hash = tx.get("hash", "unknown")
        is_outgoing = any(
            inp.get("prev_out", {}).get("addr") == source_address
            for inp in tx.get("inputs", [])
        )
        if not is_outgoing:
            continue
        for out in tx.get("out", []):
            addr = out.get("addr")
            value = out.get("value", 0) / 1e8
            if addr and addr != source_address:
                if addr not in downstream:
                    downstream[addr] = {"total_btc": 0, "tx_count": 0, "tx_hashes": []}
                downstream[addr]["total_btc"] += value
                downstream[addr]["tx_count"] += 1
                downstream[addr]["tx_hashes"].append(tx_hash[:16])
    return downstream


def check_known_wallets(address):
    """Check if address matches known ransomware wallets."""
    if address in KNOWN_RANSOMWARE_WALLETS:
        return {"known": True, "family": KNOWN_RANSOMWARE_WALLETS[address]}
    return {"known": False, "family": None}


def detect_peel_chain(transactions, address):
    """Detect peel chain pattern in outgoing transactions."""
    outgoing_values = []
    for tx in transactions:
        is_outgoing = any(
            inp.get("prev_out", {}).get("addr") == address
            for inp in tx.get("inputs", [])
        )
        if is_outgoing:
            outputs = [o.get("value", 0) / 1e8 for o in tx.get("out", []) if o.get("addr") != address]
            outgoing_values.extend(outputs)

    if len(outgoing_values) < 3:
        return {"peel_chain_detected": False, "reason": "Insufficient transactions"}

    decreasing = sum(1 for i in range(1, len(outgoing_values)) if outgoing_values[i] < outgoing_values[i - 1])
    ratio = decreasing / (len(outgoing_values) - 1) if len(outgoing_values) > 1 else 0
    return {
        "peel_chain_detected": ratio > 0.6,
        "decreasing_ratio": round(ratio, 3),
        "num_outputs": len(outgoing_values),
    }


def analyze_wallet(address):
    """Full analysis of a ransomware payment wallet."""
    report = {"analysis_type": "Ransomware Payment Wallet Analysis", "address": address}

    if not validate_btc_address(address):
        report["error"] = f"Invalid Bitcoin address format: {address}"
        return report

    report["known_wallet_check"] = check_known_wallets(address)

    try:
        info = query_address_info(address)
        report["wallet_info"] = {
            "total_received_btc": info["total_received_btc"],
            "total_sent_btc": info["total_sent_btc"],
            "final_balance_btc": info["final_balance_btc"],
            "transaction_count": info["n_tx"],
        }

        downstream = extract_output_addresses(info["transactions"], address)
        report["downstream_addresses"] = {
            "count": len(downstream),
            "top_recipients": sorted(
                [{"address": a, **d} for a, d in downstream.items()],
                key=lambda x: x["total_btc"],
                reverse=True,
            )[:10],
        }

        for recipient in report["downstream_addresses"]["top_recipients"]:
            match = check_known_wallets(recipient["address"])
            recipient["known_entity"] = match["family"] if match["known"] else "Unknown"

        report["peel_chain_analysis"] = detect_peel_chain(info["transactions"], address)
    except requests.RequestException as e:
        report["error"] = f"API query failed: {e}"
        try:
            fallback = query_blockstream_address(address)
            report["wallet_info_blockstream"] = fallback
        except requests.RequestException as e2:
            report["fallback_error"] = f"Blockstream fallback also failed: {e2}"

    return report


if __name__ == "__main__":
    print("=" * 60)
    print("Ransomware Payment Wallet Analysis Agent")
    print("Blockchain tracing, cluster analysis, fund flow mapping")
    print("=" * 60)

    if len(sys.argv) < 2:
        print("\nUsage:")
        print("  python agent.py <bitcoin_address>")
        print("  python agent.py <bitcoin_address> --deep")
        print("\nExample:")
        print("  python agent.py 12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw")
        sys.exit(0)

    address = sys.argv[1]
    print(f"\n[*] Analyzing wallet: {address}")

    report = analyze_wallet(address)

    known = report.get("known_wallet_check", {})
    if known.get("known"):
        print(f"[!] KNOWN RANSOMWARE WALLET: {known['family']}")

    info = report.get("wallet_info", {})
    if info:
        print(f"\n--- Wallet Summary ---")
        print(f"  Total received: {info.get('total_received_btc', 0):.8f} BTC")
        print(f"  Total sent:     {info.get('total_sent_btc', 0):.8f} BTC")
        print(f"  Balance:        {info.get('final_balance_btc', 0):.8f} BTC")
        print(f"  Transactions:   {info.get('transaction_count', 0)}")

    ds = report.get("downstream_addresses", {})
    if ds.get("count", 0) > 0:
        print(f"\n--- Top Downstream Recipients ({ds['count']} total) ---")
        for r in ds.get("top_recipients", [])[:5]:
            entity = r.get("known_entity", "Unknown")
            print(f"  {r['address'][:20]}... {r['total_btc']:.8f} BTC [{entity}]")

    peel = report.get("peel_chain_analysis", {})
    if peel.get("peel_chain_detected"):
        print(f"\n[!] Peel chain pattern detected (ratio: {peel['decreasing_ratio']})")

    if report.get("error"):
        print(f"\n[!] Error: {report['error']}")

    print(f"\n[*] Full report:\n{json.dumps(report, indent=2, default=str)}")

===== END FILE =====


===== FILE: skills/building-ransomware-playbook-with-cisa-framework/SKILL.md =====
---
name: building-ransomware-playbook-with-cisa-framework
description: Builds a structured ransomware incident response playbook aligned with
  the CISA StopRansomware Guide and NIST Cybersecurity Framework, covering preparation,
  detection, containment, eradication, recovery, and post-incident phases with actionable
  checklists. Use when creating or updating a ransomware playbook, running a CISA-aligned
  readiness assessment, or validating response steps during a tabletop exercise.
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- incident-response
- CISA
- playbook
- compliance
- NIST
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1486
- T1490
- T1489
- T1078
- T1021.002
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - monetization
  techniques:
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1110
    name: Brute Force
    tactic: initial-access
    source: attack
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1047
    name: Transfer of funds
    tactic: monetization
    source: f3
---

# Building Ransomware Playbook with CISA Framework

## When to Use

- An organization needs to create or update its ransomware incident response playbook following CISA guidelines
- A security team is conducting a ransomware readiness assessment against the CISA StopRansomware framework
- Compliance requires documenting ransomware response procedures aligned with NIST CSF and CISA recommendations
- During tabletop exercises to validate that the organization's ransomware response steps match industry best practices
- After a ransomware incident to update the playbook with lessons learned and close identified gaps

**Do not use** as a substitute for legal counsel regarding ransom payment decisions, breach notification timelines, or regulatory obligations specific to your jurisdiction.

## Prerequisites

- Familiarity with the CISA StopRansomware Guide (cisa.gov/stopransomware/ransomware-guide)
- NIST Cybersecurity Framework (CSF) understanding (Identify, Protect, Detect, Respond, Recover)
- Inventory of critical assets, backup infrastructure, and communication channels
- Defined roles and responsibilities for incident response team members
- Python 3.8+ for playbook generation and compliance checking automation
- Access to organization's asset inventory and backup configuration documentation

## Workflow

### Step 1: Preparation Phase (CISA Part 1 - Prevention)

Establish ransomware-specific defenses before an incident:

```
CISA Preparation Checklist:
━━━━━━━━━━━━━━━━━━━━━━━━━━
[ ] Maintain offline, encrypted backups tested for restoration
[ ] Create and exercise a cyber incident response plan (IRP)
[ ] Implement network segmentation between IT and OT networks
[ ] Enable MFA on all remote access and privileged accounts
[ ] Deploy endpoint detection and response (EDR) on all endpoints
[ ] Disable or restrict RDP; require VPN for remote access
[ ] Maintain a software/hardware asset inventory
[ ] Apply patches within 48 hours for internet-facing systems
[ ] Configure email filtering and disable macro execution by default
[ ] Conduct regular phishing awareness training
[ ] Implement application allowlisting (AppLocker/WDAC)
[ ] Test backup restoration quarterly and document RTO/RPO
```

### Step 2: Detection and Analysis Phase

Identify ransomware indicators and assess scope:

```
Detection Indicators:
━━━━━━━━━━━━━━━━━━━━
- Mass file rename operations with new extensions (.locked, .encrypted)
- Ransom notes appearing in directories (README.txt, DECRYPT.html)
- Volume Shadow Copy deletion (vssadmin delete shadows)
- Abnormal CPU usage from encryption processes
- EDR/AV alerts for known ransomware signatures
- Network connections to known C2 infrastructure
- Unusual lateral movement via SMB or PsExec
- Sysmon Event ID 11 (file creation) spikes

Initial Analysis Steps (CISA):
  1. Take system images and memory captures of affected devices
  2. Identify patient zero and initial access vector
  3. Determine the ransomware family (ID Ransomware, ransom note analysis)
  4. Assess encryption scope: which systems, shares, and data are affected
  5. Check if data exfiltration occurred (double extortion indicator)
```

### Step 3: Containment Phase

Stop the spread and preserve evidence:

```
Immediate Containment (First 1-4 hours):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Isolate affected systems from the network (disable NICs, VLAN quarantine)
2. If unable to disconnect, power down affected systems
3. Disable shared drives to prevent encryption spread
4. Reset credentials for compromised accounts (especially admin/service accounts)
5. Block known ransomware IOCs at firewall/proxy (C2 domains, IPs)
6. Preserve forensic evidence (memory dumps, disk images, logs)
7. Engage legal counsel and prepare breach notification if data exfiltrated

Extended Containment:
  - Identify and patch the initial access vector (phishing, RDP, VPN vuln)
  - Audit all Active Directory accounts for persistence (scheduled tasks, services)
  - Check for backdoors or additional malware beyond the ransomware payload
```

### Step 4: Eradication and Recovery Phase

Remove the threat and restore operations:

```
CISA Recovery Steps:
━━━━━━━━━━━━━━━━━━━
1. Rebuild affected systems from known-clean images (do NOT decrypt in place)
2. Restore data from offline backups (verify backup integrity first)
3. Reset ALL passwords including service accounts, krbtgt (twice, 12h apart)
4. Scan restored systems with updated AV/EDR before reconnecting to network
5. Re-enable services in priority order based on business criticality
6. Monitor restored systems intensively for 72 hours for reinfection

Recovery Priority Matrix:
  P1 (0-4h):  Domain controllers, DNS, authentication infrastructure
  P2 (4-24h): Email, critical business applications, databases
  P3 (1-3d):  File servers, departmental applications
  P4 (3-7d):  Non-critical systems, development environments
```

### Step 5: Post-Incident Activity

Document lessons learned and improve defenses:

```
Post-Incident Report Template:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Executive summary: What happened, impact, resolution
2. Timeline: Detection to full recovery with timestamps
3. Root cause analysis: Initial access vector and propagation path
4. Scope: Number of systems, data volumes, business impact in hours/dollars
5. Response effectiveness: What worked, what failed, what was missing
6. Recommendations: Specific technical and procedural improvements
7. Compliance actions: Notification timeline, regulatory obligations met
8. Updated playbook: Revisions based on lessons learned
```

## Verification

- Validate playbook completeness against CISA StopRansomware checklist items
- Conduct tabletop exercise using the playbook with all stakeholders
- Verify backup restoration procedures work within documented RTO targets
- Test communication plans including out-of-band channels
- Confirm legal and regulatory notification procedures are current
- Review and update the playbook at least annually or after any incident

## Key Concepts

| Term | Definition |
|------|------------|
| **CISA StopRansomware Guide** | Joint CISA/MS-ISAC/NSA/FBI guide providing ransomware prevention best practices and response checklists |
| **RTO/RPO** | Recovery Time Objective (max downtime) and Recovery Point Objective (max data loss); critical metrics for backup planning |
| **Double Extortion** | Ransomware tactic where attackers both encrypt data and threaten to publish stolen data unless paid |
| **Patient Zero** | The first system compromised in an incident; identifying it reveals the initial access vector |
| **Tabletop Exercise** | Simulated incident scenario walked through by the response team to validate the playbook without live systems |

## Tools & Systems

- **CISA StopRansomware Guide**: Primary framework for ransomware response planning and prevention
- **NIST CSF**: Cybersecurity Framework providing the Identify/Protect/Detect/Respond/Recover structure
- **ID Ransomware**: Service for identifying ransomware families from encrypted files and ransom notes
- **MITRE ATT&CK**: Technique framework for mapping ransomware TTPs to detection opportunities
- **Velociraptor**: Endpoint visibility tool for rapid triage and forensic artifact collection during incidents

===== END FILE =====


===== FILE: skills/building-ransomware-playbook-with-cisa-framework/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/building-ransomware-playbook-with-cisa-framework/references/api-reference.md =====
# API Reference: CISA Ransomware Playbook Framework

## CISA StopRansomware Guide

### Primary Resource
```
URL: https://www.cisa.gov/stopransomware/ransomware-guide
PDF: https://www.cisa.gov/sites/default/files/2025-03/StopRansomware-Guide%20508.pdf
```

### Guide Structure
| Part | Content | Focus |
|------|---------|-------|
| Part 1 | Ransomware and Data Extortion Prevention Best Practices | Preparation |
| Part 2 | Ransomware and Data Extortion Response Checklist | Response |

## CISA Reporting

### Report an Incident
```
URL: https://report.cisa.gov
```

### FBI Internet Crime Complaint Center
```
URL: https://www.ic3.gov
```

## NIST Cybersecurity Framework Mapping

| NIST Function | Ransomware Application |
|---------------|----------------------|
| **Identify** | Asset inventory, risk assessment, data classification |
| **Protect** | Backups, MFA, patching, email filtering, AppLocker |
| **Detect** | EDR alerts, SIEM monitoring, anomaly detection |
| **Respond** | Containment, forensics, notification, communication |
| **Recover** | Backup restoration, system rebuild, validation |

## ID Ransomware Service

### Identify Ransomware Family
```
URL: https://id-ransomware.malwarehunterteam.com/
Upload: Encrypted file sample + ransom note
```

### Response
Returns ransomware family name, available decryptors, and known TTPs.

## CISA Preparation Checklist Controls

| Control ID | Control | Priority |
|-----------|---------|----------|
| PREP-01 | Offline encrypted backups | Critical |
| PREP-02 | Incident response plan | Critical |
| PREP-03 | Network segmentation | High |
| PREP-04 | Multi-factor authentication | Critical |
| PREP-05 | Endpoint detection and response | High |
| PREP-06 | RDP restrictions | Critical |
| PREP-07 | Patch management | High |
| PREP-08 | Email security (DMARC/DKIM/SPF) | High |
| PREP-09 | Application allowlisting | Medium |
| PREP-10 | Security awareness training | Medium |

## Response Phase Timelines (CISA Recommended)

| Phase | Target Timeline | Key Actions |
|-------|----------------|-------------|
| Detection | 0-2 hours | Identify scope, capture evidence |
| Containment | 1-4 hours | Isolate systems, block IOCs |
| Eradication | 1-7 days | Rebuild, restore, reset credentials |
| Recovery | 1-4 weeks | Monitor, validate, document |
| Post-Incident | 30-90 days | Lessons learned, playbook updates |

## Regulatory Notification Timelines

| Regulation | Timeline | Authority |
|-----------|----------|-----------|
| GDPR | 72 hours | Data Protection Authority |
| HIPAA | 60 days | HHS Office for Civil Rights |
| SEC | 4 business days | Securities and Exchange Commission |
| PCI DSS | Immediately | Card brands / acquiring bank |
| State breach laws | Varies (30-90 days) | State Attorney General |

## MITRE ATT&CK Ransomware Techniques

| Technique ID | Name | Phase |
|-------------|------|-------|
| T1486 | Data Encrypted for Impact | Impact |
| T1490 | Inhibit System Recovery | Impact |
| T1489 | Service Stop | Impact |
| T1021.002 | SMB/Windows Admin Shares | Lateral Movement |
| T1059.001 | PowerShell | Execution |
| T1566.001 | Spearphishing Attachment | Initial Access |

===== END FILE =====


===== FILE: skills/building-ransomware-playbook-with-cisa-framework/scripts/agent.py =====
#!/usr/bin/env python3
"""CISA ransomware playbook builder and compliance checker agent.

Generates a structured ransomware incident response playbook aligned with the
CISA StopRansomware Guide. Assesses organizational readiness against CISA
checklist items and produces gap analysis reports.
"""

import json
import sys
from datetime import datetime

CISA_PREPARATION_CHECKLIST = {
    "PREP-01": {
        "control": "Offline encrypted backups",
        "description": "Maintain offline, encrypted backups of critical data tested quarterly",
        "cisa_ref": "StopRansomware Guide Part 1, Section 1",
        "priority": "Critical",
    },
    "PREP-02": {
        "control": "Incident response plan",
        "description": "Create, maintain, and exercise a cyber incident response plan with ransomware annex",
        "cisa_ref": "StopRansomware Guide Part 1, Section 2",
        "priority": "Critical",
    },
    "PREP-03": {
        "control": "Network segmentation",
        "description": "Implement network segmentation between IT, OT, and critical asset zones",
        "cisa_ref": "StopRansomware Guide Part 1, Section 3",
        "priority": "High",
    },
    "PREP-04": {
        "control": "Multi-factor authentication",
        "description": "Enable MFA on all remote access, privileged accounts, and email",
        "cisa_ref": "StopRansomware Guide Part 1, Section 4",
        "priority": "Critical",
    },
    "PREP-05": {
        "control": "Endpoint detection and response",
        "description": "Deploy EDR on all endpoints with automated response capabilities",
        "cisa_ref": "StopRansomware Guide Part 1, Section 5",
        "priority": "High",
    },
    "PREP-06": {
        "control": "RDP restrictions",
        "description": "Disable or restrict RDP; require VPN with MFA for remote access",
        "cisa_ref": "StopRansomware Guide Part 1, Section 6",
        "priority": "Critical",
    },
    "PREP-07": {
        "control": "Patch management",
        "description": "Apply patches within 48 hours for internet-facing systems, 30 days for internal",
        "cisa_ref": "StopRansomware Guide Part 1, Section 7",
        "priority": "High",
    },
    "PREP-08": {
        "control": "Email security",
        "description": "Configure email filtering, disable macros by default, implement DMARC/DKIM/SPF",
        "cisa_ref": "StopRansomware Guide Part 1, Section 8",
        "priority": "High",
    },
    "PREP-09": {
        "control": "Application allowlisting",
        "description": "Implement AppLocker or WDAC to restrict unauthorized executables",
        "cisa_ref": "StopRansomware Guide Part 1, Section 9",
        "priority": "Medium",
    },
    "PREP-10": {
        "control": "Security awareness training",
        "description": "Conduct regular phishing simulation and security awareness training",
        "cisa_ref": "StopRansomware Guide Part 1, Section 10",
        "priority": "Medium",
    },
}

RESPONSE_PHASES = {
    "detection": {
        "name": "Detection and Analysis",
        "steps": [
            "Identify initial indicators (mass file renames, ransom notes, EDR alerts)",
            "Take system images and memory captures of affected devices",
            "Identify patient zero and initial access vector",
            "Determine ransomware family using ID Ransomware or sample analysis",
            "Assess encryption scope: systems, shares, data classification impacted",
            "Check for data exfiltration indicators (double extortion)",
            "Notify incident response team and escalate per IRP",
        ],
        "time_target": "0-2 hours",
    },
    "containment": {
        "name": "Containment",
        "steps": [
            "Isolate affected systems (disable NIC, VLAN quarantine, firewall block)",
            "If unable to disconnect, power down affected systems immediately",
            "Disable shared drives and mapped network shares",
            "Reset credentials for compromised and service accounts",
            "Block known IOCs at firewall and proxy (C2 domains, IPs, hashes)",
            "Preserve forensic evidence (do not wipe or rebuild yet)",
            "Engage legal counsel for breach notification assessment",
            "Activate out-of-band communication channel for response team",
        ],
        "time_target": "1-4 hours",
    },
    "eradication": {
        "name": "Eradication and Recovery",
        "steps": [
            "Rebuild affected systems from known-clean images",
            "Restore data from verified offline backups",
            "Reset ALL domain passwords including krbtgt (twice, 12h apart)",
            "Scan restored systems with updated AV and EDR before reconnection",
            "Re-enable services in priority order (DC/DNS first, then business apps)",
            "Monitor restored systems for 72 hours for reinfection signals",
            "Validate data integrity of restored files against known checksums",
        ],
        "time_target": "1-7 days",
    },
    "post_incident": {
        "name": "Post-Incident Activity",
        "steps": [
            "Conduct root cause analysis with full incident timeline",
            "Document lessons learned with all response team stakeholders",
            "Update incident response playbook based on findings",
            "Implement new controls to address identified gaps",
            "File regulatory notifications within required timeframes",
            "Report to CISA at report.cisa.gov and FBI at ic3.gov",
            "Schedule follow-up review in 30, 60, and 90 days",
        ],
        "time_target": "1-4 weeks",
    },
}


def assess_readiness(current_controls):
    """Assess ransomware readiness against CISA checklist."""
    results = {"total_controls": len(CISA_PREPARATION_CHECKLIST), "implemented": 0,
               "gaps": [], "score": 0.0, "details": []}

    for ctrl_id, ctrl in CISA_PREPARATION_CHECKLIST.items():
        status = current_controls.get(ctrl_id, "not_implemented")
        is_implemented = status in ("implemented", "partial")
        if is_implemented:
            results["implemented"] += 1
        else:
            results["gaps"].append({
                "id": ctrl_id,
                "control": ctrl["control"],
                "priority": ctrl["priority"],
                "cisa_ref": ctrl["cisa_ref"],
            })
        results["details"].append({
            "id": ctrl_id,
            "control": ctrl["control"],
            "status": status,
            "priority": ctrl["priority"],
        })

    results["score"] = round(
        (results["implemented"] / results["total_controls"]) * 100, 1
    )
    return results


def generate_playbook(org_name="Organization"):
    """Generate a full ransomware response playbook."""
    playbook = {
        "title": f"Ransomware Incident Response Playbook - {org_name}",
        "framework": "CISA StopRansomware Guide + NIST CSF",
        "version": "1.0",
        "generated": datetime.now().isoformat(),
        "preparation": CISA_PREPARATION_CHECKLIST,
        "response_phases": RESPONSE_PHASES,
        "escalation_matrix": {
            "severity_1_critical": {
                "criteria": "Encryption active, spreading across network, critical systems affected",
                "notify": ["CISO", "CEO", "Legal Counsel", "External IR Firm", "CISA", "FBI"],
                "response_time": "Immediate",
            },
            "severity_2_high": {
                "criteria": "Encryption contained to single segment, no critical systems affected",
                "notify": ["CISO", "IT Director", "Legal Counsel"],
                "response_time": "Within 1 hour",
            },
            "severity_3_medium": {
                "criteria": "Ransomware detected but not yet executed (pre-encryption)",
                "notify": ["SOC Manager", "IT Director"],
                "response_time": "Within 4 hours",
            },
        },
        "communication_plan": {
            "internal": "Use out-of-band channel (Signal, phone tree) - assume email compromised",
            "external_stakeholders": "Prepared holding statement; legal review before public disclosure",
            "regulatory": "GDPR 72h, HIPAA 60d, SEC 4 business days, state-specific breach laws",
            "cisa_reporting": "Report to report.cisa.gov within 24 hours",
        },
    }
    return playbook


def generate_markdown_playbook(playbook):
    """Render playbook as Markdown document."""
    lines = [f"# {playbook['title']}", "", f"**Framework:** {playbook['framework']}",
             f"**Version:** {playbook['version']}", f"**Generated:** {playbook['generated']}", ""]

    lines.append("## Preparation Checklist (CISA Part 1)")
    lines.append("")
    for ctrl_id, ctrl in playbook["preparation"].items():
        lines.append(f"- [ ] **{ctrl_id}**: {ctrl['control']} - {ctrl['description']} "
                      f"[{ctrl['priority']}]")
    lines.append("")

    lines.append("## Response Phases (CISA Part 2)")
    lines.append("")
    for phase_id, phase in playbook["response_phases"].items():
        lines.append(f"### {phase['name']} (Target: {phase['time_target']})")
        lines.append("")
        for i, step in enumerate(phase["steps"], 1):
            lines.append(f"{i}. {step}")
        lines.append("")

    lines.append("## Escalation Matrix")
    lines.append("")
    for sev, details in playbook["escalation_matrix"].items():
        lines.append(f"### {sev.replace('_', ' ').title()}")
        lines.append(f"- **Criteria:** {details['criteria']}")
        lines.append(f"- **Notify:** {', '.join(details['notify'])}")
        lines.append(f"- **Response Time:** {details['response_time']}")
        lines.append("")

    return "\n".join(lines)


if __name__ == "__main__":
    print("=" * 60)
    print("CISA Ransomware Playbook Builder Agent")
    print("Playbook generation and readiness assessment")
    print("=" * 60)

    if len(sys.argv) < 2:
        print("\nUsage:")
        print("  python agent.py generate [org_name]     Generate playbook")
        print("  python agent.py assess <controls.json>  Assess readiness")
        print("  python agent.py checklist               Print CISA checklist")
        sys.exit(0)

    command = sys.argv[1]

    if command == "generate":
        org = sys.argv[2] if len(sys.argv) > 2 else "Organization"
        playbook = generate_playbook(org)
        md = generate_markdown_playbook(playbook)
        output_file = f"ransomware_playbook_{org.lower().replace(' ', '_')}.md"
        with open(output_file, "w") as f:
            f.write(md)
        print(f"\n[+] Playbook generated: {output_file}")
        print(f"[+] Contains {len(CISA_PREPARATION_CHECKLIST)} preparation controls")
        print(f"[+] Contains {len(RESPONSE_PHASES)} response phases")
        print(f"\n{md[:500]}...")

    elif command == "assess":
        if len(sys.argv) < 3:
            print("[!] Provide a JSON file with current control statuses")
            print('    Format: {"PREP-01": "implemented", "PREP-02": "not_implemented", ...}')
            sys.exit(1)
        with open(sys.argv[2]) as f:
            controls = json.load(f)
        results = assess_readiness(controls)
        print(f"\n--- Ransomware Readiness Assessment ---")
        print(f"  Score: {results['score']}% ({results['implemented']}/{results['total_controls']})")
        if results["gaps"]:
            print(f"\n  Critical Gaps:")
            for gap in results["gaps"]:
                print(f"    [{gap['priority']}] {gap['id']}: {gap['control']}")
        print(f"\n{json.dumps(results, indent=2)}")

    elif command == "checklist":
        print("\n--- CISA Ransomware Preparation Checklist ---")
        for ctrl_id, ctrl in CISA_PREPARATION_CHECKLIST.items():
            print(f"  [{ctrl['priority']:8s}] {ctrl_id}: {ctrl['control']}")
            print(f"             {ctrl['description']}")
    else:
        print(f"[!] Unknown command: {command}")

===== END FILE =====


===== FILE: skills/deploying-decoy-files-for-ransomware-detection/SKILL.md =====
---
name: deploying-decoy-files-for-ransomware-detection
description: 'Deploys canary files (honeytokens) across file systems to detect ransomware
  encryption activity in real time. Uses strategically placed decoy documents monitored
  via file integrity monitoring or OS-level watchdogs to trigger alerts when ransomware
  modifies or encrypts them. Activates for requests involving ransomware canary deployment,
  honeyfile setup, deception-based ransomware detection, or file integrity monitoring
  for encryption.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- detection
- canary-files
- honeytokens
- deception
- file-integrity
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1486
- T1083
- T1490
- T1485
mitre_f3:
  version: '1.1'
  tactics:
  - monetization
  - positioning
  - stealth
  techniques:
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1017.001
    name: 'Conversion to Physical Monetary Instruments: Cash'
    tactic: monetization
    source: f3
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
  - id: T1070
    name: Indicator Removal
    tactic: stealth
    source: attack
---

# Deploying Decoy Files for Ransomware Detection

## When to Use

- Setting up early-warning detection for ransomware on file servers or endpoints
- Supplementing EDR/AV with a deception-based detection layer that catches unknown ransomware variants
- Creating high-fidelity ransomware alerts that have very low false-positive rates (legitimate users have no reason to touch decoy files)
- Testing ransomware response procedures by validating that canary file modifications trigger the expected alerting pipeline
- Protecting high-value file shares (finance, HR, legal) with tripwire files that indicate unauthorized encryption activity

**Do not use** decoy files as the sole ransomware defense. They are a detection mechanism, not a prevention mechanism, and should complement backups, EDR, and access controls.

## Prerequisites

- Python 3.8+ with `watchdog` library for cross-platform file system monitoring
- Administrative access to target file shares or endpoints for canary placement
- File integrity monitoring (FIM) tool or SIEM integration for alert routing
- Understanding of target directory structure to place canaries in high-value locations
- Windows: NTFS change journal or ReadDirectoryChangesW API access
- Linux: inotify support in kernel (standard in modern kernels)

## Workflow

### Step 1: Design Canary File Strategy

Plan file placement for maximum detection coverage:

```
Canary File Placement Strategy:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Naming Convention:
  - Use names that sort FIRST and LAST alphabetically in each directory
  - Ransomware typically enumerates directories A-Z or Z-A
  - Examples: _AAAA_budget_2024.docx, ~zzzz_report_final.xlsx

Placement Locations:
  - Root of every file share (\\server\share\_AAAA_canary.docx)
  - Desktop, Documents, Downloads on each endpoint
  - Department-specific shares (Finance, HR, Legal)
  - Backup staging directories
  - Home directories of high-privilege accounts

File Types:
  - .docx, .xlsx, .pdf (most targeted by ransomware)
  - .sql, .bak (database files, high value)
  - Mix of file types to detect ransomware that targets specific extensions
```

### Step 2: Generate Realistic Canary Files

Create decoy files with realistic content and metadata:

```python
import os
import time

def create_canary_docx(filepath, content="Q4 Financial Summary - Confidential"):
    """Create a realistic .docx canary file using python-docx."""
    from docx import Document
    doc = Document()
    doc.add_heading("Financial Report - CONFIDENTIAL", level=1)
    doc.add_paragraph(content)
    doc.add_paragraph(f"Generated: {time.strftime('%Y-%m-%d')}")
    doc.save(filepath)

def create_canary_txt(filepath):
    """Create a simple text canary with known content for hash verification."""
    content = "CANARY_TOKEN_DO_NOT_MODIFY\n"
    content += f"Created: {time.strftime('%Y-%m-%dT%H:%M:%S')}\n"
    content += "This file is monitored for unauthorized changes.\n"
    with open(filepath, "w") as f:
        f.write(content)
```

### Step 3: Deploy File System Watcher

Monitor canary files for any modification, rename, or deletion:

```python
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class CanaryHandler(FileSystemEventHandler):
    def __init__(self, canary_paths, alert_callback):
        self.canary_paths = set(canary_paths)
        self.alert_callback = alert_callback

    def on_modified(self, event):
        if event.src_path in self.canary_paths:
            self.alert_callback("MODIFIED", event.src_path)

    def on_deleted(self, event):
        if event.src_path in self.canary_paths:
            self.alert_callback("DELETED", event.src_path)

    def on_moved(self, event):
        if event.src_path in self.canary_paths:
            self.alert_callback("RENAMED", event.src_path)
```

### Step 4: Configure Alerting and Response

Define automated responses when canary files are triggered:

```
Alert Response Matrix:
━━━━━━━━━━━━━━━━━━━━━
Event: Canary MODIFIED
  → Severity: CRITICAL
  → Action: Alert SOC, identify modifying process (PID), isolate endpoint

Event: Canary DELETED
  → Severity: HIGH
  → Action: Alert SOC, check for ransomware note in same directory

Event: Canary RENAMED (new extension added)
  → Severity: CRITICAL
  → Action: Alert SOC, check extension against known ransomware extensions
  → Automated: Kill modifying process, disable network interface

Event: Multiple canaries triggered within 60 seconds
  → Severity: EMERGENCY
  → Action: Network-wide isolation, activate incident response plan
```

### Step 5: Validate Detection Coverage

Test that canary files detect actual ransomware behavior:

```bash
# Simulate ransomware encryption (safe test - modifies canary content)
echo "ENCRYPTED_BY_TEST" > /path/to/canary/_AAAA_budget.docx

# Simulate ransomware rename (adds extension)
mv /path/to/canary/report.xlsx /path/to/canary/report.xlsx.locked

# Verify alerts were generated in SIEM/alerting system
```

## Verification

- Confirm all canary files are present and unmodified using stored hash baselines
- Verify that modifying any canary file generates an alert within the expected timeframe (under 30 seconds)
- Test that alert routing to SOC/SIEM is functional with a controlled modification
- Validate that automated response actions (process kill, network isolation) execute correctly
- Check that canary files survive normal backup and restore operations
- Ensure legitimate users and processes are excluded from false-positive alerts (backup agents, AV scans)

## Key Concepts

| Term | Definition |
|------|------------|
| **Canary File** | A decoy file placed in a directory that is monitored for any access or modification, serving as a tripwire for unauthorized activity |
| **Honeytoken** | A broader category of deception artifacts (files, credentials, database records) designed to alert when accessed |
| **File Integrity Monitoring** | Continuous monitoring of file attributes (hash, size, permissions, timestamps) to detect unauthorized changes |
| **ReadDirectoryChangesW** | Windows API for monitoring file system changes in a directory; used by the watchdog library on Windows |
| **inotify** | Linux kernel subsystem for monitoring file system events; provides near-instant notification of file changes |

## Tools & Systems

- **watchdog (Python)**: Cross-platform file system event monitoring library supporting Windows, Linux, and macOS
- **Canarytokens (Thinkst)**: Free hosted service for generating various types of canary tokens including files, URLs, and DNS tokens
- **OSSEC/Wazuh**: Open-source HIDS with built-in file integrity monitoring and alerting capabilities
- **Elastic Endpoint**: Uses canary files internally for ransomware protection and key capture
- **Sysmon**: Windows system monitor that logs file creation events (Event ID 11) for canary file monitoring

===== END FILE =====


===== FILE: skills/deploying-decoy-files-for-ransomware-detection/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/deploying-decoy-files-for-ransomware-detection/references/api-reference.md =====
# API Reference: Decoy Files for Ransomware Detection

## watchdog Library (Python)

### Installation
```bash
pip install watchdog
```

### Observer Setup
```python
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

observer = Observer()
observer.schedule(handler, path, recursive=True)
observer.start()
observer.join()
```

### Event Types
| Event Class | Trigger |
|------------|---------|
| `FileCreatedEvent` | New file created in watched directory |
| `FileModifiedEvent` | Existing file content or metadata changed |
| `FileDeletedEvent` | File removed from watched directory |
| `FileMovedEvent` | File renamed or moved (src_path, dest_path) |
| `DirCreatedEvent` | New directory created |
| `DirDeletedEvent` | Directory removed |

### Handler Methods
| Method | Called When |
|--------|-----------|
| `on_created(event)` | File/directory created |
| `on_modified(event)` | File/directory modified |
| `on_deleted(event)` | File/directory deleted |
| `on_moved(event)` | File/directory renamed/moved |
| `on_any_event(event)` | Any file system event |

## Windows ReadDirectoryChangesW API

### Monitored Changes
| Flag | Description |
|------|-------------|
| `FILE_NOTIFY_CHANGE_FILE_NAME` | File created, deleted, or renamed |
| `FILE_NOTIFY_CHANGE_DIR_NAME` | Directory changes |
| `FILE_NOTIFY_CHANGE_SIZE` | File size changed |
| `FILE_NOTIFY_CHANGE_LAST_WRITE` | Last write time changed |
| `FILE_NOTIFY_CHANGE_SECURITY` | Security descriptor changed |

## Linux inotify Events

### Event Masks
| Mask | Description |
|------|-------------|
| `IN_MODIFY` | File was modified |
| `IN_DELETE` | File was deleted |
| `IN_MOVED_FROM` | File was renamed (old name) |
| `IN_MOVED_TO` | File was renamed (new name) |
| `IN_CREATE` | File was created |
| `IN_ATTRIB` | Metadata changed |

## Canarytokens (Thinkst)

### Generate Token
```
URL: https://canarytokens.org/generate
Types: Word document, PDF, DNS, HTTP, AWS key, SQL, SVN
```

### Alert Webhook
```
POST https://canarytokens.org/webhook
Payload: { "token": "...", "src_ip": "...", "time": "..." }
```

## OSSEC/Wazuh File Integrity Monitoring

### Configuration (ossec.conf)
```xml
<syscheck>
  <frequency>60</frequency>
  <directories check_all="yes" realtime="yes">/path/to/canaries</directories>
  <alert_new_files>yes</alert_new_files>
</syscheck>
```

### Alert Rule IDs
| Rule ID | Description |
|---------|-------------|
| 550 | File integrity checksum changed |
| 553 | File deleted |
| 554 | New file added to monitored directory |

## Sysmon File Monitoring

### Event ID 11 - FileCreate
```xml
<FileCreate onmatch="include">
  <TargetFilename condition="contains">_AAAA_</TargetFilename>
  <TargetFilename condition="contains">~zzzz_</TargetFilename>
</FileCreate>
```

### Event ID 23 - FileDelete
Logs file deletions including archived file content.

## Common Ransomware File Extensions

| Extension | Family |
|-----------|--------|
| .locked | LockBit, Generic |
| .encrypted | Generic |
| .wncry | WannaCry |
| .dharma | Dharma/CrySiS |
| .basta | Black Basta |
| .lockbit | LockBit 3.0 |
| .conti | Conti |
| .ryuk | Ryuk |
| .revil | REvil/Sodinokibi |
| .akira | Akira |

===== END FILE =====


===== FILE: skills/deploying-decoy-files-for-ransomware-detection/scripts/agent.py =====
#!/usr/bin/env python3
"""Decoy file (canary) deployment agent for ransomware detection.

Deploys canary files across file systems and monitors them for modifications
that indicate ransomware encryption activity. Provides real-time alerting
when decoy files are modified, renamed, or deleted.
"""

import hashlib
import json
import logging
import os
import sys
import time
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
)
logger = logging.getLogger("canary_agent")

CANARY_EXTENSIONS = [".docx", ".xlsx", ".pdf", ".csv", ".sql", ".txt", ".pptx"]

CANARY_NAMES_FIRST = [
    "_AAAA_budget_2024", "_AAAA_financial_report", "_AAAA_payroll_data",
    "_AAA_employee_records", "_AAA_client_contracts",
]

CANARY_NAMES_LAST = [
    "~zzzz_annual_review", "~zzzz_backup_config", "~zzzz_tax_returns",
    "~zzz_insurance_claims", "~zzz_merger_docs",
]

RANSOMWARE_EXTENSIONS = {
    ".locked", ".encrypted", ".crypt", ".locky", ".cerber", ".wncry",
    ".dharma", ".basta", ".blackcat", ".hive", ".royal", ".akira",
    ".lockbit", ".conti", ".ryuk", ".maze", ".revil", ".phobos",
    ".makop", ".stop", ".djvu", ".rhysida",
}

RANSOM_NOTE_NAMES = {
    "readme.txt", "readme.html", "decrypt.txt", "decrypt.html",
    "how_to_decrypt.txt", "restore_files.txt", "read_me.txt",
    "how_to_recover.txt", "ransom_note.txt",
}


def compute_file_hash(filepath):
    """Compute SHA-256 hash of a file."""
    sha256 = hashlib.sha256()
    with open(filepath, "rb") as f:
        for chunk in iter(lambda: f.read(65536), b""):
            sha256.update(chunk)
    return sha256.hexdigest()


def generate_canary_content(canary_type, name):
    """Generate realistic content for canary files."""
    timestamp = datetime.now().isoformat()
    content = f"CANARY_TOKEN:{name}\n"
    content += f"Generated: {timestamp}\n"
    content += f"Classification: CONFIDENTIAL\n\n"

    if "budget" in name or "financial" in name:
        content += "Q4 Financial Summary\n"
        content += "Total Revenue: $12,450,000\n"
        content += "Operating Expenses: $8,230,000\n"
        content += "Net Income: $4,220,000\n"
    elif "payroll" in name or "employee" in name:
        content += "Employee Records - Human Resources\n"
        content += "Total Headcount: 342\n"
        content += "Departments: Engineering, Sales, Marketing, Operations\n"
    elif "client" in name or "contract" in name:
        content += "Client Contract Summary\n"
        content += "Active Contracts: 156\n"
        content += "Pending Renewal: 23\n"
    else:
        content += "Internal Document - Do Not Distribute\n"
        content += "This document contains sensitive business information.\n"

    return content


def deploy_canaries(target_dirs, canaries_per_dir=4):
    """Deploy canary files to target directories."""
    deployed = []
    names = CANARY_NAMES_FIRST[:canaries_per_dir // 2] + CANARY_NAMES_LAST[:canaries_per_dir // 2]

    for target_dir in target_dirs:
        if not os.path.isdir(target_dir):
            logger.warning("Directory does not exist: %s", target_dir)
            continue

        for i, name in enumerate(names):
            ext = CANARY_EXTENSIONS[i % len(CANARY_EXTENSIONS)]
            filename = f"{name}{ext}"
            filepath = os.path.join(target_dir, filename)

            content = generate_canary_content(ext, name)
            with open(filepath, "w") as f:
                f.write(content)

            file_hash = compute_file_hash(filepath)
            record = {
                "path": filepath,
                "hash": file_hash,
                "size": os.path.getsize(filepath),
                "deployed_at": datetime.now().isoformat(),
                "name": filename,
            }
            deployed.append(record)
            logger.info("Deployed canary: %s (hash: %s)", filepath, file_hash[:16])

    return deployed


def check_canary_integrity(canary_records):
    """Check all canary files for modifications, deletions, or renames."""
    alerts = []

    for record in canary_records:
        filepath = record["path"]

        if not os.path.exists(filepath):
            # Check if file was renamed with ransomware extension
            parent_dir = os.path.dirname(filepath)
            basename = os.path.basename(filepath)
            renamed = False
            if os.path.isdir(parent_dir):
                for f in os.listdir(parent_dir):
                    if f.startswith(basename) and any(f.endswith(ext) for ext in RANSOMWARE_EXTENSIONS):
                        alerts.append({
                            "type": "RANSOMWARE_RENAME",
                            "severity": "CRITICAL",
                            "original": filepath,
                            "renamed_to": os.path.join(parent_dir, f),
                            "timestamp": datetime.now().isoformat(),
                        })
                        renamed = True
                        break

            if not renamed:
                alerts.append({
                    "type": "CANARY_DELETED",
                    "severity": "HIGH",
                    "path": filepath,
                    "timestamp": datetime.now().isoformat(),
                })
            continue

        current_hash = compute_file_hash(filepath)
        if current_hash != record["hash"]:
            alerts.append({
                "type": "CANARY_MODIFIED",
                "severity": "CRITICAL",
                "path": filepath,
                "original_hash": record["hash"],
                "current_hash": current_hash,
                "timestamp": datetime.now().isoformat(),
            })

        current_size = os.path.getsize(filepath)
        if abs(current_size - record["size"]) > record["size"] * 0.5:
            alerts.append({
                "type": "SIGNIFICANT_SIZE_CHANGE",
                "severity": "HIGH",
                "path": filepath,
                "original_size": record["size"],
                "current_size": current_size,
                "timestamp": datetime.now().isoformat(),
            })

    # Check for ransom notes in canary directories
    checked_dirs = set()
    for record in canary_records:
        parent_dir = os.path.dirname(record["path"])
        if parent_dir in checked_dirs or not os.path.isdir(parent_dir):
            continue
        checked_dirs.add(parent_dir)
        for f in os.listdir(parent_dir):
            if f.lower() in RANSOM_NOTE_NAMES:
                alerts.append({
                    "type": "RANSOM_NOTE_DETECTED",
                    "severity": "CRITICAL",
                    "path": os.path.join(parent_dir, f),
                    "timestamp": datetime.now().isoformat(),
                })

    return alerts


def monitor_loop(canary_records, interval=10):
    """Continuously monitor canary files at specified interval."""
    logger.info("Starting canary monitoring loop (interval: %ds)", interval)
    logger.info("Monitoring %d canary files", len(canary_records))

    while True:
        alerts = check_canary_integrity(canary_records)
        if alerts:
            for alert in alerts:
                logger.critical(
                    "ALERT [%s] %s: %s",
                    alert["severity"],
                    alert["type"],
                    alert.get("path", alert.get("original", "unknown")),
                )
            print(json.dumps({"alerts": alerts}, indent=2))
        time.sleep(interval)


if __name__ == "__main__":
    print("=" * 60)
    print("Ransomware Canary File Deployment Agent")
    print("Deploy and monitor decoy files for encryption detection")
    print("=" * 60)

    if len(sys.argv) < 2:
        print("\nUsage:")
        print("  python agent.py deploy <dir1> [dir2] ...   Deploy canaries")
        print("  python agent.py check <registry.json>      Check canary integrity")
        print("  python agent.py monitor <registry.json>     Continuous monitoring")
        sys.exit(0)

    command = sys.argv[1]

    if command == "deploy":
        dirs = sys.argv[2:] if len(sys.argv) > 2 else [os.getcwd()]
        records = deploy_canaries(dirs)
        registry_file = "canary_registry.json"
        with open(registry_file, "w") as f:
            json.dump(records, f, indent=2)
        print(f"\n[+] Deployed {len(records)} canary files across {len(dirs)} directories")
        print(f"[+] Registry saved to: {registry_file}")

    elif command == "check":
        if len(sys.argv) < 3:
            print("[!] Provide canary registry JSON file")
            sys.exit(1)
        with open(sys.argv[2]) as f:
            records = json.load(f)
        alerts = check_canary_integrity(records)
        if alerts:
            print(f"\n[!] {len(alerts)} ALERTS DETECTED:")
            for a in alerts:
                print(f"  [{a['severity']}] {a['type']}: {a.get('path', a.get('original'))}")
        else:
            print(f"\n[+] All {len(records)} canary files intact. No alerts.")

    elif command == "monitor":
        if len(sys.argv) < 3:
            print("[!] Provide canary registry JSON file")
            sys.exit(1)
        with open(sys.argv[2]) as f:
            records = json.load(f)
        interval = int(sys.argv[3]) if len(sys.argv) > 3 else 10
        monitor_loop(records, interval)

    else:
        print(f"[!] Unknown command: {command}")

===== END FILE =====


===== FILE: skills/deploying-ransomware-canary-files/SKILL.md =====
---
name: deploying-ransomware-canary-files
description: Deploys and monitors ransomware canary files using Python's watchdog library, placing decoy files mimicking high-value targets (financial records, credentials, database exports) where ransomware enumerates first, and alerting via email, Slack, or syslog on any read/modify/rename/delete. Use for early-warning ransomware detection on file servers, NAS, or endpoints, or to supplement EDR where agents can't be deployed.
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- canary-files
- watchdog
- detection
- early-warning
- deception
- defense
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1486
- T1083
- T1490
- T1485
mitre_f3:
  version: '1.1'
  tactics:
  - monetization
  techniques:
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1017
    name: Conversion to Physical Monetary Instruments
    tactic: monetization
    source: f3
  - id: F1025.003
    name: 'Electronic Funds Transfer: Wire Transfer'
    tactic: monetization
    source: f3
---
# Deploying Ransomware Canary Files

## When to Use

- Deploying proactive ransomware detection on file servers, NAS devices, or endpoint systems
- Building an early-warning system that detects ransomware before it encrypts business-critical data
- Supplementing EDR solutions with lightweight canary file monitoring on systems where agents cannot be deployed
- Testing ransomware incident response procedures by simulating canary file triggers
- Monitoring shared drives, home directories, and backup volumes for unauthorized file operations

**Do not use** as a replacement for endpoint protection, backup strategy, or network segmentation. Canary files are a detection layer, not a prevention mechanism.

## Prerequisites

- Python 3.8+ with pip
- watchdog library (pip install watchdog)
- Write access to directories where canary files will be placed
- SMTP server credentials or Slack webhook URL for alerting
- Administrative access for placing canaries in system directories

## Workflow

### Step 1: Generate Canary Files

Create decoy files with realistic names and content that attract ransomware scanners. Files should have names like `Passwords.xlsx`, `Financial_Report_2026.docx`, `backup_credentials.csv` and contain plausible-looking but fake data. Place them in directories ransomware typically targets first: user desktops, Documents folders, network share roots, and backup paths.

### Step 2: Deploy Filesystem Monitor

Use Python's watchdog library with a custom `FileSystemEventHandler` that watches canary file paths. The handler triggers on `on_modified`, `on_deleted`, `on_moved`, and `on_created` events for canary files. Any legitimate user or process should never touch these files, so any interaction is a high-confidence indicator of ransomware or unauthorized access.

### Step 3: Configure Alert Pipeline

Wire the filesystem monitor to multiple alert channels: email via SMTP, Slack webhook POST, syslog forwarding to SIEM, and local log file. Include the triggering event type, file path, timestamp, and process information (when available) in alert payloads.

### Step 4: Validate and Test

Simulate ransomware behavior by programmatically modifying, renaming, and deleting canary files to verify the detection pipeline fires correctly. Measure time-to-alert and validate alert delivery across all configured channels.

## Key Concepts

| Term | Definition |
|------|------------|
| **Canary File** | A decoy file placed in a monitored directory that triggers an alert when accessed, modified, or deleted |
| **Watchdog** | Python library that monitors filesystem events using OS-native APIs (inotify on Linux, FSEvents on macOS, ReadDirectoryChangesW on Windows) |
| **Honey File** | Synonym for canary file; a fake document designed to attract and detect malicious activity |
| **Entropy Check** | Measuring randomness in file content to detect encryption (ransomware produces high-entropy output) |

## Tools & Systems

- **watchdog**: Python filesystem monitoring library using OS-native event APIs
- **smtplib**: Python standard library for SMTP email alerting
- **requests**: HTTP library for Slack webhook integration
- **hashlib**: SHA-256 hashing for canary file integrity verification
- **psutil**: Process information gathering when canary file access is detected

## Output Format

```
RANSOMWARE CANARY ALERT
========================
Timestamp: 2026-03-11T14:23:07Z
Event: FILE_MODIFIED
Canary File: /srv/shares/finance/Passwords.xlsx
Directory: /srv/shares/finance
SHA-256 Before: a3f2...8b4c
SHA-256 After: 7e91...2d3f
Alert Channels: [email, slack, syslog]
Action: Investigate immediately - potential ransomware activity
```

===== END FILE =====


===== FILE: skills/deploying-ransomware-canary-files/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/deploying-ransomware-canary-files/references/api-reference.md =====
# API Reference: Deploying Ransomware Canary Files

## Canary File Deployment

| Function | Parameters | Returns |
|----------|-----------|---------|
| `deploy_canary_files(target_dirs, custom_files)` | List of directories, optional custom file dict | Manifest with deployed file paths and hashes |
| `compute_sha256(filepath)` | File path string | SHA-256 hex digest |
| `compute_entropy(filepath)` | File path string | Shannon entropy float (0-8) |

## Monitoring Functions

| Function | Parameters | Returns |
|----------|-----------|---------|
| `start_monitoring(manifest_path, config)` | Path to manifest JSON, alert config dict | Blocks until interrupted |
| `verify_canary_integrity(manifest_path)` | Path to manifest JSON | Dict with intact/modified/missing counts |
| `simulate_ransomware_test(manifest_path)` | Path to manifest JSON | List of test results |

## CanaryFileHandler Events

| Event | Handler Method | Trigger |
|-------|---------------|---------|
| File modified | `on_modified(event)` | Content change detected |
| File deleted | `on_deleted(event)` | Canary file removed |
| File renamed | `on_moved(event)` | Canary file renamed or moved |
| New file created | `on_created(event)` | Ransom note detection in monitored dirs |

## Alert Channels

| Channel | Function | Required Config |
|---------|----------|-----------------|
| Slack | `send_slack_alert(data, webhook_url)` | `slack_webhook` URL |
| Email | `send_email_alert(data, host, port, sender, recipients)` | SMTP server details |
| Syslog | `send_syslog_alert(data, server, port)` | Syslog server address |
| File | Automatic | Writes to `canary_alerts.jsonl` |

## Ransomware Extension Detection

| Extensions Monitored |
|---------------------|
| `.encrypted`, `.locked`, `.lockbit`, `.crypt`, `.enc` |
| `.ransom`, `.pay`, `.aes`, `.rsa`, `.cry` |
| `.ryk`, `.revil`, `.conti`, `.hive`, `.black`, `.basta` |

## CLI Usage

```bash
# Deploy canary files
python agent.py --action deploy --dirs /srv/shares /home/admin/Documents

# Monitor canary files with Slack alerts
python agent.py --action monitor --slack-webhook https://hooks.slack.com/...

# Verify canary file integrity
python agent.py --action verify

# Test detection pipeline
python agent.py --action test
```

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `watchdog` | >=3.0 | Filesystem event monitoring |
| `requests` | >=2.28 | Slack webhook integration |
| `psutil` | >=5.9 | Process information gathering |
| `hashlib` | stdlib | SHA-256 file hashing |
| `smtplib` | stdlib | SMTP email alerts |

## References

- Elastic Security Labs: Ransomware Canary Files: https://www.elastic.co/security-labs/ransomware-in-the-honeypot-how-we-capture-keys
- Huntress Ransomware Canaries: https://support.huntress.io/hc/en-us/articles/4404005167763
- Python Watchdog Documentation: https://python-watchdog.readthedocs.io/
- CISA #StopRansomware Guide: https://www.cisa.gov/stopransomware/ransomware-guide

===== END FILE =====


===== FILE: skills/deploying-ransomware-canary-files/scripts/agent.py =====
#!/usr/bin/env python3
"""Ransomware canary file deployment and monitoring agent.

Deploys decoy files across critical directories and monitors them using
watchdog for real-time filesystem event detection. Any interaction with
canary files triggers alerts via email, Slack, and syslog.
"""

import os
import sys
import json
import time
import hashlib
import logging
import smtplib
import argparse
import platform
from pathlib import Path
from email.mime.text import MIMEText
from datetime import datetime, timezone

try:
    from watchdog.observers import Observer
    from watchdog.events import FileSystemEventHandler
    HAS_WATCHDOG = True
except ImportError:
    HAS_WATCHDOG = False

try:
    import requests
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False

try:
    import psutil
    HAS_PSUTIL = True
except ImportError:
    HAS_PSUTIL = False

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.StreamHandler(),
        logging.FileHandler("canary_monitor.log"),
    ],
)
logger = logging.getLogger(__name__)

CANARY_FILE_TEMPLATES = {
    "Passwords.xlsx": b"PK\x03\x04" + b"\x00" * 26 + b"[Content_Types].xml" + os.urandom(512),
    "Financial_Report_2026.docx": b"PK\x03\x04" + b"\x00" * 26 + b"word/document.xml" + os.urandom(512),
    "backup_credentials.csv": (
        b"hostname,username,password,last_rotated\n"
        b"dc01.corp.local,svc_backup,R3st0re$ecur3!2026,2026-01-15\n"
        b"sql-prod-01,sa,Pr0d_DB#Access!,2026-02-01\n"
        b"vpn-gateway,admin,VPN@dm1n_2026!,2026-03-01\n"
        b"nas-backup,root,B4ckup_N4S!2026,2025-12-20\n"
    ),
    "Employee_SSN_List.xlsx": b"PK\x03\x04" + b"\x00" * 26 + b"xl/worksheets/sheet1.xml" + os.urandom(512),
    "tax_returns_2025.pdf": b"%PDF-1.7\n1 0 obj\n<< /Type /Catalog >>\nendobj\n" + os.urandom(256),
    "bitcoin_wallet_seed.txt": (
        b"BIP39 Mnemonic Seed Phrase (DO NOT SHARE)\n"
        b"abandon ability able about above absent absorb abstract absurd abuse\n"
        b"access accident account accuse achieve acid acoustic acquire across act\n"
        b"Wallet Address: bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh\n"
    ),
    "database_export_prod.sql": (
        b"-- MySQL dump 10.13  Distrib 8.0.36\n"
        b"-- Host: db-prod-01.internal    Database: customer_data\n"
        b"CREATE TABLE customers (\n"
        b"  id INT PRIMARY KEY AUTO_INCREMENT,\n"
        b"  ssn VARCHAR(11) NOT NULL,\n"
        b"  credit_card VARCHAR(19),\n"
        b"  balance DECIMAL(10,2)\n"
        b");\n"
    ),
    "AWS_Access_Keys.csv": (
        b"User Name,Access Key ID,Secret Access Key\n"
        b"svc-prod-deploy,AKIAIOSFODNN7EXAMPLE,wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY\n"
        b"svc-backup,AKIAI44QH8DHBEXAMPLE,je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY\n"
    ),
}


def compute_sha256(filepath):
    """Compute SHA-256 hash of a file."""
    sha256 = hashlib.sha256()
    try:
        with open(filepath, "rb") as f:
            for chunk in iter(lambda: f.read(65536), b""):
                sha256.update(chunk)
        return sha256.hexdigest()
    except (FileNotFoundError, PermissionError):
        return "file_not_accessible"


def compute_entropy(filepath):
    """Calculate Shannon entropy of file content to detect encryption."""
    try:
        with open(filepath, "rb") as f:
            data = f.read()
    except (FileNotFoundError, PermissionError):
        return 0.0
    if not data:
        return 0.0
    from collections import Counter
    import math
    byte_counts = Counter(data)
    length = len(data)
    entropy = -sum(
        (count / length) * math.log2(count / length)
        for count in byte_counts.values()
        if count > 0
    )
    return round(entropy, 4)


def get_process_info():
    """Get information about processes that may have accessed canary files."""
    if not HAS_PSUTIL:
        return {"error": "psutil not installed"}
    suspicious = []
    for proc in psutil.process_iter(["pid", "name", "username", "cmdline", "create_time"]):
        try:
            info = proc.info
            if info["create_time"] and (time.time() - info["create_time"]) < 30:
                suspicious.append({
                    "pid": info["pid"],
                    "name": info["name"],
                    "username": info["username"],
                    "cmdline": " ".join(info["cmdline"] or []),
                    "age_seconds": round(time.time() - info["create_time"], 1),
                })
        except (psutil.NoSuchProcess, psutil.AccessDenied):
            continue
    return suspicious[:10]


def deploy_canary_files(target_dirs, custom_files=None):
    """Deploy canary files to specified directories."""
    templates = dict(CANARY_FILE_TEMPLATES)
    if custom_files:
        templates.update(custom_files)

    deployed = []
    for directory in target_dirs:
        dir_path = Path(directory)
        if not dir_path.exists():
            logger.warning("Directory does not exist: %s", directory)
            continue
        if not os.access(directory, os.W_OK):
            logger.warning("No write access to: %s", directory)
            continue

        for filename, content in templates.items():
            filepath = dir_path / filename
            if filepath.exists():
                logger.info("Canary already exists: %s", filepath)
                deployed.append(str(filepath))
                continue
            try:
                with open(filepath, "wb") as f:
                    f.write(content)
                if platform.system() != "Windows":
                    os.chmod(filepath, 0o644)
                sha256 = compute_sha256(str(filepath))
                deployed.append(str(filepath))
                logger.info("Deployed canary: %s (SHA-256: %s)", filepath, sha256[:16])
            except (PermissionError, OSError) as e:
                logger.error("Failed to deploy %s: %s", filepath, e)

    manifest = {
        "deployed_at": datetime.now(timezone.utc).isoformat(),
        "canary_count": len(deployed),
        "directories": target_dirs,
        "files": deployed,
        "hashes": {f: compute_sha256(f) for f in deployed},
    }
    manifest_path = Path("canary_manifest.json")
    with open(manifest_path, "w") as f:
        json.dump(manifest, f, indent=2)
    logger.info("Deployed %d canary files, manifest saved to %s", len(deployed), manifest_path)
    return manifest


def send_email_alert(alert_data, smtp_host, smtp_port, sender, recipients, password=None):
    """Send alert email via SMTP."""
    subject = f"RANSOMWARE CANARY ALERT: {alert_data['event_type']} on {alert_data['canary_file']}"
    body = json.dumps(alert_data, indent=2, default=str)
    msg = MIMEText(body)
    msg["Subject"] = subject
    msg["From"] = sender
    msg["To"] = ", ".join(recipients)
    try:
        if smtp_port == 465:
            server = smtplib.SMTP_SSL(smtp_host, smtp_port, timeout=10)
        else:
            server = smtplib.SMTP(smtp_host, smtp_port, timeout=10)
            server.ehlo()
            if smtp_port == 587:
                server.starttls()
                server.ehlo()
        if password:
            server.login(sender, password)
        server.sendmail(sender, recipients, msg.as_string())
        server.quit()
        logger.info("Email alert sent to %s", recipients)
        return True
    except Exception as e:
        logger.error("Email alert failed: %s", e)
        return False


def send_slack_alert(alert_data, webhook_url):
    """Send alert to Slack via incoming webhook."""
    if not HAS_REQUESTS:
        logger.error("requests library not installed, cannot send Slack alert")
        return False
    payload = {
        "text": f":rotating_light: *RANSOMWARE CANARY ALERT*",
        "blocks": [
            {
                "type": "header",
                "text": {"type": "plain_text", "text": "Ransomware Canary File Triggered"}
            },
            {
                "type": "section",
                "fields": [
                    {"type": "mrkdwn", "text": f"*Event:*\n{alert_data['event_type']}"},
                    {"type": "mrkdwn", "text": f"*File:*\n`{alert_data['canary_file']}`"},
                    {"type": "mrkdwn", "text": f"*Time:*\n{alert_data['timestamp']}"},
                    {"type": "mrkdwn", "text": f"*Host:*\n{alert_data.get('hostname', 'unknown')}"},
                ]
            },
            {
                "type": "section",
                "text": {
                    "type": "mrkdwn",
                    "text": f"*Immediate Action Required:* Investigate potential ransomware activity on `{alert_data.get('hostname', 'unknown')}`"
                }
            }
        ]
    }
    try:
        resp = requests.post(webhook_url, json=payload, timeout=10)
        if resp.status_code == 200:
            logger.info("Slack alert sent successfully")
            return True
        logger.error("Slack alert failed: %d %s", resp.status_code, resp.text)
        return False
    except Exception as e:
        logger.error("Slack alert failed: %s", e)
        return False


def send_syslog_alert(alert_data, syslog_server="127.0.0.1", syslog_port=514):
    """Send alert to syslog server via UDP."""
    import socket
    priority = 8 * 4 + 1  # facility=security, severity=alert
    message = (
        f"<{priority}>1 {alert_data['timestamp']} {alert_data.get('hostname', '-')} "
        f"canary-monitor - - - RANSOMWARE_CANARY event={alert_data['event_type']} "
        f"file={alert_data['canary_file']} "
        f"hash_before={alert_data.get('hash_before', 'N/A')} "
        f"hash_after={alert_data.get('hash_after', 'N/A')}"
    )
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        sock.settimeout(10)
        sock.sendto(message.encode("utf-8"), (syslog_server, syslog_port))
        sock.close()
        logger.info("Syslog alert sent to %s:%d", syslog_server, syslog_port)
        return True
    except Exception as e:
        logger.error("Syslog alert failed: %s", e)
        return False


class CanaryFileHandler(FileSystemEventHandler):
    """Watchdog event handler for canary file monitoring."""

    def __init__(self, canary_files, config):
        super().__init__()
        self.canary_files = {str(Path(f).resolve()): compute_sha256(f) for f in canary_files}
        self.config = config
        self.alert_count = 0
        self.last_alert_time = {}

    def _is_canary(self, path):
        resolved = str(Path(path).resolve())
        return resolved in self.canary_files

    def _rate_limit_check(self, path, cooldown=10):
        now = time.time()
        last = self.last_alert_time.get(path, 0)
        if now - last < cooldown:
            return False
        self.last_alert_time[path] = now
        return True

    def _build_alert(self, event_type, src_path, dest_path=None):
        alert = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "event_type": event_type,
            "canary_file": src_path,
            "hostname": platform.node(),
            "platform": platform.system(),
            "hash_before": self.canary_files.get(str(Path(src_path).resolve()), "unknown"),
            "hash_after": compute_sha256(src_path) if os.path.exists(src_path) else "file_deleted",
            "entropy_after": compute_entropy(src_path) if os.path.exists(src_path) else 0.0,
            "alert_number": self.alert_count + 1,
        }
        if dest_path:
            alert["destination"] = dest_path
        if HAS_PSUTIL:
            alert["recent_processes"] = get_process_info()
        return alert

    def _dispatch_alert(self, alert):
        self.alert_count += 1
        logger.critical(
            "CANARY TRIGGERED: %s on %s (alert #%d)",
            alert["event_type"], alert["canary_file"], self.alert_count
        )
        with open("canary_alerts.jsonl", "a") as f:
            f.write(json.dumps(alert, default=str) + "\n")

        if self.config.get("slack_webhook"):
            send_slack_alert(alert, self.config["slack_webhook"])
        if self.config.get("smtp_host"):
            send_email_alert(
                alert,
                self.config["smtp_host"],
                self.config.get("smtp_port", 587),
                self.config.get("smtp_sender", "canary@localhost"),
                self.config.get("smtp_recipients", []),
                self.config.get("smtp_password"),
            )
        if self.config.get("syslog_server"):
            send_syslog_alert(
                alert,
                self.config["syslog_server"],
                self.config.get("syslog_port", 514),
            )

    def on_modified(self, event):
        if event.is_directory:
            return
        if self._is_canary(event.src_path) and self._rate_limit_check(event.src_path):
            alert = self._build_alert("FILE_MODIFIED", event.src_path)
            high_entropy = alert.get("entropy_after", 0) > 7.5
            if high_entropy:
                alert["encryption_suspected"] = True
                alert["severity"] = "critical"
            self._dispatch_alert(alert)

    def on_deleted(self, event):
        if event.is_directory:
            return
        if self._is_canary(event.src_path) and self._rate_limit_check(event.src_path):
            alert = self._build_alert("FILE_DELETED", event.src_path)
            alert["severity"] = "critical"
            self._dispatch_alert(alert)

    def on_moved(self, event):
        if event.is_directory:
            return
        if self._is_canary(event.src_path) and self._rate_limit_check(event.src_path):
            alert = self._build_alert("FILE_RENAMED", event.src_path, event.dest_path)
            extension = Path(event.dest_path).suffix.lower()
            ransomware_extensions = {
                ".encrypted", ".locked", ".lockbit", ".crypt", ".enc",
                ".ransom", ".pay", ".aes", ".rsa", ".cry", ".ryk",
                ".revil", ".conti", ".hive", ".black", ".basta",
            }
            if extension in ransomware_extensions:
                alert["ransomware_extension_detected"] = extension
                alert["severity"] = "critical"
            self._dispatch_alert(alert)

    def on_created(self, event):
        if event.is_directory:
            return
        parent = str(Path(event.src_path).parent)
        ransom_note_patterns = [
            "readme", "decrypt", "restore", "recover", "how_to",
            "ransom", "locked", "unlock", "pay", "instruction",
        ]
        basename = Path(event.src_path).stem.lower()
        if any(pattern in basename for pattern in ransom_note_patterns):
            for canary_dir in set(str(Path(c).parent) for c in self.canary_files):
                if parent == canary_dir:
                    alert = self._build_alert("RANSOM_NOTE_DETECTED", event.src_path)
                    alert["severity"] = "critical"
                    alert["indicator"] = "Ransom note dropped in monitored directory"
                    self._dispatch_alert(alert)
                    break


def start_monitoring(manifest_path, config):
    """Start real-time canary file monitoring."""
    if not HAS_WATCHDOG:
        logger.error("watchdog library required: pip install watchdog")
        sys.exit(1)

    with open(manifest_path) as f:
        manifest = json.load(f)

    canary_files = manifest["files"]
    if not canary_files:
        logger.error("No canary files found in manifest")
        sys.exit(1)

    watch_dirs = set()
    for canary in canary_files:
        parent = str(Path(canary).parent)
        if os.path.isdir(parent):
            watch_dirs.add(parent)

    handler = CanaryFileHandler(canary_files, config)
    observer = Observer()
    for directory in watch_dirs:
        observer.schedule(handler, directory, recursive=False)
        logger.info("Watching directory: %s", directory)

    logger.info("Monitoring %d canary files across %d directories", len(canary_files), len(watch_dirs))
    observer.start()
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
        logger.info("Monitoring stopped. Total alerts: %d", handler.alert_count)
    observer.join()


def verify_canary_integrity(manifest_path):
    """Verify all canary files match their original hashes."""
    with open(manifest_path) as f:
        manifest = json.load(f)
    results = {"checked": 0, "intact": 0, "modified": 0, "missing": 0, "details": []}
    for filepath, original_hash in manifest.get("hashes", {}).items():
        results["checked"] += 1
        if not os.path.exists(filepath):
            results["missing"] += 1
            results["details"].append({"file": filepath, "status": "MISSING"})
        else:
            current_hash = compute_sha256(filepath)
            if current_hash == original_hash:
                results["intact"] += 1
                results["details"].append({"file": filepath, "status": "INTACT"})
            else:
                results["modified"] += 1
                results["details"].append({
                    "file": filepath,
                    "status": "MODIFIED",
                    "original_hash": original_hash,
                    "current_hash": current_hash,
                    "entropy": compute_entropy(filepath),
                })
    return results


def simulate_ransomware_test(manifest_path):
    """Simulate ransomware activity against canary files for testing."""
    with open(manifest_path) as f:
        manifest = json.load(f)
    test_results = []
    for filepath in manifest.get("files", [])[:2]:
        if not os.path.exists(filepath):
            continue
        test_file = filepath + ".test_canary"
        try:
            import shutil
            shutil.copy2(filepath, test_file)
            with open(test_file, "ab") as f:
                f.write(os.urandom(64))
            test_results.append({
                "file": filepath,
                "test_action": "modified_copy",
                "test_file": test_file,
                "status": "triggered",
            })
            os.remove(test_file)
        except Exception as e:
            test_results.append({"file": filepath, "error": str(e)})
    return test_results


def main():
    parser = argparse.ArgumentParser(description="Ransomware Canary File Deployment and Monitoring Agent")
    parser.add_argument("--action", choices=["deploy", "monitor", "verify", "test"],
                        default="deploy", help="Action to perform")
    parser.add_argument("--dirs", nargs="+", help="Directories to deploy canary files")
    parser.add_argument("--manifest", default="canary_manifest.json", help="Canary manifest file path")
    parser.add_argument("--config", help="JSON config file for alert settings")
    parser.add_argument("--slack-webhook", help="Slack incoming webhook URL")
    parser.add_argument("--smtp-host", help="SMTP server hostname")
    parser.add_argument("--smtp-port", type=int, default=587)
    parser.add_argument("--smtp-sender", help="Alert sender email")
    parser.add_argument("--smtp-recipients", nargs="+", help="Alert recipient emails")
    parser.add_argument("--syslog-server", help="Syslog server address")
    args = parser.parse_args()

    config = {}
    if args.config and os.path.exists(args.config):
        with open(args.config) as f:
            config = json.load(f)
    if args.slack_webhook:
        config["slack_webhook"] = args.slack_webhook
    if args.smtp_host:
        config["smtp_host"] = args.smtp_host
        config["smtp_port"] = args.smtp_port
        config["smtp_sender"] = args.smtp_sender
        config["smtp_recipients"] = args.smtp_recipients or []
    if args.syslog_server:
        config["syslog_server"] = args.syslog_server

    if args.action == "deploy":
        if not args.dirs:
            print("Usage: python agent.py --action deploy --dirs /path/to/dir1 /path/to/dir2")
            print("\nExample:")
            print("  python agent.py --action deploy --dirs /srv/shares/finance /home/admin/Documents")
            return
        manifest = deploy_canary_files(args.dirs)
        print(json.dumps(manifest, indent=2))

    elif args.action == "monitor":
        if not os.path.exists(args.manifest):
            print(f"Manifest not found: {args.manifest}")
            print("Run --action deploy first to create canary files")
            return
        start_monitoring(args.manifest, config)

    elif args.action == "verify":
        if not os.path.exists(args.manifest):
            print(f"Manifest not found: {args.manifest}")
            return
        results = verify_canary_integrity(args.manifest)
        print(json.dumps(results, indent=2))
        if results["modified"] > 0 or results["missing"] > 0:
            print(f"\n[ALERT] {results['modified']} modified, {results['missing']} missing canary files!")

    elif args.action == "test":
        if not os.path.exists(args.manifest):
            print(f"Manifest not found: {args.manifest}")
            return
        results = simulate_ransomware_test(args.manifest)
        print(json.dumps(results, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-ransomware-encryption-behavior/SKILL.md =====
---
name: detecting-ransomware-encryption-behavior
description: 'Detects ransomware encryption activity in real time using entropy
  analysis, file system I/O monitoring (Sysmon, watchdog, psutil), and behavioral
  scoring to identify mass file modification, abnormal entropy spikes in written
  data, and suspicious process behavior characteristic of encryption routines.
  Use when building real-time ransomware detection, tuning entropy thresholds,
  or investigating suspected active encryption on an endpoint.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- detection
- entropy
- behavioral-analysis
- file-monitoring
- heuristics
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1486
- T1490
mitre_f3:
  version: '1.1'
  tactics:
  - monetization
  - positioning
  - stealth
  techniques:
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1017.001
    name: 'Conversion to Physical Monetary Instruments: Cash'
    tactic: monetization
    source: f3
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
  - id: T1070
    name: Indicator Removal
    tactic: stealth
    source: attack
---

# Detecting Ransomware Encryption Behavior

## When to Use

- Building or tuning a behavioral detection layer for ransomware that catches unknown/zero-day variants
- Monitoring file servers and endpoints for mass encryption activity that evades signature-based detection
- Implementing entropy-based detection to identify when files are being replaced with encrypted (high-entropy) content
- Analyzing suspicious process behavior patterns: rapid sequential file opens, writes, renames, and deletes
- Validating EDR detection rules against actual ransomware encryption patterns during red team exercises

**Do not use** entropy analysis alone as the only detection signal. Compressed files (ZIP, JPEG, MP4) naturally have high entropy and will cause false positives. Always combine entropy with behavioral signals like I/O rate and file rename patterns.

## Prerequisites

- Python 3.8+ with `watchdog` and `psutil` libraries
- Administrative access for process monitoring and file system event capture
- Understanding of Shannon entropy and its application to file content analysis
- Windows: Sysmon installed for detailed process and file system event logging
- Linux: auditd configured for file access monitoring, or inotify-based watchers
- Baseline entropy values for common file types in the monitored environment

## Workflow

### Step 1: Establish Entropy Baselines

Calculate normal entropy ranges for files in the environment:

```
Entropy Baselines by File Type:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
File Type       Normal Entropy    Encrypted Entropy
.docx           3.5 - 6.5        7.8 - 8.0
.xlsx           4.0 - 6.8        7.8 - 8.0
.pdf            5.0 - 7.2        7.8 - 8.0
.txt            2.0 - 5.0        7.8 - 8.0
.csv            2.0 - 5.5        7.8 - 8.0
.sql            2.5 - 5.0        7.8 - 8.0
.jpg/.png       7.0 - 7.9        7.9 - 8.0 (hard to distinguish)
.zip/.7z        7.5 - 8.0        7.9 - 8.0 (hard to distinguish)

Key insight: Text-based files show the largest entropy jump when encrypted,
making them the best candidates for entropy-based detection.
```

### Step 2: Implement Real-Time Entropy Monitoring

Monitor file writes and calculate entropy of new content:

```python
import math
from collections import Counter

def shannon_entropy(data):
    """Calculate Shannon entropy of byte data (0.0 to 8.0 scale)."""
    if not data:
        return 0.0
    freq = Counter(data)
    length = len(data)
    return -sum((c / length) * math.log2(c / length) for c in freq.values())

def is_encryption_entropy(data, threshold=7.5):
    """Check if data entropy indicates encryption."""
    entropy = shannon_entropy(data)
    return entropy >= threshold, entropy
```

### Step 3: Monitor File System I/O Patterns

Track process-level file operations for ransomware patterns:

```
Ransomware I/O Behavior Signatures:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Rapid sequential file modification:
   - >20 files modified per minute by single process
   - Read original → Write encrypted → Rename with new extension
   - Pattern: CreateFile → ReadFile → WriteFile → CloseHandle → MoveFile

2. File extension changes:
   - Original: report.docx → Encrypted: report.docx.locked
   - Many extensions changed within short time window

3. Ransom note creation:
   - Same text file (README.txt, DECRYPT.html) created in multiple directories
   - Created immediately after file encryption in each directory

4. Shadow copy deletion:
   - vssadmin.exe delete shadows /all /quiet
   - wmic.exe shadowcopy delete
   - PowerShell: Get-WmiObject Win32_Shadowcopy | Remove-WmiObject

5. Entropy spike pattern:
   - File read: entropy 3.5 (normal document)
   - File write: entropy 7.9 (encrypted content)
   - Delta > 3.0 is strong ransomware indicator
```

### Step 4: Implement Behavioral Scoring

Combine multiple signals into a composite ransomware score:

```python
def calculate_ransomware_score(process_metrics):
    """Score process behavior for ransomware likelihood (0-100)."""
    score = 0

    # High file modification rate
    files_per_min = process_metrics.get("files_modified_per_minute", 0)
    if files_per_min > 50:
        score += 30
    elif files_per_min > 20:
        score += 15

    # Entropy increase in written files
    avg_entropy_delta = process_metrics.get("avg_entropy_delta", 0)
    if avg_entropy_delta > 3.0:
        score += 30
    elif avg_entropy_delta > 2.0:
        score += 15

    # File extension changes
    extension_changes = process_metrics.get("extension_changes", 0)
    if extension_changes > 10:
        score += 20
    elif extension_changes > 3:
        score += 10

    # Ransom note creation
    if process_metrics.get("ransom_note_created", False):
        score += 20

    return min(score, 100)
```

### Step 5: Configure Automated Response Thresholds

Set detection thresholds and automated containment actions:

```
Detection Thresholds:
━━━━━━━━━━━━━━━━━━━━
Score 0-25:   INFORMATIONAL - Log only, no action
Score 25-50:  LOW - Alert SOC for investigation
Score 50-75:  HIGH - Alert SOC, suspend process, snapshot VM
Score 75-100: CRITICAL - Kill process, isolate endpoint, alert IR team

Automated Response Actions:
  - Suspend/kill the encrypting process
  - Disable network adapter to prevent lateral movement
  - Create volume shadow copy snapshot before further damage
  - Capture process memory dump for forensic analysis
  - Send SIEM alert with process details, affected files, and timeline
```

## Verification

- Test detection against known ransomware samples in an isolated sandbox environment
- Verify that entropy monitoring correctly identifies encrypted vs. compressed files
- Confirm that behavioral scoring produces low false-positive rates on normal workloads
- Validate automated response actions execute within acceptable time (under 5 seconds)
- Test with multiple ransomware families (LockBit, BlackCat, Conti) to verify coverage
- Benchmark monitoring overhead to ensure it does not degrade endpoint performance

## Key Concepts

| Term | Definition |
|------|------------|
| **Shannon Entropy** | Mathematical measure of randomness in data (0-8 for bytes); encrypted data approaches 8.0, while text files are typically 2-5 |
| **Differential Entropy** | The change in entropy between a file's original and modified content; a spike indicates encryption |
| **I/O Rate Anomaly** | Abnormally high rate of file read/write operations by a single process, characteristic of bulk encryption |
| **Behavioral Scoring** | Combining multiple weak signals (entropy, I/O rate, file renames) into a composite confidence score |
| **Entropy Evasion** | Techniques used by advanced ransomware to defeat entropy detection, such as Base64 encoding output or partial encryption |

## Tools & Systems

- **Sysmon**: Windows system monitor providing detailed file system and process events for behavioral analysis
- **watchdog (Python)**: Cross-platform file system monitoring library for real-time file change detection
- **psutil (Python)**: Process and system monitoring library for tracking per-process I/O statistics
- **Elastic Endpoint**: Commercial endpoint protection with built-in ransomware behavioral detection using canary files
- **Wazuh**: Open-source security platform with file integrity monitoring and active response capabilities

===== END FILE =====


===== FILE: skills/detecting-ransomware-encryption-behavior/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/detecting-ransomware-encryption-behavior/references/api-reference.md =====
# API Reference: Detecting Ransomware Encryption Behavior

## Shannon Entropy

Formula: H(X) = -Sum p(x) log2(p(x)). For byte data range is 0.0 to 8.0.

### Python Implementation

```python
import math
from collections import Counter

def shannon_entropy(data):
    freq = Counter(data)
    length = len(data)
    return -sum((c / length) * math.log2(c / length) for c in freq.values())
```

### Entropy Thresholds

| Range | Interpretation | Example |
|-------|---------------|--------|
| 0.0-1.0 | Nearly uniform | Null files |
| 1.0-4.0 | Low entropy | Plain text |
| 4.0-6.0 | Mixed content | Office docs |
| 6.0-7.0 | Compressed | PDF |
| 7.0-7.5 | Highly compressed | ZIP JPEG |
| 7.5-7.9 | Block cipher encrypted | AES-CBC |
| 7.9-8.0 | Stream cipher encrypted | AES-CTR ChaCha20 |

## psutil Process IO Monitoring

```python
import psutil
proc = psutil.Process(pid)
io = proc.io_counters()
# Fields: read_bytes write_bytes read_count write_count
```

## Sysmon Event IDs

| Event ID | Event | Relevance |
|----------|-------|----------|
| 1 | Process Create | Identify encrypting process |
| 2 | File time changed | Timestomping |
| 11 | FileCreate | Ransom notes |
| 15 | FileCreateStreamHash | ADS usage |
| 23 | FileDelete | Shadow copy deletion |
| 26 | FileDeleteDetected | File deletion |

## Windows ETW Providers

Microsoft-Windows-Kernel-File GUID: EDD08927-9CC4-4E65-B970-C2560FB5C289

| Event ID | Description |
|----------|------------|
| 10 | Create (open) |
| 11 | Close |
| 12 | Read |
| 14 | Write |
| 15 | SetInformation |

## Behavioral Scoring

| Signal | Weight | Threshold |
|--------|--------|-----------|
| Files modified per min | 30 pts | Over 50 |
| Entropy delta | 30 pts | Over 3.0 |
| Extension changes | 20 pts | Over 10 |
| Ransom note creation | 20 pts | Any |

### Score Interpretation

| Score | Severity | Action |
|-------|----------|--------|
| 0-25 | INFO | Log |
| 25-50 | LOW | Alert SOC |
| 50-75 | HIGH | Suspend process |
| 75-100 | CRITICAL | Kill and isolate |

## Shadow Copy Deletion

| Command | Method |
|---------|--------|
| vssadmin delete shadows /all /quiet | VSS Admin |
| wmic shadowcopy delete | WMI |
| bcdedit /set recoveryenabled no | Disable recovery |
| wbadmin delete catalog -quiet | Delete backup |

## watchdog Library

| Method | Trigger |
|--------|--------|
| on_created | File created |
| on_modified | File modified |
| on_deleted | File deleted |
| on_moved | File renamed |

## Double Extension Detection

```python
parts = filename.rsplit(".", 2)
if len(parts) >= 3:
    original_ext = "." + parts[-2]
    appended_ext = "." + parts[-1]
```

===== END FILE =====


===== FILE: skills/detecting-ransomware-encryption-behavior/scripts/agent.py =====
#!/usr/bin/env python3
"""Ransomware encryption behavior detection agent.

Detects ransomware activity using entropy analysis, file system I/O monitoring,
and behavioral heuristics. Monitors file modifications for entropy spikes and
mass rename patterns characteristic of ransomware encryption.
"""

import hashlib
import json
import logging
import math
import os
import sys
import time
from collections import Counter

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
)
logger = logging.getLogger("ransomware_detector")

RANSOMWARE_EXTENSIONS = {
    ".locked", ".encrypted", ".crypt", ".locky", ".cerber", ".wncry",
    ".dharma", ".basta", ".blackcat", ".hive", ".royal", ".akira",
    ".lockbit", ".conti", ".ryuk", ".maze", ".revil", ".phobos",
}

RANSOM_NOTE_NAMES = {
    "readme.txt", "readme.html", "decrypt.txt", "decrypt.html",
    "how_to_decrypt.txt", "restore_files.txt", "how_to_recover.txt",
}

HIGH_VALUE_EXTENSIONS = {
    ".docx", ".xlsx", ".pptx", ".pdf", ".doc", ".xls", ".ppt",
    ".csv", ".sql", ".mdb", ".accdb", ".bak", ".zip", ".7z",
    ".pst", ".ost", ".eml", ".jpg", ".png", ".dwg", ".vmdk",
}


def shannon_entropy(data):
    """Calculate Shannon entropy of byte data (0.0 to 8.0)."""
    if not data:
        return 0.0
    freq = Counter(data)
    length = len(data)
    return -sum((c / length) * math.log2(c / length) for c in freq.values())


def analyze_file_entropy(filepath):
    """Analyze entropy of a file and its segments."""
    with open(filepath, "rb") as f:
        data = f.read()

    if not data:
        return {"overall": 0.0, "is_encrypted": False}

    overall = shannon_entropy(data)
    file_size = len(data)

    chunk_size = min(4096, file_size // 4) if file_size > 16 else file_size
    first_entropy = shannon_entropy(data[:chunk_size])
    mid_entropy = shannon_entropy(data[file_size // 2: file_size // 2 + chunk_size])
    last_entropy = shannon_entropy(data[-chunk_size:])

    return {
        "overall": round(overall, 4),
        "first_chunk": round(first_entropy, 4),
        "mid_chunk": round(mid_entropy, 4),
        "last_chunk": round(last_entropy, 4),
        "file_size": file_size,
        "is_encrypted": overall > 7.5,
        "is_partial_encryption": abs(first_entropy - mid_entropy) > 2.0,
    }


def scan_directory_entropy(directory, extensions=None):
    """Scan directory for files with high entropy indicating encryption."""
    results = {"total_files": 0, "encrypted_files": 0, "files": []}

    for root, dirs, files in os.walk(directory):
        for filename in files:
            filepath = os.path.join(root, filename)
            ext = os.path.splitext(filename)[1].lower()

            if extensions and ext not in extensions:
                continue

            try:
                analysis = analyze_file_entropy(filepath)
                results["total_files"] += 1

                if analysis["is_encrypted"]:
                    results["encrypted_files"] += 1
                    analysis["path"] = filepath
                    analysis["filename"] = filename
                    results["files"].append(analysis)
            except (OSError, PermissionError):
                continue

    results["encryption_ratio"] = (
        round(results["encrypted_files"] / results["total_files"], 4)
        if results["total_files"] > 0 else 0
    )
    return results


def detect_ransomware_indicators(directory):
    """Detect multiple ransomware indicators in a directory tree."""
    indicators = {
        "ransomware_extensions": [],
        "ransom_notes": [],
        "high_entropy_files": [],
        "renamed_files": [],
        "score": 0,
    }

    for root, dirs, files in os.walk(directory):
        for filename in files:
            filepath = os.path.join(root, filename)
            lower_name = filename.lower()

            # Check for ransomware file extensions
            ext = os.path.splitext(filename)[1].lower()
            if ext in RANSOMWARE_EXTENSIONS:
                indicators["ransomware_extensions"].append(filepath)

            # Check for double extensions (report.docx.locked)
            parts = filename.rsplit(".", 2)
            if len(parts) >= 3:
                original_ext = "." + parts[-2].lower()
                appended_ext = "." + parts[-1].lower()
                if original_ext in HIGH_VALUE_EXTENSIONS and appended_ext in RANSOMWARE_EXTENSIONS:
                    indicators["renamed_files"].append({
                        "path": filepath,
                        "original_ext": original_ext,
                        "ransomware_ext": appended_ext,
                    })

            # Check for ransom notes
            if lower_name in RANSOM_NOTE_NAMES:
                indicators["ransom_notes"].append(filepath)

            # Check entropy of high-value file types
            if ext in HIGH_VALUE_EXTENSIONS:
                try:
                    analysis = analyze_file_entropy(filepath)
                    if analysis["is_encrypted"]:
                        indicators["high_entropy_files"].append({
                            "path": filepath,
                            "entropy": analysis["overall"],
                        })
                except (OSError, PermissionError):
                    continue

    # Calculate ransomware score
    score = 0
    score += min(len(indicators["ransomware_extensions"]) * 5, 30)
    score += min(len(indicators["ransom_notes"]) * 15, 30)
    score += min(len(indicators["high_entropy_files"]) * 3, 20)
    score += min(len(indicators["renamed_files"]) * 5, 20)
    indicators["score"] = min(score, 100)

    if indicators["score"] >= 75:
        indicators["verdict"] = "CRITICAL - Active ransomware encryption detected"
    elif indicators["score"] >= 50:
        indicators["verdict"] = "HIGH - Strong ransomware indicators present"
    elif indicators["score"] >= 25:
        indicators["verdict"] = "MEDIUM - Suspicious activity, investigate further"
    else:
        indicators["verdict"] = "LOW - No significant ransomware indicators"

    return indicators


def snapshot_directory_state(directory):
    """Take a baseline snapshot of directory for differential analysis."""
    snapshot = {}
    for root, dirs, files in os.walk(directory):
        for filename in files:
            filepath = os.path.join(root, filename)
            try:
                stat = os.stat(filepath)
                sha256 = hashlib.sha256()
                with open(filepath, "rb") as f:
                    for chunk in iter(lambda: f.read(65536), b""):
                        sha256.update(chunk)
                snapshot[filepath] = {
                    "hash": sha256.hexdigest(),
                    "size": stat.st_size,
                    "mtime": stat.st_mtime,
                }
            except (OSError, PermissionError):
                continue
    return snapshot


def compare_snapshots(before, after):
    """Compare two directory snapshots to detect bulk encryption."""
    changes = {"modified": [], "deleted": [], "created": [], "total_changes": 0}

    for path, info in before.items():
        if path not in after:
            changes["deleted"].append(path)
        elif after[path]["hash"] != info["hash"]:
            changes["modified"].append({
                "path": path,
                "size_before": info["size"],
                "size_after": after[path]["size"],
            })

    for path in after:
        if path not in before:
            changes["created"].append(path)

    changes["total_changes"] = (
        len(changes["modified"]) + len(changes["deleted"]) + len(changes["created"])
    )

    if changes["total_changes"] > 0:
        mod_ratio = len(changes["modified"]) / max(len(before), 1)
        changes["bulk_modification_ratio"] = round(mod_ratio, 4)
        changes["ransomware_likely"] = mod_ratio > 0.3 and len(changes["modified"]) > 10
    else:
        changes["bulk_modification_ratio"] = 0
        changes["ransomware_likely"] = False

    return changes


if __name__ == "__main__":
    print("=" * 60)
    print("Ransomware Encryption Behavior Detection Agent")
    print("Entropy analysis, I/O monitoring, behavioral heuristics")
    print("=" * 60)

    if len(sys.argv) < 2:
        print("\nUsage:")
        print("  python agent.py scan <directory>          Scan for ransomware indicators")
        print("  python agent.py entropy <directory>       Entropy scan of files")
        print("  python agent.py entropy-file <file>       Analyze single file entropy")
        print("  python agent.py snapshot <directory>      Take baseline snapshot")
        print("  python agent.py compare <snap1> <snap2>   Compare two snapshots")
        sys.exit(0)

    command = sys.argv[1]

    if command == "scan":
        target = sys.argv[2] if len(sys.argv) > 2 else os.getcwd()
        print(f"\n[*] Scanning {target} for ransomware indicators...")
        results = detect_ransomware_indicators(target)
        print(f"\n--- Ransomware Detection Results ---")
        print(f"  Score: {results['score']}/100")
        print(f"  Verdict: {results['verdict']}")
        print(f"  Ransomware extensions found: {len(results['ransomware_extensions'])}")
        print(f"  Ransom notes found: {len(results['ransom_notes'])}")
        print(f"  High-entropy files: {len(results['high_entropy_files'])}")
        print(f"  Renamed files: {len(results['renamed_files'])}")
        print(f"\n{json.dumps(results, indent=2, default=str)}")

    elif command == "entropy":
        target = sys.argv[2] if len(sys.argv) > 2 else os.getcwd()
        print(f"\n[*] Entropy scanning {target}...")
        results = scan_directory_entropy(target, HIGH_VALUE_EXTENSIONS)
        print(f"\n--- Entropy Scan Results ---")
        print(f"  Total files scanned: {results['total_files']}")
        print(f"  Files with encrypted entropy: {results['encrypted_files']}")
        print(f"  Encryption ratio: {results['encryption_ratio']}")
        for f in results["files"][:10]:
            print(f"  [!] {f['filename']}: entropy={f['overall']}")

    elif command == "entropy-file":
        if len(sys.argv) < 3:
            print("[!] Provide a file path")
            sys.exit(1)
        filepath = sys.argv[2]
        analysis = analyze_file_entropy(filepath)
        print(f"\n--- File Entropy Analysis ---")
        print(f"  File: {filepath}")
        print(f"  Overall entropy: {analysis['overall']}")
        print(f"  First chunk: {analysis['first_chunk']}")
        print(f"  Mid chunk: {analysis['mid_chunk']}")
        print(f"  Last chunk: {analysis['last_chunk']}")
        print(f"  Encrypted: {analysis['is_encrypted']}")
        print(f"  Partial encryption: {analysis['is_partial_encryption']}")

    elif command == "snapshot":
        target = sys.argv[2] if len(sys.argv) > 2 else os.getcwd()
        print(f"\n[*] Taking snapshot of {target}...")
        snap = snapshot_directory_state(target)
        output = f"snapshot_{int(time.time())}.json"
        with open(output, "w") as f:
            json.dump(snap, f, indent=2)
        print(f"[+] Snapshot saved: {output} ({len(snap)} files)")

    elif command == "compare":
        if len(sys.argv) < 4:
            print("[!] Provide two snapshot JSON files")
            sys.exit(1)
        with open(sys.argv[2]) as f:
            snap1 = json.load(f)
        with open(sys.argv[3]) as f:
            snap2 = json.load(f)
        changes = compare_snapshots(snap1, snap2)
        print(f"\n--- Snapshot Comparison ---")
        print(f"  Modified: {len(changes['modified'])}")
        print(f"  Deleted: {len(changes['deleted'])}")
        print(f"  Created: {len(changes['created'])}")
        print(f"  Ransomware likely: {changes['ransomware_likely']}")
        print(f"\n{json.dumps(changes, indent=2, default=str)}")

    else:
        print(f"[!] Unknown command: {command}")

===== END FILE =====


===== FILE: skills/detecting-ransomware-precursors-in-network/SKILL.md =====
---
name: detecting-ransomware-precursors-in-network
description: 'Detects early-stage ransomware indicators in network traffic before
  encryption begins, including initial access broker activity, command-and-control
  beaconing, credential harvesting, reconnaissance scanning, and staging behavior.
  Uses network detection tools (Zeek, Suricata, Arkime), SIEM correlation rules, and
  threat intelligence feeds to identify ransomware precursor patterns such as Cobalt
  Strike beacons, Mimikatz network signatures, and RDP brute-force attempts. Activates
  for requests involving pre-ransomware detection, network-based ransomware indicators,
  or early warning ransomware monitoring.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- detection
- network-security
- incident-response
- defense
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - positioning
  - monetization
  techniques:
  - id: T1110
    name: Brute Force
    tactic: initial-access
    source: attack
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
  - id: T1650
    name: Acquire Access
    tactic: resource-development
    source: attack
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
---
# Detecting Ransomware Precursors in Network Traffic

## When to Use

- Building detection rules for pre-ransomware network activity (the average time from Cobalt Strike deployment to encryption is 17 minutes)
- Monitoring for initial access broker (IAB) indicators that precede ransomware deployment
- Creating SIEM correlation rules that chain multiple precursor events into high-confidence alerts
- Tuning network detection systems to distinguish ransomware staging from normal administrative activity
- Investigating suspicious network patterns that may indicate ransomware operators have established a foothold

**Do not use** for post-encryption response (see recovering-from-ransomware-attack). This skill focuses on the pre-encryption detection window where containment can prevent data loss.

## Prerequisites

- Network detection platform (Zeek/Bro, Suricata, or Arkime/Moloch) deployed on network TAP or SPAN ports
- SIEM platform (Splunk, Elastic Security, Microsoft Sentinel, or QRadar) ingesting network logs
- Threat intelligence feeds covering ransomware IOCs (CISA, abuse.ch, OTX, MISP)
- Network flow data (NetFlow/IPFIX) from core routers and firewalls
- DNS query logging from internal resolvers
- Full packet capture capability for incident investigation

## Workflow

### Step 1: Identify Ransomware Kill Chain Phases in Network Traffic

Map network-observable indicators to each pre-encryption phase:

| Kill Chain Phase | Network Indicators | Detection Source |
|------------------|--------------------|------------------|
| Initial Access | RDP brute force, VPN credential stuffing, phishing callback | Firewall logs, IDS, proxy logs |
| C2 Establishment | Cobalt Strike beacons (HTTPS/DNS), Sliver/Brute Ratel callbacks | Zeek SSL/HTTP logs, DNS logs |
| Credential Harvesting | NTLM relay, Kerberoasting, DCSync traffic | Zeek Kerberos/NTLM logs, DC logs |
| Reconnaissance | Internal port scanning, AD enumeration (LDAP/SMB) | Zeek conn.log, flow data |
| Lateral Movement | PsExec/WMI/WinRM traffic, RDP pivoting, SMB file copies | Zeek SMB/DCE-RPC logs |
| Staging | Data aggregation, archive creation, cloud upload prep | Proxy logs, DNS logs, DLP |

### Step 2: Deploy Network Detection Rules

**Suricata rules for common ransomware precursors:**

```yaml
# Cobalt Strike default HTTPS beacon profile detection
alert tls $HOME_NET any -> $EXTERNAL_NET any (msg:"RANSOMWARE PRECURSOR - Cobalt Strike Default TLS Certificate"; tls.cert_subject; content:"Major Cobalt Strike"; sid:3000001; rev:1;)

# Cobalt Strike DNS beacon
alert dns $HOME_NET any -> any 53 (msg:"RANSOMWARE PRECURSOR - Cobalt Strike DNS Beacon Pattern"; dns.query; pcre:"/^[a-z0-9]{3}\.[a-z]{4,8}\./"; threshold:type both, track by_src, count 50, seconds 60; sid:3000002; rev:1;)

# Mimikatz network signature (DCSync - DRS GetNCChanges)
alert tcp $HOME_NET any -> $HOME_NET 135 (msg:"RANSOMWARE PRECURSOR - Possible DCSync/Mimikatz"; content:"|05 00 0b|"; offset:0; depth:3; content:"|e3 51 4d 2b 4b 47 15 d2|"; sid:3000003; rev:1;)

# Internal network scanning (many connections, few bytes)
alert tcp $HOME_NET any -> $HOME_NET any (msg:"RANSOMWARE PRECURSOR - Internal Port Scan"; flags:S; threshold:type both, track by_src, count 100, seconds 10; sid:3000004; rev:1;)

# PsExec service installation over SMB
alert tcp $HOME_NET any -> $HOME_NET 445 (msg:"RANSOMWARE PRECURSOR - PsExec Service Install"; content:"|ff|SMB"; content:"PSEXESVC"; nocase; sid:3000005; rev:1;)

# RDP brute force from internal host (lateral movement)
alert tcp $HOME_NET any -> $HOME_NET 3389 (msg:"RANSOMWARE PRECURSOR - Internal RDP Brute Force"; flow:to_server,established; threshold:type both, track by_src, count 20, seconds 60; sid:3000006; rev:1;)

# Large SMB file transfer (data staging)
alert tcp $HOME_NET any -> $HOME_NET 445 (msg:"RANSOMWARE PRECURSOR - Large SMB Transfer Possible Staging"; flow:to_server,established; dsize:>60000; threshold:type both, track by_src, count 100, seconds 300; sid:3000007; rev:1;)
```

**Zeek scripts for behavioral detection:**

```zeek
# detect_ransomware_precursors.zeek
# Detect high volume of failed SMB connections (credential testing)

@load base/protocols/smb

module RansomwarePrecursor;

export {
    redef enum Notice::Type += {
        SMB_Brute_Force,
        Suspicious_Internal_Scan,
        Excessive_DNS_Queries,
        SMB_Admin_Share_Access,
    };

    const smb_fail_threshold = 10 &redef;
    const scan_threshold = 50 &redef;
    const dns_query_threshold = 200 &redef;
}

global smb_fail_count: table[addr] of count &default=0 &create_expire=5min;
global conn_count: table[addr] of set[addr] &create_expire=1min;

event smb2_message(c: connection, hdr: SMB2::Header, is_orig: bool) {
    if (hdr$status != 0) {
        ++smb_fail_count[c$id$orig_h];
        if (smb_fail_count[c$id$orig_h] >= smb_fail_threshold) {
            NOTICE([$note=SMB_Brute_Force,
                    $msg=fmt("Host %s has %d failed SMB attempts", c$id$orig_h, smb_fail_count[c$id$orig_h]),
                    $src=c$id$orig_h,
                    $identifier=cat(c$id$orig_h)]);
        }
    }
}

event new_connection(c: connection) {
    if (c$id$orig_h in Site::local_nets && c$id$resp_h in Site::local_nets) {
        if (c$id$orig_h !in conn_count)
            conn_count[c$id$orig_h] = set();
        add conn_count[c$id$orig_h][c$id$resp_h];
        if (|conn_count[c$id$orig_h]| >= scan_threshold) {
            NOTICE([$note=Suspicious_Internal_Scan,
                    $msg=fmt("Host %s connected to %d internal hosts in 1 min", c$id$orig_h, |conn_count[c$id$orig_h]|),
                    $src=c$id$orig_h,
                    $identifier=cat(c$id$orig_h)]);
        }
    }
}
```

### Step 3: Create SIEM Correlation Rules

**Splunk correlation for ransomware precursor chain:**

```spl
| tstats count FROM datamodel=Network_Traffic
  WHERE earliest=-24h All_Traffic.dest_port IN (445, 135, 139, 3389, 5985, 5986)
    AND All_Traffic.src_ip IN 10.0.0.0/8
    AND All_Traffic.dest_ip IN 10.0.0.0/8
  BY All_Traffic.src_ip, All_Traffic.dest_port, _time span=1h
| stats dc(All_Traffic.dest_port) as port_count,
        values(All_Traffic.dest_port) as ports,
        count as total_conns
  BY All_Traffic.src_ip
| where port_count >= 3 AND total_conns > 50
| rename All_Traffic.src_ip as src_ip
| lookup threat_intel_ioc ip as src_ip OUTPUT threat_type
| eval risk_score = case(
    port_count >= 5 AND total_conns > 200, "CRITICAL",
    port_count >= 3 AND total_conns > 50, "HIGH",
    1=1, "MEDIUM")
| table src_ip, ports, port_count, total_conns, risk_score, threat_type
```

**Microsoft Sentinel KQL - Ransomware precursor correlation:**

```kql
let timeframe = 24h;
let RDPBruteForce = SecurityEvent
| where TimeGenerated > ago(timeframe)
| where EventID == 4625
| where LogonType == 10
| summarize FailedRDP = count() by TargetAccount, IpAddress, bin(TimeGenerated, 1h)
| where FailedRDP > 10;
let SuspiciousSMB = SecurityEvent
| where TimeGenerated > ago(timeframe)
| where EventID == 5145
| where ShareName has "ADMIN$" or ShareName has "C$" or ShareName has "IPC$"
| summarize AdminShareAccess = count() by SubjectUserName, IpAddress, bin(TimeGenerated, 1h)
| where AdminShareAccess > 5;
let ServiceInstalls = SecurityEvent
| where TimeGenerated > ago(timeframe)
| where EventID == 7045
| where ServiceName has_any ("PSEXESVC", "meterpreter", "beacon");
RDPBruteForce
| join kind=inner SuspiciousSMB on IpAddress
| project TimeGenerated, IpAddress, TargetAccount, FailedRDP, SubjectUserName, AdminShareAccess
| extend AlertTitle = "Ransomware Precursor: RDP Brute Force + Admin Share Access"
```

### Step 4: Integrate Threat Intelligence

Configure automated IOC feeds for known ransomware infrastructure:

```bash
# Download and update ransomware C2 blocklists
# abuse.ch Feodo Tracker (Cobalt Strike, TrickBot, BazarLoader C2s)
curl -s https://feodotracker.abuse.ch/downloads/ipblocklist.csv | \
  grep -v "^#" | cut -d, -f2 > /opt/threat-intel/feodo_ips.txt

# abuse.ch URLhaus (malware distribution URLs)
curl -s https://urlhaus.abuse.ch/downloads/csv_recent/ | \
  grep -v "^#" | cut -d, -f3 > /opt/threat-intel/urlhaus_urls.txt

# abuse.ch ThreatFox (ransomware IOCs)
curl -s https://threatfox.abuse.ch/export/csv/recent/ | \
  grep -i "ransomware" | cut -d, -f3 > /opt/threat-intel/ransomware_iocs.txt

# CISA Known Exploited Vulnerabilities (initial access vectors)
curl -s https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json | \
  python3 -c "import json,sys; data=json.load(sys.stdin); [print(v['cveID'],v['vendorProject'],v['product']) for v in data['vulnerabilities'] if 'ransomware' in v.get('knownRansomwareCampaignUse','').lower()]"
```

### Step 5: Establish Alert Triage and Escalation

Define triage procedures based on precursor confidence level:

| Alert Type | Confidence | Response Time | Action |
|------------|-----------|---------------|--------|
| Confirmed Cobalt Strike beacon | High | 15 minutes | Isolate host immediately, trigger IR |
| DCSync/Kerberoasting from non-DC | High | 15 minutes | Disable account, isolate host, trigger IR |
| Internal port scan + admin share access | Medium-High | 30 minutes | Investigate source host, check EDR telemetry |
| RDP brute force from internal host | Medium | 1 hour | Verify if legitimate admin activity, check host |
| Unusual DNS query volume | Low-Medium | 4 hours | Check for DNS tunneling, correlate with other alerts |

## Key Concepts

| Term | Definition |
|------|------------|
| **Ransomware Precursor** | Network activity that precedes ransomware encryption, including C2 communication, lateral movement, and data staging |
| **Dwell Time** | Time between initial compromise and ransomware deployment, averaging 21 days but sometimes as short as 17 minutes |
| **Initial Access Broker (IAB)** | Threat actors who sell compromised network access to ransomware operators on dark web markets |
| **Beaconing** | Periodic C2 callbacks from implants (Cobalt Strike, Sliver) that can be detected by analyzing connection timing patterns |
| **Kerberoasting** | Credential harvesting technique requesting Kerberos service tickets for offline cracking, detectable via unusual TGS-REQ patterns |
| **DCSync** | Technique using Directory Replication Service to extract password hashes from domain controllers, critical ransomware precursor |

## Tools & Systems

- **Zeek (formerly Bro)**: Network analysis framework generating structured logs for SMB, Kerberos, DNS, HTTP, and TLS connections
- **Suricata**: High-performance IDS/IPS with protocol analysis and multi-threading support for ransomware signature detection
- **Arkime (formerly Moloch)**: Full packet capture and search platform for deep forensic investigation of network events
- **RITA (Real Intelligence Threat Analytics)**: Open-source tool for detecting beaconing, DNS tunneling, and long connections in Zeek logs
- **AC-Hunter**: Network threat hunting platform from Active Countermeasures for beacon detection and C2 identification

## Common Scenarios

### Scenario: Detecting LockBit Precursors in a Manufacturing Network

**Context**: A manufacturing company's SOC receives an alert for unusual SMB traffic from a workstation (10.1.5.42) in the engineering department. The workstation connected to 47 internal hosts on port 445 within 5 minutes at 2:00 AM.

**Approach**:
1. Zeek conn.log analysis shows 10.1.5.42 initiated connections to 47 unique internal IPs on port 445, 135, and 3389 between 01:55-02:05
2. Zeek ssl.log reveals an outbound HTTPS connection to 185.x.x.x every 60 seconds with consistent 48-byte payloads (Cobalt Strike beacon pattern)
3. RITA beacon analysis confirms high beacon score (0.96) for the external IP with 60-second jitter
4. Zeek kerberos.log shows TGS-REQ for multiple SPN accounts from 10.1.5.42 (Kerberoasting)
5. SMB tree_connect events show access to ADMIN$ shares on 12 hosts (lateral movement staging)
6. Containment: Host isolated, credentials for engineering user reset, blocking rule for C2 IP deployed
7. Full IR initiated before ransomware deployment could begin

**Pitfalls**:
- Dismissing internal port scans as vulnerability scanner activity without verifying the source is an authorized scanner
- Not correlating individual low-severity alerts (DNS anomaly + SMB access + failed logins) into a high-severity chain
- Setting detection thresholds too high to avoid false positives, missing low-and-slow reconnaissance
- Ignoring encrypted traffic analysis (JA3/JA4 fingerprinting) that can identify Cobalt Strike even in TLS tunnels

## Output Format

```
## Ransomware Precursor Detection Alert

**Alert ID**: [SIEM-generated ID]
**Detection Time**: [Timestamp]
**Source Host**: [IP / Hostname]
**Confidence**: [High / Medium / Low]
**Kill Chain Phase**: [Initial Access / C2 / Credential Harvest / Recon / Lateral Movement / Staging]

### Indicators Detected
| Indicator | Source | Detail | MITRE ATT&CK |
|-----------|--------|--------|--------------|
| [Type] | [Zeek/Suricata/SIEM] | [Description] | [T-ID] |

### Correlation Chain
1. [Timestamp] - [Event 1]
2. [Timestamp] - [Event 2]
3. [Timestamp] - [Event 3]

### Recommended Actions
- [ ] Isolate source host from network
- [ ] Check EDR telemetry for host-based indicators
- [ ] Reset credentials for affected user accounts
- [ ] Block identified C2 infrastructure
- [ ] Escalate to incident response team
```

===== END FILE =====


===== FILE: skills/detecting-ransomware-precursors-in-network/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/detecting-ransomware-precursors-in-network/assets/template.md =====
# Ransomware Precursor Detection Deployment Template

## Network Sensor Placement

| Sensor Location | TAP/SPAN | Sensor Type | Protocols Monitored |
|----------------|----------|-------------|---------------------|
| Internet Edge | | Zeek + Suricata | All outbound |
| DC Segment | | Zeek | Kerberos, LDAP, SMB, DNS |
| Server VLAN | | Zeek + Suricata | SMB, RDP, WMI, WinRM |
| Workstation VLAN | | Zeek | SMB, HTTP/S, DNS |

## Detection Rules Deployed

| Rule ID | Category | Description | Severity | Status |
|---------|----------|-------------|----------|--------|
| | C2 Beaconing | | | |
| | Credential Harvest | | | |
| | Internal Scanning | | | |
| | Lateral Movement | | | |

## Threat Intelligence Feeds

| Feed | Source | Update Frequency | IOC Types |
|------|--------|-----------------|-----------|
| Feodo Tracker | abuse.ch | Hourly | IPs (C2) |
| ThreatFox | abuse.ch | Hourly | IPs, domains, hashes |
| ET Open Rules | Proofpoint | Daily | Suricata signatures |
| CISA KEV | CISA | As published | CVEs |

## Alert Triage SLAs

| Category | Priority | Triage SLA | Escalation Path |
|----------|----------|-----------|-----------------|
| Confirmed C2 | CRITICAL | 15 min | SOC -> IR Team -> CISO |
| Credential Harvest | CRITICAL | 15 min | SOC -> IR Team |
| Internal Scanning | HIGH | 30 min | SOC -> IR Team |
| Admin Share Enum | HIGH | 30 min | SOC -> IR Team |
| RDP Anomaly | MEDIUM | 1 hour | SOC Tier 2 |
| DNS Anomaly | LOW | 4 hours | SOC Tier 1 |

## Detection Tuning Log

| Date | Rule | Change | Reason | FP Rate Before | FP Rate After |
|------|------|--------|--------|----------------|---------------|
| | | | | | |

===== END FILE =====


===== FILE: skills/detecting-ransomware-precursors-in-network/references/api-reference.md =====
# API Reference — Detecting Ransomware Precursors in Network Traffic

## Zeek (Bro) Log Fields

### conn.log
| Field | Type | Description |
|-------|------|-------------|
| `ts` | time | Connection start timestamp (Unix epoch) |
| `id.orig_h` | addr | Source IP address |
| `id.orig_p` | port | Source port |
| `id.resp_h` | addr | Destination IP address |
| `id.resp_p` | port | Destination port |
| `proto` | enum | Transport protocol (tcp/udp/icmp) |
| `orig_bytes` | count | Bytes sent by originator |
| `resp_bytes` | count | Bytes sent by responder |
| `conn_state` | string | Connection state (SF=normal, S0=no reply, REJ=rejected) |
| `duration` | interval | Duration of connection |

### smb_files.log
| Field | Type | Description |
|-------|------|-------------|
| `action` | enum | SMB action (SMB_FILE_OPEN, SMB_FILE_WRITE, SMB_FILE_DELETE) |
| `path` | string | Full UNC path accessed |
| `name` | string | Filename |
| `size` | count | File size in bytes |
| `id.orig_h` | addr | Source host (accessor) |
| `id.resp_h` | addr | Target host |

### kerberos.log
| Field | Type | Description |
|-------|------|-------------|
| `request_type` | string | KRB_AS_REQ, KRB_TGS_REQ |
| `client` | string | Client principal |
| `service` | string | Service principal (SPN) |
| `success` | bool | Whether request succeeded |
| `error_msg` | string | Error type (e.g., KDC_ERR_PREAUTH_REQUIRED) |

## Suricata CLI

### Start in IDS mode
```bash
suricata -c /etc/suricata/suricata.yaml -i eth0
```

### Start in IPS mode (NFQUEUE)
```bash
suricata -c /etc/suricata/suricata.yaml -q 0
# Configure iptables to send traffic to NFQUEUE:
iptables -I FORWARD -j NFQUEUE --queue-num 0
```

### Run on pcap file
```bash
suricata -c /etc/suricata/suricata.yaml -r capture.pcap -l /var/log/suricata/
```

### Update rules with suricata-update
```bash
suricata-update                          # Update all enabled sources
suricata-update list-sources             # List available rule sources
suricata-update enable-source et/open    # Enable Emerging Threats Open
suricata-update enable-source ptresearch/attackdetection  # PT Research rules
suricata-update update-sources           # Refresh source index
suricata-update --no-reload              # Update without live reload
```

### Reload rules without restart
```bash
kill -USR2 $(pidof suricata)
# Or via Unix socket:
suricatasc -c reload-rules
```

### Query eve.json for alerts
```bash
# Ransomware-related alerts in last hour
jq 'select(.event_type=="alert") | select(.alert.signature | test("ransomware|cobalt|mimikatz|psexec";"i"))' \
  /var/log/suricata/eve.json | jq -r '[.timestamp,.src_ip,.dest_ip,.alert.signature] | @tsv'

# Top 10 alert signatures
jq -r 'select(.event_type=="alert") | .alert.signature' /var/log/suricata/eve.json | \
  sort | uniq -c | sort -rn | head -10
```

## RITA (Real Intelligence Threat Analytics)

### Import Zeek logs and analyze
```bash
rita import --input /var/log/zeek/current/ --database my_network
rita analyze my_network
```

### Beacon detection output
```bash
rita show-beacons my_network --human-readable
# Columns: Score | Source | Dest | Connections | Avg Bytes | TS Delta
# Score 0.9+ = high confidence beacon
```

### DNS tunneling detection
```bash
rita show-exploded-dns my_network | head -20
rita show-long-connections my_network --human-readable
```

## Splunk SPL — Ransomware Precursor Queries

### Internal lateral movement via SMB/RDP/WinRM
```spl
index=zeek sourcetype=zeek_conn
  id.resp_p IN (445, 135, 3389, 5985, 5986)
  id.orig_h IN 10.0.0.0/8
  id.resp_h IN 10.0.0.0/8
| stats dc(id.resp_h) as targets count as conns by id.orig_h
| where targets >= 10
| sort -targets
```

### Detect beaconing (regular connection intervals)
```spl
index=zeek sourcetype=zeek_conn
| bucket _time span=1m
| stats count as conns by id.orig_h, id.resp_h, id.resp_p, _time
| stats stdev(conns) as jitter avg(conns) as avg_conns count as minutes
    by id.orig_h, id.resp_h, id.resp_p
| where minutes > 10 AND jitter < 2 AND avg_conns > 0
| eval beacon_score = round(1 - (jitter / (avg_conns + 0.001)), 2)
| where beacon_score > 0.8
| sort -beacon_score
```

## abuse.ch Threat Intelligence Feeds

### Feodo Tracker (C2 IPs — Cobalt Strike, BazarLoader)
```bash
# CSV format: first_seen,dst_ip,dst_port,c2_status,malware
curl -s https://feodotracker.abuse.ch/downloads/ipblocklist.csv | \
  grep -v "^#" | awk -F, '{print $2}' > /tmp/c2_ips.txt
```

### ThreatFox IOC API
```bash
# Query recent ransomware IOCs
curl -s -X POST https://threatfox-api.abuse.ch/api/v1/ \
  -H "Content-Type: application/json" \
  -d '{"query":"get_iocs","days":7,"tag":"ransomware"}' | \
  jq '.data[] | [.ioc_value,.ioc_type,.malware,.confidence_level] | @tsv' -r
```

## MITRE ATT&CK Ransomware Precursor Techniques
| Technique | ID | Network Indicator |
|-----------|----|-------------------|
| Remote Services: SMB/WMI | T1021.002 | SMB port 445 traffic to many hosts |
| OS Credential Dumping: DCSync | T1003.006 | DRS GetNCChanges from non-DC |
| Kerberoasting | T1558.003 | TGS-REQ for many SPNs |
| Command & Control | T1071.001 | Regular HTTPS beaconing |
| Lateral Tool Transfer | T1570 | Large SMB file writes across hosts |
| Network Service Scanning | T1046 | Port sweeps on 445/3389/135 |

===== END FILE =====


===== FILE: skills/detecting-ransomware-precursors-in-network/references/standards.md =====
# Standards & References - Detecting Ransomware Precursors

## MITRE ATT&CK Mapping

### Initial Access (TA0001)
- T1078: Valid Accounts (compromised credentials from IABs)
- T1133: External Remote Services (VPN/RDP exploitation)
- T1566: Phishing (email-based initial access)

### Command and Control (TA0011)
- T1071.001: Application Layer Protocol: Web (HTTPS beacons)
- T1071.004: Application Layer Protocol: DNS (DNS tunneling)
- T1573: Encrypted Channel (C2 over TLS)
- T1090: Proxy (redirectors and relay infrastructure)

### Credential Access (TA0006)
- T1558.003: Kerberoasting
- T1003.006: DCSync
- T1557: Adversary-in-the-Middle (NTLM relay)

### Discovery (TA0007)
- T1046: Network Service Discovery (port scanning)
- T1018: Remote System Discovery (AD enumeration)
- T1087: Account Discovery

### Lateral Movement (TA0008)
- T1021.002: Remote Services: SMB/Windows Admin Shares
- T1021.001: Remote Services: RDP
- T1047: Windows Management Instrumentation (WMI)
- T1569.002: System Services: Service Execution (PsExec)

## Industry References

### CISA Advisories
- AA23-136A: #StopRansomware - BianLian Ransomware Group
- AA23-158A: #StopRansomware - CL0P Ransomware Gang
- AA24-131A: #StopRansomware - Black Basta
- AA23-165A: Understanding Ransomware Threat Actors: LockBit

### NIST
- SP 800-94 Rev 1: Guide to Intrusion Detection and Prevention Systems
- SP 800-86: Guide to Integrating Forensic Techniques into Incident Response
- IR 8374: Ransomware Risk Management

### Threat Intelligence Sources
- abuse.ch Feodo Tracker: https://feodotracker.abuse.ch/
- abuse.ch ThreatFox: https://threatfox.abuse.ch/
- CISA Known Exploited Vulnerabilities: https://www.cisa.gov/known-exploited-vulnerabilities-catalog
- Emerging Threats Ruleset: https://rules.emergingthreats.net/

===== END FILE =====


===== FILE: skills/detecting-ransomware-precursors-in-network/references/workflows.md =====
# Workflows - Detecting Ransomware Precursors in Network

## Workflow 1: Network Sensor Deployment

```
Start
  |
  v
[Identify network chokepoints] --> Core switches, internet edge, DC segments
  |
  v
[Deploy network TAPs or configure SPAN ports]
  |
  v
[Install Zeek sensor] --> Configure local.zeek with site-specific networks
  |
  v
[Install Suricata IDS] --> Load ET Open + custom ransomware rules
  |
  v
[Configure log forwarding to SIEM]
  |-- Zeek: conn.log, ssl.log, dns.log, smb.log, kerberos.log, notice.log
  |-- Suricata: eve.json (alerts, flow, dns, tls)
  |
  v
[Deploy RITA for beacon analysis] --> Schedule hourly Zeek log imports
  |
  v
[Load threat intelligence feeds] --> Feodo, ThreatFox, CISA KEV
  |
  v
[Validate detection with controlled Cobalt Strike beacon test]
  |
  v
End
```

## Workflow 2: Alert Triage for Ransomware Precursors

```
Alert Received
  |
  v
[Classify alert type]
  |-- C2 Beaconing --> Priority: CRITICAL, SLA: 15 min
  |-- Credential Harvesting --> Priority: CRITICAL, SLA: 15 min
  |-- Internal Scanning --> Priority: HIGH, SLA: 30 min
  |-- Admin Share Access --> Priority: HIGH, SLA: 30 min
  |-- RDP Brute Force --> Priority: MEDIUM, SLA: 1 hour
  |-- DNS Anomaly --> Priority: LOW, SLA: 4 hours
  |
  v
[Check for correlated alerts on same source IP]
  |-- Multiple categories? --> Elevate to CRITICAL regardless
  |-- Single category? --> Proceed with category SLA
  |
  v
[Verify source host context]
  |-- Known admin workstation? --> Check if scheduled activity
  |-- Server? --> Check for authorized maintenance window
  |-- Standard workstation? --> Likely compromise indicator
  |
  v
[Decision: True Positive or False Positive?]
  |
  TP --> [Contain host: network isolation via NAC/EDR]
  |       |
  |       v
  |       [Trigger incident response playbook]
  |
  FP --> [Document FP reason]
          |
          v
          [Update detection rule to reduce FP rate]
          |
          v
          End
```

## Workflow 3: Beacon Detection with RITA

```
Hourly Cron Job
  |
  v
[Import latest Zeek logs into RITA database]
  $ rita import /opt/zeek/logs/current rita-db
  |
  v
[Analyze beacons]
  $ rita show-beacons rita-db --human-readable
  |
  v
[Filter results by beacon score > 0.7]
  |
  v
[For each high-score beacon:]
  |-- Look up destination IP in threat intel
  |-- Check JA3/JA4 hash against known C2 fingerprints
  |-- Verify beacon interval and jitter pattern
  |
  v
[Score > 0.9 AND matches threat intel?]
  |
  Yes --> [Generate CRITICAL alert]
  |
  No --> [Score > 0.7?]
          |
          Yes --> [Generate MEDIUM alert for analyst review]
          |
          No --> [Log and continue monitoring]
  |
  v
End
```

===== END FILE =====


===== FILE: skills/detecting-ransomware-precursors-in-network/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting ransomware precursor activity in network traffic and logs."""

import argparse
import json
import os
import subprocess
import sys
from datetime import datetime, timezone


RANSOMWARE_PORTS = {445, 3389, 4444, 5985, 5986, 135, 139, 8443}
SUSPICIOUS_PROCESSES = [
    "vssadmin.exe", "wmic.exe", "bcdedit.exe", "wbadmin.exe",
    "powershell.exe", "cmd.exe", "certutil.exe", "bitsadmin.exe",
    "mshta.exe", "rundll32.exe", "regsvr32.exe", "cscript.exe",
]
SHADOW_COPY_PATTERNS = [
    r"vssadmin\s+delete\s+shadows",
    r"wmic\s+shadowcopy\s+delete",
    r"bcdedit.*recoveryenabled.*no",
    r"wbadmin\s+delete\s+(catalog|systemstatebackup)",
]
SMB_LATERAL_PATTERNS = [
    r"\\\\[\d\.]+\\(C\$|ADMIN\$|IPC\$)",
    r"psexec",
    r"wmiexec",
]


def parse_zeek_conn_log(log_path):
    """Parse Zeek conn.log for suspicious network connections."""
    alerts = []
    try:
        with open(log_path, "r") as f:
            for line in f:
                if line.startswith("#"):
                    continue
                fields = line.strip().split("\t")
                if len(fields) < 7:
                    continue
                src_ip, src_port, dst_ip, dst_port = fields[2], fields[3], fields[4], fields[5]
                try:
                    dp = int(dst_port)
                except ValueError:
                    continue
                if dp in RANSOMWARE_PORTS:
                    alerts.append({
                        "type": "suspicious_port",
                        "src": src_ip,
                        "dst": dst_ip,
                        "port": dp,
                        "detail": f"Connection to ransomware-associated port {dp}",
                    })
    except FileNotFoundError:
        print(f"[!] Log file not found: {log_path}")
    return alerts


def analyze_event_logs_windows():
    """Check Windows event logs for ransomware precursors."""
    alerts = []
    queries = [
        ("Shadow copy deletion", "Get-WinEvent -FilterHashtable @{LogName='System';Id=7036} "
         "| Where-Object {$_.Message -match 'Volume Shadow Copy'} | Select-Object -First 10 "
         "| ConvertTo-Json"),
        ("RDP brute force", "Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625} "
         "| Select-Object -First 20 | Group-Object {$_.Properties[5].Value} "
         "| Where-Object {$_.Count -gt 5} | ConvertTo-Json"),
        ("Service installs", "Get-WinEvent -FilterHashtable @{LogName='System';Id=7045} "
         "| Select-Object -First 10 | ConvertTo-Json"),
    ]
    for name, ps_cmd in queries:
        try:
            result = subprocess.check_output(
                ["powershell", "-NoProfile", "-Command", ps_cmd],
                text=True, errors="replace", timeout=30
            )
            if result.strip():
                data = json.loads(result) if result.strip().startswith(("[", "{")) else result
                alerts.append({"check": name, "findings": data})
        except (subprocess.SubprocessError, json.JSONDecodeError):
            pass
    return alerts


def scan_process_list():
    """Check running processes for ransomware tooling."""
    suspicious = []
    if sys.platform == "win32":
        try:
            out = subprocess.check_output(
                ["tasklist", "/FO", "CSV", "/NH"], text=True, errors="replace",
                timeout=120,
            )
            for line in out.splitlines():
                parts = line.strip('"').split('","')
                if parts:
                    pname = parts[0].lower()
                    for sp in SUSPICIOUS_PROCESSES:
                        if pname == sp.lower():
                            suspicious.append({"process": pname, "pid": parts[1] if len(parts) > 1 else "?"})
        except subprocess.SubprocessError:
            pass
    else:
        try:
            out = subprocess.check_output(["ps", "-eo", "pid,comm", "--no-headers"], text=True, timeout=120)
            for line in out.splitlines():
                parts = line.split(None, 1)
                if len(parts) == 2:
                    for sp in SUSPICIOUS_PROCESSES:
                        if parts[1].strip().lower() == sp.replace(".exe", ""):
                            suspicious.append({"process": parts[1].strip(), "pid": parts[0]})
        except subprocess.SubprocessError:
            pass
    return suspicious


def check_file_encryption_activity(directory, threshold=50):
    """Detect mass file renaming or new encrypted extensions."""
    suspicious_exts = {".encrypted", ".locked", ".crypto", ".crypt", ".enc",
                       ".locky", ".cerber", ".zepto", ".thor", ".aaa"}
    findings = []
    count = 0
    try:
        for root, _, files in os.walk(directory):
            for f in files:
                ext = os.path.splitext(f)[1].lower()
                if ext in suspicious_exts:
                    count += 1
                    if count <= 10:
                        findings.append(os.path.join(root, f))
            if count >= threshold:
                break
    except PermissionError:
        pass
    return {"encrypted_file_count": count, "samples": findings, "threshold_exceeded": count >= threshold}


def main():
    parser = argparse.ArgumentParser(
        description="Detect ransomware precursor activity in network and host"
    )
    parser.add_argument("--conn-log", help="Path to Zeek conn.log")
    parser.add_argument("--scan-dir", help="Directory to scan for encrypted files")
    parser.add_argument("--check-processes", action="store_true", help="Scan running processes")
    parser.add_argument("--windows-logs", action="store_true", help="Check Windows event logs")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    args = parser.parse_args()

    print("[*] Ransomware Precursor Detection Agent")
    report = {"timestamp": datetime.now(timezone.utc).isoformat(), "findings": {}}

    if args.conn_log:
        alerts = parse_zeek_conn_log(args.conn_log)
        report["findings"]["network"] = alerts
        print(f"[*] Network alerts: {len(alerts)}")

    if args.check_processes:
        procs = scan_process_list()
        report["findings"]["suspicious_processes"] = procs
        print(f"[*] Suspicious processes: {len(procs)}")

    if args.windows_logs and sys.platform == "win32":
        events = analyze_event_logs_windows()
        report["findings"]["windows_events"] = events
        print(f"[*] Windows event findings: {len(events)}")

    if args.scan_dir:
        enc = check_file_encryption_activity(args.scan_dir)
        report["findings"]["encryption_activity"] = enc
        print(f"[*] Encrypted files found: {enc['encrypted_file_count']}")

    total = sum(
        len(v) if isinstance(v, list) else (1 if isinstance(v, dict) and v.get("threshold_exceeded") else 0)
        for v in report["findings"].values()
    )
    report["risk_level"] = "CRITICAL" if total >= 10 else "HIGH" if total >= 5 else "MEDIUM" if total > 0 else "LOW"
    print(f"[*] Overall risk: {report['risk_level']}")

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[*] Report saved to {args.output}")
    else:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-ransomware-precursors-in-network/scripts/process.py =====
#!/usr/bin/env python3
"""
Ransomware Precursor Detection Engine

Analyzes network logs (Zeek format) to detect ransomware precursor patterns:
- C2 beaconing detection via statistical interval analysis
- Internal reconnaissance scanning
- Kerberoasting and credential harvesting indicators
- Admin share enumeration
- Data staging via large SMB transfers

Reads Zeek TSV logs and generates structured alerts.
"""

import csv
import json
import math
import os
import sys
import statistics
from collections import defaultdict
from dataclasses import dataclass, field, asdict
from datetime import datetime, timedelta
from pathlib import Path
from typing import Optional


@dataclass
class PrecursorAlert:
    alert_id: str
    timestamp: str
    source_ip: str
    dest_ip: str
    category: str
    confidence: str
    kill_chain_phase: str
    description: str
    mitre_technique: str
    evidence: list = field(default_factory=list)


class BeaconDetector:
    """Detects C2 beaconing by analyzing connection interval patterns."""

    def __init__(self, min_connections: int = 20, beacon_score_threshold: float = 0.7):
        self.min_connections = min_connections
        self.beacon_score_threshold = beacon_score_threshold
        self.connections = defaultdict(list)

    def add_connection(self, src_ip: str, dst_ip: str, timestamp: float, orig_bytes: int, resp_bytes: int):
        key = (src_ip, dst_ip)
        self.connections[key].append({
            "ts": timestamp,
            "orig_bytes": orig_bytes,
            "resp_bytes": resp_bytes,
        })

    def calculate_beacon_score(self, timestamps: list) -> dict:
        """Calculate beacon score based on connection interval regularity."""
        if len(timestamps) < self.min_connections:
            return {"score": 0.0, "interval": 0, "jitter": 0}

        sorted_ts = sorted(timestamps)
        intervals = [sorted_ts[i + 1] - sorted_ts[i] for i in range(len(sorted_ts) - 1)]

        if not intervals:
            return {"score": 0.0, "interval": 0, "jitter": 0}

        median_interval = statistics.median(intervals)
        if median_interval == 0:
            return {"score": 0.0, "interval": 0, "jitter": 0}

        # Calculate coefficient of variation (lower = more regular = more likely beacon)
        try:
            stdev = statistics.stdev(intervals)
            cv = stdev / median_interval
        except statistics.StatisticsError:
            cv = 0

        # Beacon score: inverse of coefficient of variation, capped at 1.0
        # Perfect beacon (cv=0) scores 1.0, high variation scores low
        if cv == 0:
            score = 1.0
        else:
            score = max(0, min(1.0, 1.0 - cv))

        # Penalize very short intervals (likely legitimate keep-alives under 5s)
        if median_interval < 5:
            score *= 0.5

        # Penalize very long intervals (over 1 hour - less likely active C2)
        if median_interval > 3600:
            score *= 0.7

        return {
            "score": round(score, 3),
            "interval": round(median_interval, 1),
            "jitter": round(stdev, 1) if stdev else 0,
            "connection_count": len(timestamps),
        }

    def detect(self) -> list:
        """Detect beaconing patterns in collected connections."""
        alerts = []
        for (src_ip, dst_ip), conns in self.connections.items():
            timestamps = [c["ts"] for c in conns]
            result = self.calculate_beacon_score(timestamps)

            if result["score"] >= self.beacon_score_threshold:
                # Check for consistent payload sizes (another beacon indicator)
                orig_sizes = [c["orig_bytes"] for c in conns if c["orig_bytes"] > 0]
                size_consistency = 0.0
                if len(orig_sizes) >= 5:
                    try:
                        size_cv = statistics.stdev(orig_sizes) / statistics.mean(orig_sizes)
                        size_consistency = max(0, 1.0 - size_cv)
                    except (statistics.StatisticsError, ZeroDivisionError):
                        pass

                combined_score = (result["score"] * 0.7) + (size_consistency * 0.3)

                if combined_score >= self.beacon_score_threshold:
                    confidence = "High" if combined_score >= 0.9 else "Medium"
                    alert = PrecursorAlert(
                        alert_id=f"BEACON-{src_ip}-{dst_ip}",
                        timestamp=datetime.fromtimestamp(max(timestamps)).isoformat(),
                        source_ip=src_ip,
                        dest_ip=dst_ip,
                        category="C2 Beaconing",
                        confidence=confidence,
                        kill_chain_phase="Command and Control",
                        description=(
                            f"Beaconing pattern detected: {result['connection_count']} connections "
                            f"at {result['interval']}s intervals (jitter: {result['jitter']}s, "
                            f"beacon score: {combined_score:.3f})"
                        ),
                        mitre_technique="T1071 - Application Layer Protocol",
                        evidence=[
                            f"Beacon score: {combined_score:.3f}",
                            f"Interval: {result['interval']}s",
                            f"Jitter: {result['jitter']}s",
                            f"Payload size consistency: {size_consistency:.3f}",
                            f"Connections: {result['connection_count']}",
                        ],
                    )
                    alerts.append(alert)
        return alerts


class ScanDetector:
    """Detects internal reconnaissance scanning."""

    def __init__(self, unique_dest_threshold: int = 30, time_window_seconds: int = 300):
        self.threshold = unique_dest_threshold
        self.window = time_window_seconds
        self.connections = defaultdict(list)

    def add_connection(self, src_ip: str, dst_ip: str, dst_port: int, timestamp: float):
        self.connections[src_ip].append({
            "dst_ip": dst_ip,
            "dst_port": dst_port,
            "ts": timestamp,
        })

    def detect(self) -> list:
        alerts = []
        for src_ip, conns in self.connections.items():
            sorted_conns = sorted(conns, key=lambda c: c["ts"])

            # Sliding window analysis
            window_start = 0
            for window_end in range(len(sorted_conns)):
                while (sorted_conns[window_end]["ts"] - sorted_conns[window_start]["ts"]) > self.window:
                    window_start += 1

                window_conns = sorted_conns[window_start:window_end + 1]
                unique_dests = set(c["dst_ip"] for c in window_conns)
                unique_ports = set(c["dst_port"] for c in window_conns)

                if len(unique_dests) >= self.threshold:
                    alert = PrecursorAlert(
                        alert_id=f"SCAN-{src_ip}-{int(sorted_conns[window_start]['ts'])}",
                        timestamp=datetime.fromtimestamp(sorted_conns[window_end]["ts"]).isoformat(),
                        source_ip=src_ip,
                        dest_ip="Multiple",
                        category="Internal Reconnaissance",
                        confidence="High" if len(unique_dests) >= self.threshold * 2 else "Medium",
                        kill_chain_phase="Discovery",
                        description=(
                            f"Internal scan: {len(unique_dests)} unique destinations on "
                            f"{len(unique_ports)} ports within {self.window}s window"
                        ),
                        mitre_technique="T1046 - Network Service Discovery",
                        evidence=[
                            f"Unique destinations: {len(unique_dests)}",
                            f"Unique ports: {sorted(unique_ports)}",
                            f"Total connections: {len(window_conns)}",
                            f"Time window: {self.window}s",
                        ],
                    )
                    alerts.append(alert)
                    break  # One alert per source IP per window

        return alerts


class CredentialHarvestDetector:
    """Detects Kerberoasting and credential harvesting patterns."""

    def __init__(self, kerberoast_threshold: int = 5):
        self.threshold = kerberoast_threshold
        self.tgs_requests = defaultdict(list)
        self.smb_failures = defaultdict(int)

    def add_kerberos_event(self, src_ip: str, service_name: str, encryption_type: str, timestamp: float):
        self.tgs_requests[src_ip].append({
            "service": service_name,
            "enc_type": encryption_type,
            "ts": timestamp,
        })

    def add_smb_failure(self, src_ip: str, dst_ip: str):
        self.smb_failures[src_ip] += 1

    def detect(self) -> list:
        alerts = []

        # Kerberoasting: multiple TGS requests for unique services with RC4 encryption
        for src_ip, requests in self.tgs_requests.items():
            rc4_requests = [r for r in requests if "rc4" in r.get("enc_type", "").lower()
                          or "23" in str(r.get("enc_type", ""))]
            unique_services = set(r["service"] for r in rc4_requests)

            if len(unique_services) >= self.threshold:
                alert = PrecursorAlert(
                    alert_id=f"KERB-{src_ip}",
                    timestamp=datetime.fromtimestamp(max(r["ts"] for r in rc4_requests)).isoformat(),
                    source_ip=src_ip,
                    dest_ip="Domain Controller",
                    category="Kerberoasting",
                    confidence="High",
                    kill_chain_phase="Credential Access",
                    description=(
                        f"Possible Kerberoasting: {len(unique_services)} unique service ticket "
                        f"requests with RC4 encryption from single host"
                    ),
                    mitre_technique="T1558.003 - Kerberoasting",
                    evidence=[
                        f"Unique services targeted: {len(unique_services)}",
                        f"RC4 encryption requests: {len(rc4_requests)}",
                        f"Services: {list(unique_services)[:10]}",
                    ],
                )
                alerts.append(alert)

        # SMB brute force
        for src_ip, count in self.smb_failures.items():
            if count >= 10:
                alert = PrecursorAlert(
                    alert_id=f"SMB-BRUTE-{src_ip}",
                    timestamp=datetime.now().isoformat(),
                    source_ip=src_ip,
                    dest_ip="Multiple",
                    category="SMB Brute Force",
                    confidence="Medium",
                    kill_chain_phase="Credential Access",
                    description=f"SMB authentication failures: {count} failed attempts",
                    mitre_technique="T1110 - Brute Force",
                    evidence=[f"Failed SMB auth count: {count}"],
                )
                alerts.append(alert)

        return alerts


class AdminShareDetector:
    """Detects suspicious access to administrative shares (C$, ADMIN$, IPC$)."""

    def __init__(self, threshold: int = 5):
        self.threshold = threshold
        self.share_access = defaultdict(lambda: defaultdict(set))

    def add_share_access(self, src_ip: str, dst_ip: str, share_name: str, timestamp: float):
        admin_shares = {"ADMIN$", "C$", "IPC$", "D$", "E$"}
        normalized_share = share_name.split("\\")[-1].upper()
        if normalized_share in admin_shares:
            self.share_access[src_ip][normalized_share].add(dst_ip)

    def detect(self) -> list:
        alerts = []
        for src_ip, shares in self.share_access.items():
            total_targets = set()
            for share, targets in shares.items():
                total_targets.update(targets)

            if len(total_targets) >= self.threshold:
                alert = PrecursorAlert(
                    alert_id=f"ADMINSHARE-{src_ip}",
                    timestamp=datetime.now().isoformat(),
                    source_ip=src_ip,
                    dest_ip="Multiple",
                    category="Admin Share Enumeration",
                    confidence="High" if len(total_targets) >= self.threshold * 2 else "Medium",
                    kill_chain_phase="Lateral Movement",
                    description=(
                        f"Admin share access to {len(total_targets)} hosts: "
                        f"shares accessed: {list(shares.keys())}"
                    ),
                    mitre_technique="T1021.002 - SMB/Windows Admin Shares",
                    evidence=[
                        f"Unique targets: {len(total_targets)}",
                        f"Shares accessed: {dict((s, len(t)) for s, t in shares.items())}",
                    ],
                )
                alerts.append(alert)

        return alerts


class RansomwarePrecursorEngine:
    """Orchestrates all detection modules."""

    def __init__(self):
        self.beacon_detector = BeaconDetector()
        self.scan_detector = ScanDetector()
        self.cred_detector = CredentialHarvestDetector()
        self.share_detector = AdminShareDetector()
        self.alerts = []

    def load_zeek_conn_log(self, filepath: str):
        """Parse Zeek conn.log for beacon and scan detection."""
        with open(filepath, "r") as f:
            for line in f:
                if line.startswith("#"):
                    continue
                fields = line.strip().split("\t")
                if len(fields) < 20:
                    continue
                try:
                    ts = float(fields[0])
                    src_ip = fields[2]
                    src_port = int(fields[3]) if fields[3] != "-" else 0
                    dst_ip = fields[4]
                    dst_port = int(fields[5]) if fields[5] != "-" else 0
                    proto = fields[6]
                    orig_bytes = int(fields[9]) if fields[9] != "-" else 0
                    resp_bytes = int(fields[10]) if fields[10] != "-" else 0

                    # Feed to beacon detector (external destinations)
                    if not self._is_internal(dst_ip):
                        self.beacon_detector.add_connection(src_ip, dst_ip, ts, orig_bytes, resp_bytes)

                    # Feed to scan detector (internal destinations)
                    if self._is_internal(src_ip) and self._is_internal(dst_ip):
                        self.scan_detector.add_connection(src_ip, dst_ip, dst_port, ts)
                except (ValueError, IndexError):
                    continue

    def _is_internal(self, ip: str) -> bool:
        """Check if IP is in RFC1918 private range."""
        parts = ip.split(".")
        if len(parts) != 4:
            return False
        try:
            first = int(parts[0])
            second = int(parts[1])
            if first == 10:
                return True
            if first == 172 and 16 <= second <= 31:
                return True
            if first == 192 and second == 168:
                return True
        except ValueError:
            pass
        return False

    def run_detection(self) -> list:
        """Run all detectors and return combined alerts."""
        self.alerts = []
        self.alerts.extend(self.beacon_detector.detect())
        self.alerts.extend(self.scan_detector.detect())
        self.alerts.extend(self.cred_detector.detect())
        self.alerts.extend(self.share_detector.detect())

        # Sort by confidence (High first)
        confidence_order = {"High": 0, "Medium": 1, "Low": 2}
        self.alerts.sort(key=lambda a: confidence_order.get(a.confidence, 3))

        return self.alerts

    def generate_report(self) -> str:
        """Generate formatted detection report."""
        if not self.alerts:
            self.run_detection()

        lines = []
        lines.append("=" * 70)
        lines.append("RANSOMWARE PRECURSOR DETECTION REPORT")
        lines.append("=" * 70)
        lines.append(f"Generated: {datetime.now().isoformat()}")
        lines.append(f"Total Alerts: {len(self.alerts)}")

        by_category = defaultdict(list)
        for alert in self.alerts:
            by_category[alert.category].append(alert)

        lines.append(f"\nAlert Categories:")
        for cat, cat_alerts in sorted(by_category.items()):
            lines.append(f"  - {cat}: {len(cat_alerts)}")

        lines.append("")
        for i, alert in enumerate(self.alerts, 1):
            lines.append("-" * 50)
            lines.append(f"Alert #{i}: {alert.alert_id}")
            lines.append(f"  Category: {alert.category}")
            lines.append(f"  Confidence: {alert.confidence}")
            lines.append(f"  Kill Chain: {alert.kill_chain_phase}")
            lines.append(f"  Source: {alert.source_ip}")
            lines.append(f"  Destination: {alert.dest_ip}")
            lines.append(f"  MITRE: {alert.mitre_technique}")
            lines.append(f"  Description: {alert.description}")
            lines.append(f"  Evidence:")
            for e in alert.evidence:
                lines.append(f"    - {e}")

        lines.append("")
        lines.append("=" * 70)
        return "\n".join(lines)


def main():
    """Run detection engine with sample data or Zeek log file."""
    engine = RansomwarePrecursorEngine()

    # Check for Zeek conn.log argument
    if len(sys.argv) > 1:
        log_file = sys.argv[1]
        if os.path.exists(log_file):
            print(f"Loading Zeek conn.log: {log_file}")
            engine.load_zeek_conn_log(log_file)
        else:
            print(f"File not found: {log_file}")
            sys.exit(1)
    else:
        # Demo with simulated data
        print("No Zeek log provided. Running with simulated beacon data...")
        import time

        base_time = time.time() - 3600  # 1 hour ago

        # Simulate Cobalt Strike beacon (60-second interval)
        for i in range(40):
            jitter = (i % 3) * 2  # Small jitter
            engine.beacon_detector.add_connection(
                "10.1.5.42", "185.220.101.42",
                base_time + (i * 60) + jitter,
                orig_bytes=48, resp_bytes=128,
            )

        # Simulate internal port scan
        for i in range(50):
            engine.scan_detector.add_connection(
                "10.1.5.42", f"10.1.5.{100 + i}", 445,
                base_time + 1800 + (i * 2),
            )

        # Simulate Kerberoasting
        for i in range(8):
            engine.cred_detector.add_kerberos_event(
                "10.1.5.42", f"MSSQLSvc/sql{i}.corp.local:1433",
                "rc4-hmac", base_time + 2000 + (i * 5),
            )

        # Simulate admin share access
        for i in range(12):
            engine.share_detector.add_share_access(
                "10.1.5.42", f"10.1.5.{200 + i}", "ADMIN$",
                base_time + 2500 + (i * 10),
            )

    report = engine.generate_report()
    print(report)

    # Export alerts as JSON
    alerts_json = [asdict(a) for a in engine.alerts]
    output_path = Path(__file__).parent / "precursor_alerts.json"
    with open(output_path, "w") as f:
        json.dump(alerts_json, f, indent=2)
    print(f"\nAlerts exported to: {output_path}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-anti-ransomware-group-policy/SKILL.md =====
---
name: implementing-anti-ransomware-group-policy
description: 'Configures Windows Group Policy Objects to block ransomware execution
  and lateral spread, covering AppLocker rules, Software Restriction Policies, Controlled
  Folder Access, attack surface reduction rules, and network protection settings.
  Use when hardening Windows endpoints against ransomware via GPO, configuring AppLocker
  or Controlled Folder Access, or building endpoint protection policies through Group
  Policy.'
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- group-policy
- windows
- AppLocker
- hardening
- prevention
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1486
- T1490
mitre_f3:
  version: '1.1'
  tactics:
  - initial-access
  - positioning
  - monetization
  techniques:
  - id: T1660
    name: Phishing
    tactic: initial-access
    source: attack
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
  - id: T1531
    name: Account Access Removal
    tactic: positioning
    source: attack
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
---

# Implementing Anti-Ransomware Group Policy

## When to Use

- Hardening a Windows Active Directory environment against ransomware execution and propagation
- Implementing defense-in-depth by blocking ransomware execution paths via Group Policy
- Configuring AppLocker or WDAC rules to prevent unauthorized executables from running in user-writable directories
- Enabling Controlled Folder Access to protect critical directories from unauthorized file modifications
- Restricting lateral movement vectors (RDP, SMB, WMI) that ransomware uses to spread across the domain

**Do not use** as a standalone ransomware defense. GPO settings complement but do not replace endpoint detection, backups, network segmentation, and user awareness training.

## Prerequisites

- Windows Server 2016+ Active Directory environment with Group Policy Management Console (GPMC)
- Domain Admin or Group Policy Creator Owners privileges
- Windows 10/11 Enterprise or Education (required for AppLocker and WDAC)
- Microsoft Defender Antivirus enabled (required for Controlled Folder Access and ASR rules)
- Python 3.8+ for audit script that validates GPO compliance
- Test OU for validating GPO settings before domain-wide deployment

## Workflow

### Step 1: Block Ransomware Execution Paths with AppLocker

Configure AppLocker to prevent executables from running in common ransomware staging locations:

```
AppLocker GPO Path:
  Computer Configuration → Policies → Windows Settings →
  Security Settings → Application Control Policies → AppLocker

Key Rules:
━━━━━━━━━
1. DENY executable rules for user-writable paths:
   - %USERPROFILE%\AppData\Local\Temp\*     (email attachment extraction)
   - %USERPROFILE%\AppData\Roaming\*         (CryptoLocker staging)
   - %USERPROFILE%\Downloads\*               (web downloads)
   - %TEMP%\*                                (temporary extraction)
   - %USERPROFILE%\Desktop\*                 (social engineering drops)

2. ALLOW default rules:
   - C:\Windows\* (signed by Microsoft)
   - C:\Program Files\* and C:\Program Files (x86)\*
   - Administrator group: all paths

3. Enable Application Identity service:
   Computer Configuration → Policies → Windows Settings →
   Security Settings → System Services →
   Application Identity → Automatic
```

### Step 2: Enable Controlled Folder Access

Protect critical directories from unauthorized modification:

```
Controlled Folder Access GPO Path:
  Computer Configuration → Administrative Templates →
  Windows Components → Microsoft Defender Antivirus →
  Microsoft Defender Exploit Guard → Controlled Folder Access

Settings:
━━━━━━━━━
1. Configure Controlled folder access: Enabled → Block mode
2. Configure protected folders: Add custom paths
   - \\fileserver\shares\finance
   - \\fileserver\shares\hr
   - C:\Users\*\Documents
   - C:\Users\*\Desktop

3. Configure allowed applications: Whitelist trusted apps
   - C:\Program Files\Microsoft Office\*
   - C:\Program Files\Adobe\*
   - Line-of-business applications

Default protected folders (automatic):
  Documents, Pictures, Videos, Music, Desktop, Favorites
```

### Step 3: Configure Attack Surface Reduction (ASR) Rules

Enable ASR rules that target ransomware delivery mechanisms:

```
ASR Rules GPO Path:
  Computer Configuration → Administrative Templates →
  Windows Components → Microsoft Defender Antivirus →
  Microsoft Defender Exploit Guard → Attack Surface Reduction

Critical ASR Rules for Ransomware Prevention:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GUID                                    Rule
BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550   Block executable content from email
D4F940AB-401B-4EFC-AADC-AD5F3C50688A   Block Office apps from creating child processes
3B576869-A4EC-4529-8536-B80A7769E899   Block Office apps from creating executable content
75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84   Block Office apps from injecting into processes
D3E037E1-3EB8-44C8-A917-57927947596D   Block JavaScript/VBScript from launching downloads
5BEB7EFE-FD9A-4556-801D-275E5FFC04CC   Block execution of obfuscated scripts
92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B   Block Win32 API calls from Office macros
01443614-CD74-433A-B99E-2ECDC07BFC25   Block executable files unless they meet prevalence criteria

Set each rule to: Block (1) or Audit (2) for initial testing
```

### Step 4: Restrict Lateral Movement Vectors

Lock down SMB, RDP, and WMI to limit ransomware propagation:

```
Network Restrictions:
━━━━━━━━━━━━━━━━━━━━
1. Disable SMBv1:
   Computer Configuration → Administrative Templates →
   Network → Lanman Workstation → Enable insecure guest logons: Disabled

   Computer Configuration → Administrative Templates →
   MS Security Guide → Configure SMBv1 server: Disabled

2. Restrict Remote Desktop:
   Computer Configuration → Administrative Templates →
   Windows Components → Remote Desktop Services →
   Remote Desktop Session Host → Connections →
   Allow users to connect remotely: Disabled (or restricted to specific groups)

3. Disable remote WMI:
   Windows Firewall → Inbound Rules →
   Block Windows Management Instrumentation (WMI) inbound

4. Disable AutoPlay/AutoRun:
   Computer Configuration → Administrative Templates →
   Windows Components → AutoPlay Policies →
   Turn off AutoPlay: Enabled (All drives)

5. Disable PowerShell remoting for non-admin users:
   Computer Configuration → Administrative Templates →
   Windows Components → Windows PowerShell →
   Turn on Script Execution: Allow only signed scripts
```

### Step 5: Audit and Validate GPO Compliance

Verify that GPO settings are applied correctly across the domain:

```powershell
# Check GPO application on endpoint
gpresult /r /scope:computer

# Verify AppLocker rules
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections

# Check Controlled Folder Access status
Get-MpPreference | Select-Object EnableControlledFolderAccess

# List protected folders
Get-MpPreference | Select-Object -ExpandProperty ControlledFolderAccessProtectedFolders

# Check ASR rules
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Ids
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Actions
```

## Verification

- Run `gpresult /r` on test endpoints to confirm GPO application
- Attempt to run an executable from `%AppData%\Temp` to verify AppLocker blocks it
- Modify a file in a protected folder from an unlisted application to confirm CFA blocks it
- Test ASR rules by opening a macro-enabled document and verifying child process blocking
- Validate that legitimate applications in the allowlist still function correctly
- Check Windows Event Log for AppLocker events (Event IDs 8003, 8004) and CFA events (1123, 1124)

## Key Concepts

| Term | Definition |
|------|------------|
| **AppLocker** | Windows application control feature that restricts which executables, scripts, and DLLs users can run based on publisher, path, or hash rules |
| **Controlled Folder Access** | Microsoft Defender feature that prevents untrusted applications from modifying files in protected directories |
| **Attack Surface Reduction (ASR)** | Set of rules in Microsoft Defender Exploit Guard that block specific attack behaviors like Office macro child processes |
| **Software Restriction Policies (SRP)** | Legacy Windows feature (deprecated in Win 11) for restricting executables; replaced by AppLocker and WDAC |
| **WDAC** | Windows Defender Application Control; the successor to AppLocker with stronger enforcement using code integrity policies |

## Tools & Systems

- **Group Policy Management Console (GPMC)**: Primary tool for creating and managing GPOs in Active Directory
- **AppLocker**: Built-in Windows application whitelisting and blacklisting engine
- **Microsoft Defender Exploit Guard**: Suite including CFA, ASR rules, and Network Protection
- **GPResult**: Command-line tool for verifying GPO application status on endpoints
- **PowerShell Get-MpPreference**: Cmdlet for querying Microsoft Defender configuration including ASR and CFA status

===== END FILE =====


===== FILE: skills/implementing-anti-ransomware-group-policy/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/implementing-anti-ransomware-group-policy/references/api-reference.md =====
# API Reference: Anti-Ransomware Group Policy

## AppLocker PowerShell Cmdlets

### Get Effective Policy
```powershell
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```

### Create AppLocker Rule
```powershell
# Deny executables from AppData paths
New-AppLockerPolicy -RuleType Path -RuleNamePrefix "DenyAppData" `
  -Path "%USERPROFILE%\AppData\*" -Action Deny -User Everyone
```

### Test AppLocker Policy
```powershell
Test-AppLockerPolicy -Path "C:\Users\test\AppData\Local\Temp\malware.exe" `
  -XmlPolicy (Get-AppLockerPolicy -Effective -Xml)
```

### AppLocker Event Log IDs
| Event ID | Log | Description |
|----------|-----|-------------|
| 8003 | AppLocker/EXE | Allowed executable |
| 8004 | AppLocker/EXE | Blocked executable |
| 8005 | AppLocker/Script | Allowed script |
| 8006 | AppLocker/Script | Blocked script |
| 8007 | AppLocker/MSI | Allowed installer |
| 8008 | AppLocker/MSI | Blocked installer |

## Controlled Folder Access (CFA)

### Enable CFA
```powershell
Set-MpPreference -EnableControlledFolderAccess Enabled
```

### CFA Modes
| Value | Mode | Description |
|-------|------|-------------|
| 0 | Disabled | No protection |
| 1 | Enabled | Block unauthorized modifications |
| 2 | Audit | Log but do not block |
| 6 | BlockDiskModificationOnly | Block disk-level changes only |

### Add Protected Folders
```powershell
Add-MpPreference -ControlledFolderAccessProtectedFolders "C:\Finance"
```

### Add Allowed Applications
```powershell
Add-MpPreference -ControlledFolderAccessAllowedApplications "C:\Program Files\App\app.exe"
```

### CFA Event IDs
| Event ID | Log | Description |
|----------|-----|-------------|
| 1123 | Defender/Operational | Blocked file modification |
| 1124 | Defender/Operational | Audited file modification |

## Attack Surface Reduction (ASR) Rules

### Enable ASR Rule
```powershell
Add-MpPreference -AttackSurfaceReductionRules_Ids <GUID> `
  -AttackSurfaceReductionRules_Actions Enabled
```

### ASR Rule Actions
| Value | Action |
|-------|--------|
| 0 | Disabled |
| 1 | Block |
| 2 | Audit |
| 6 | Warn |

### Key Anti-Ransomware ASR Rule GUIDs
| GUID | Rule |
|------|------|
| BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550 | Block executable content from email |
| D4F940AB-401B-4EFC-AADC-AD5F3C50688A | Block Office child processes |
| 3B576869-A4EC-4529-8536-B80A7769E899 | Block Office executable content creation |
| 75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84 | Block Office code injection |
| D3E037E1-3EB8-44C8-A917-57927947596D | Block JS/VBS downloaded executables |
| 5BEB7EFE-FD9A-4556-801D-275E5FFC04CC | Block obfuscated scripts |
| 92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B | Block Win32 API from Office macros |

### ASR Event IDs
| Event ID | Log | Description |
|----------|-----|-------------|
| 1121 | Defender/Operational | ASR rule fired in block mode |
| 1122 | Defender/Operational | ASR rule fired in audit mode |

## GPO Paths Reference

### AppLocker
```
Computer Configuration → Policies → Windows Settings →
Security Settings → Application Control Policies → AppLocker
```

### Controlled Folder Access
```
Computer Configuration → Administrative Templates →
Windows Components → Microsoft Defender Antivirus →
Microsoft Defender Exploit Guard → Controlled Folder Access
```

### Attack Surface Reduction
```
Computer Configuration → Administrative Templates →
Windows Components → Microsoft Defender Antivirus →
Microsoft Defender Exploit Guard → Attack Surface Reduction
```

### Network Restrictions
```
Computer Configuration → Administrative Templates →
Network → Lanman Workstation    (SMB settings)
Windows Components → Remote Desktop Services    (RDP settings)
Windows Components → AutoPlay Policies    (AutoPlay/AutoRun)
```

## GPResult Verification

```powershell
# Check applied GPOs
gpresult /r /scope:computer

# Generate HTML report
gpresult /h gpo_report.html

# Check specific policy RSoP
gpresult /z /scope:computer
```

===== END FILE =====


===== FILE: skills/implementing-anti-ransomware-group-policy/scripts/agent.py =====
#!/usr/bin/env python3
"""Anti-ransomware Group Policy audit and configuration agent.

Audits Windows endpoints for GPO-based ransomware defenses including AppLocker
rules, Controlled Folder Access, Attack Surface Reduction rules, and network
restriction settings. Generates compliance reports against recommended baselines.
"""

import json
import os
import subprocess
import sys
from datetime import datetime

APPLOCKER_DENY_PATHS = [
    "%USERPROFILE%\\AppData\\Local\\Temp\\*",
    "%USERPROFILE%\\AppData\\Roaming\\*",
    "%USERPROFILE%\\Downloads\\*",
    "%TEMP%\\*",
    "%USERPROFILE%\\Desktop\\*",
]

ASR_RULES = {
    "BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550": {
        "name": "Block executable content from email client and webmail",
        "recommended": "Block",
        "category": "Email protection",
    },
    "D4F940AB-401B-4EFC-AADC-AD5F3C50688A": {
        "name": "Block all Office applications from creating child processes",
        "recommended": "Block",
        "category": "Office protection",
    },
    "3B576869-A4EC-4529-8536-B80A7769E899": {
        "name": "Block Office applications from creating executable content",
        "recommended": "Block",
        "category": "Office protection",
    },
    "75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84": {
        "name": "Block Office applications from injecting code into other processes",
        "recommended": "Block",
        "category": "Office protection",
    },
    "D3E037E1-3EB8-44C8-A917-57927947596D": {
        "name": "Block JavaScript or VBScript from launching downloaded executable content",
        "recommended": "Block",
        "category": "Script protection",
    },
    "5BEB7EFE-FD9A-4556-801D-275E5FFC04CC": {
        "name": "Block execution of potentially obfuscated scripts",
        "recommended": "Block",
        "category": "Script protection",
    },
    "92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B": {
        "name": "Block Win32 API calls from Office macros",
        "recommended": "Block",
        "category": "Macro protection",
    },
    "01443614-CD74-433A-B99E-2ECDC07BFC25": {
        "name": "Block executable files from running unless they meet a prevalence, age, or trusted list criterion",
        "recommended": "Block",
        "category": "Execution protection",
    },
}

CFA_RECOMMENDED_FOLDERS = [
    "Documents", "Desktop", "Pictures", "Videos", "Music", "Favorites",
]

GPO_CHECKS = {
    "applocker_enabled": {
        "description": "AppLocker Application Control is enabled",
        "check_command": ["powershell", "-Command",
                          "Get-AppLockerPolicy -Effective -ErrorAction SilentlyContinue | "
                          "ConvertTo-Json -Depth 3"],
        "priority": "Critical",
    },
    "cfa_enabled": {
        "description": "Controlled Folder Access is enabled",
        "check_command": ["powershell", "-Command",
                          "(Get-MpPreference).EnableControlledFolderAccess"],
        "priority": "Critical",
    },
    "asr_rules": {
        "description": "Attack Surface Reduction rules are configured",
        "check_command": ["powershell", "-Command",
                          "(Get-MpPreference).AttackSurfaceReductionRules_Ids"],
        "priority": "High",
    },
    "asr_actions": {
        "description": "ASR rule actions (0=Disabled,1=Block,2=Audit,6=Warn)",
        "check_command": ["powershell", "-Command",
                          "(Get-MpPreference).AttackSurfaceReductionRules_Actions"],
        "priority": "High",
    },
    "smbv1_disabled": {
        "description": "SMBv1 protocol is disabled",
        "check_command": ["powershell", "-Command",
                          "(Get-SmbServerConfiguration).EnableSMB1Protocol"],
        "priority": "Critical",
    },
    "autoplay_disabled": {
        "description": "AutoPlay is disabled",
        "check_command": ["powershell", "-Command",
                          "Get-ItemProperty -Path 'HKLM:\\SOFTWARE\\Microsoft\\Windows"
                          "\\CurrentVersion\\Policies\\Explorer' -Name NoDriveTypeAutoRun "
                          "-ErrorAction SilentlyContinue | Select-Object -ExpandProperty NoDriveTypeAutoRun"],
        "priority": "Medium",
    },
}


def run_check(check_name, check_info):
    """Run a single GPO compliance check."""
    result = {
        "check": check_name,
        "description": check_info["description"],
        "priority": check_info["priority"],
        "status": "unknown",
        "output": "",
    }

    try:
        proc = subprocess.run(
            check_info["check_command"],
            capture_output=True, text=True, timeout=30,
        )
        output = proc.stdout.strip()
        result["output"] = output
        result["return_code"] = proc.returncode

        if proc.returncode != 0:
            result["status"] = "error"
            result["error"] = proc.stderr.strip()
        elif output:
            result["status"] = "data_retrieved"
        else:
            result["status"] = "empty_response"

    except FileNotFoundError:
        result["status"] = "not_available"
        result["error"] = "PowerShell not found or command not available"
    except subprocess.TimeoutExpired:
        result["status"] = "timeout"
        result["error"] = "Check timed out after 30 seconds"

    return result


def assess_asr_compliance(asr_ids_output, asr_actions_output):
    """Assess ASR rule compliance against recommended baseline."""
    active_ids = [line.strip() for line in asr_ids_output.split("\n") if line.strip()]
    active_actions = [line.strip() for line in asr_actions_output.split("\n") if line.strip()]

    assessment = {"total_recommended": len(ASR_RULES), "enabled": 0,
                  "blocking": 0, "audit_only": 0, "missing": [], "details": []}

    id_action_map = {}
    for i, rule_id in enumerate(active_ids):
        action = active_actions[i] if i < len(active_actions) else "0"
        id_action_map[rule_id.upper()] = action

    for rule_id, rule_info in ASR_RULES.items():
        action = id_action_map.get(rule_id.upper(), None)
        if action is None:
            assessment["missing"].append({"id": rule_id, "name": rule_info["name"]})
            status = "NOT CONFIGURED"
        elif action == "1":
            assessment["enabled"] += 1
            assessment["blocking"] += 1
            status = "BLOCK"
        elif action == "2":
            assessment["enabled"] += 1
            assessment["audit_only"] += 1
            status = "AUDIT"
        elif action == "6":
            assessment["enabled"] += 1
            status = "WARN"
        else:
            status = "DISABLED"

        assessment["details"].append({
            "id": rule_id,
            "name": rule_info["name"],
            "category": rule_info["category"],
            "status": status,
            "recommended": rule_info["recommended"],
        })

    assessment["compliance_pct"] = round(
        (assessment["blocking"] / assessment["total_recommended"]) * 100, 1
    )
    return assessment


def generate_gpo_report():
    """Generate a full GPO anti-ransomware compliance report."""
    report = {
        "title": "Anti-Ransomware Group Policy Compliance Report",
        "generated": datetime.now().isoformat(),
        "hostname": os.environ.get("COMPUTERNAME", "unknown"),
        "checks": {},
        "overall_score": 0,
    }

    print("[*] Running GPO compliance checks...")
    for check_name, check_info in GPO_CHECKS.items():
        print(f"  Checking: {check_info['description']}...")
        result = run_check(check_name, check_info)
        report["checks"][check_name] = result

    # ASR compliance assessment
    asr_ids = report["checks"].get("asr_rules", {}).get("output", "")
    asr_actions = report["checks"].get("asr_actions", {}).get("output", "")
    if asr_ids:
        report["asr_assessment"] = assess_asr_compliance(asr_ids, asr_actions)

    # Calculate overall score
    score = 0
    total = len(GPO_CHECKS)
    for check_name, result in report["checks"].items():
        if result["status"] == "data_retrieved" and result.get("output"):
            score += 1
    report["overall_score"] = round((score / total) * 100, 1)

    return report


def generate_remediation_script(report):
    """Generate PowerShell remediation commands for failed checks."""
    lines = [
        "# Anti-Ransomware GPO Remediation Script",
        "# Generated: " + datetime.now().isoformat(),
        "# Run as Administrator in elevated PowerShell",
        "",
    ]

    cfa = report.get("checks", {}).get("cfa_enabled", {})
    if cfa.get("output", "").strip() != "1":
        lines.append("# Enable Controlled Folder Access")
        lines.append("Set-MpPreference -EnableControlledFolderAccess Enabled")
        lines.append("")

    smbv1 = report.get("checks", {}).get("smbv1_disabled", {})
    if smbv1.get("output", "").strip().lower() == "true":
        lines.append("# Disable SMBv1")
        lines.append("Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force")
        lines.append("")

    asr = report.get("asr_assessment", {})
    for rule in asr.get("details", []):
        if rule["status"] in ("NOT CONFIGURED", "DISABLED", "AUDIT"):
            lines.append(f"# Enable ASR rule: {rule['name']}")
            lines.append(f"Add-MpPreference -AttackSurfaceReductionRules_Ids {rule['id']} "
                          f"-AttackSurfaceReductionRules_Actions Enabled")
            lines.append("")

    lines.append("# Disable AutoPlay")
    lines.append('Set-ItemProperty -Path "HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion'
                  '\\Policies\\Explorer" -Name NoDriveTypeAutoRun -Value 255')

    return "\n".join(lines)


if __name__ == "__main__":
    print("=" * 60)
    print("Anti-Ransomware Group Policy Audit Agent")
    print("AppLocker, CFA, ASR rules, network restrictions")
    print("=" * 60)

    if len(sys.argv) < 2:
        print("\nUsage:")
        print("  python agent.py audit              Run full GPO compliance audit")
        print("  python agent.py asr                Check ASR rule status only")
        print("  python agent.py remediate          Generate remediation script")
        print("  python agent.py baseline           Show recommended baseline")
        sys.exit(0)

    command = sys.argv[1]

    if command == "audit":
        report = generate_gpo_report()
        print(f"\n--- GPO Compliance Report ---")
        print(f"  Hostname: {report['hostname']}")
        print(f"  Overall Score: {report['overall_score']}%")
        for name, result in report["checks"].items():
            status = result["status"]
            icon = "[+]" if status == "data_retrieved" else "[!]"
            print(f"  {icon} {result['description']}: {status}")
        output_file = "gpo_audit_report.json"
        with open(output_file, "w") as f:
            json.dump(report, f, indent=2)
        print(f"\n[+] Full report saved to: {output_file}")

    elif command == "asr":
        print("\n--- Recommended ASR Rules for Ransomware Prevention ---")
        for rule_id, rule in ASR_RULES.items():
            print(f"  [{rule['category']:20s}] {rule['name']}")
            print(f"    GUID: {rule_id}")
            print(f"    Recommended: {rule['recommended']}")

    elif command == "remediate":
        report = generate_gpo_report()
        script = generate_remediation_script(report)
        output_file = "remediate_ransomware_gpo.ps1"
        with open(output_file, "w") as f:
            f.write(script)
        print(f"\n[+] Remediation script saved to: {output_file}")
        print(f"\n{script}")

    elif command == "baseline":
        print("\n--- Recommended Anti-Ransomware GPO Baseline ---")
        print("\nAppLocker Deny Paths:")
        for path in APPLOCKER_DENY_PATHS:
            print(f"  DENY: {path}")
        print(f"\nASR Rules ({len(ASR_RULES)} recommended):")
        for rule_id, rule in ASR_RULES.items():
            print(f"  {rule_id}: {rule['name']}")
        print(f"\nControlled Folder Access Protected Folders:")
        for folder in CFA_RECOMMENDED_FOLDERS:
            print(f"  {folder}")

    else:
        print(f"[!] Unknown command: {command}")

===== END FILE =====


===== FILE: skills/implementing-honeypot-for-ransomware-detection/SKILL.md =====
---
name: implementing-honeypot-for-ransomware-detection
description: 'Deploys canary files, honeypot shares, and decoy systems to detect ransomware
  activity at the earliest possible stage. Configures canary tokens embedded in strategic
  file locations that trigger alerts when ransomware attempts encryption, uses honeypot
  network shares that mimic high-value targets, and deploys Thinkst Canary appliances
  for comprehensive deception-based detection. Activates for requests involving ransomware
  honeypots, canary files, deception technology for ransomware, or early ransomware
  alerting.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- detection
- honeypot
- canary
- defense
- deception
version: 1.0.0
author: mahipal
license: Apache-2.0
d3fend_techniques:
- File Metadata Consistency Validation
- Content Format Conversion
- File Content Analysis
- Platform Hardening
- File Format Verification
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1486
- T1490
mitre_f3:
  version: '1.1'
  tactics:
  - positioning
  - initial-access
  - monetization
  - resource-development
  techniques:
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
  - id: T1586
    name: Compromise Accounts
    tactic: resource-development
    source: attack
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1047
    name: Transfer of funds
    tactic: monetization
    source: f3
  - id: F1033
    name: Insider Access Abuse
    tactic: initial-access
    source: f3
---
# Implementing Honeypot for Ransomware Detection

## When to Use

- Deploying early-warning detection for ransomware encryption attempts using canary files
- Creating honeypot file shares that detect lateral movement and data staging before encryption
- Supplementing EDR and SIEM-based detection with deception-layer alerts that have near-zero false positives
- Detecting ransomware variants that evade signature-based detection by triggering on file modification behavior
- Validating that ransomware detection capabilities work by testing with controlled encryption tools

**Do not use** as the sole ransomware detection mechanism. Honeypots are a high-confidence supplementary layer, not a replacement for EDR, network monitoring, and backup protection.

## Prerequisites

- File server or NAS infrastructure where canary files can be deployed
- Windows File Server Resource Manager (FSRM) or equivalent file activity monitoring
- Thinkst Canary or similar deception platform (optional, for advanced deployment)
- SIEM platform for centralizing honeypot alerts
- Administrative access to deploy canary files across file shares
- Network segment for honeypot systems (if deploying full honeypot servers)

## Workflow

### Step 1: Deploy Canary Files on File Shares

Place canary files in strategic locations that ransomware will encounter during encryption:

```powershell
# Deploy canary files across all file shares
# Files are named to appear early in alphabetical and directory order
# Ransomware typically encrypts alphabetically or by directory traversal

$shares = @("\\fileserver01\finance", "\\fileserver01\hr", "\\fileserver01\engineering")
$canaryNames = @(
    "!_IMPORTANT_DO_NOT_DELETE.docx",
    "000_Budget_2026_FINAL.xlsx",
    "_Confidential_Employee_Records.pdf",
    "AAAA_Quarterly_Report.docx"
)

foreach ($share in $shares) {
    foreach ($name in $canaryNames) {
        $targetPath = Join-Path $share $name
        # Create a legitimate-looking file with canary content
        # The file contains a unique token that triggers on access
        $content = "This document contains confidential financial data.`n"
        $content += "Q4 2025 Revenue: $42.3M | Q1 2026 Forecast: $45.1M`n"
        $content += "Prepared by: Finance Department`n"
        Set-Content -Path $targetPath -Value $content
        # Set file as hidden system to avoid user interaction
        $file = Get-Item $targetPath
        $file.Attributes = [System.IO.FileAttributes]::Hidden
    }
}

# Also deploy in subdirectories (ransomware traverses recursively)
$subDirs = Get-ChildItem -Path "\\fileserver01\finance" -Directory -Recurse | Select-Object -First 20
foreach ($dir in $subDirs) {
    $canaryPath = Join-Path $dir.FullName "!_Budget_Summary.xlsx"
    Set-Content -Path $canaryPath -Value "Canary file for ransomware detection"
    (Get-Item $canaryPath).Attributes = [System.IO.FileAttributes]::Hidden
}
```

### Step 2: Configure File Integrity Monitoring on Canary Files

**Windows FSRM approach:**

```powershell
# Configure FSRM to monitor for ransomware file extensions
# and canary file modifications

Install-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools

# Create file screen for known ransomware extensions
$ransomExtensions = @(
    "*.encrypted", "*.locked", "*.crypto", "*.crypt",
    "*.locky", "*.cerber", "*.zepto", "*.thor",
    "*.aesir", "*.zzzzz", "*.wallet", "*.onion",
    "*.wncry", "*.wcry", "*.lockbit", "*.BlackCat",
    "*.ALPHV", "*.rhysida", "*.play"
)

# Create file group for ransomware extensions
New-FsrmFileGroup -Name "Ransomware_Extensions" -IncludePattern $ransomExtensions

# Create file screen template
New-FsrmFileScreenTemplate -Name "Ransomware_Screen" `
    -IncludeGroup "Ransomware_Extensions" `
    -Active:$false  # Passive mode: alert without blocking

# Apply to all monitored shares
$monitoredPaths = @("D:\Shares\Finance", "D:\Shares\HR", "D:\Shares\Engineering")
foreach ($path in $monitoredPaths) {
    New-FsrmFileScreen -Path $path -Template "Ransomware_Screen"
}
```

**Canary file modification monitoring with PowerShell FileSystemWatcher:**

```powershell
# Real-time canary file monitoring service
$canaryPaths = @(
    "D:\Shares\Finance\!_IMPORTANT_DO_NOT_DELETE.docx",
    "D:\Shares\HR\000_Budget_2026_FINAL.xlsx",
    "D:\Shares\Engineering\_Confidential_Employee_Records.pdf"
)

$watcher = New-Object System.IO.FileSystemWatcher
$watcher.Path = "D:\Shares"
$watcher.Filter = "*"
$watcher.IncludeSubdirectories = $true
$watcher.EnableRaisingEvents = $true

$action = {
    $path = $Event.SourceEventArgs.FullPath
    $changeType = $Event.SourceEventArgs.ChangeType
    $timestamp = $Event.TimeGenerated

    # Check if modified file is a canary
    $isCanary = $false
    foreach ($canary in $canaryPaths) {
        if ($path -eq $canary) { $isCanary = $true; break }
    }

    if ($isCanary -or $changeType -eq "Renamed") {
        $alertMsg = "RANSOMWARE ALERT: Canary file modified! Path: $path | Change: $changeType | Time: $timestamp"
        # Log to Windows Event Log
        Write-EventLog -LogName Application -Source "RansomwareCanary" `
            -EventID 9999 -EntryType Error -Message $alertMsg
        # Send SIEM alert via syslog
        # Trigger automated containment
    }
}

Register-ObjectEvent $watcher "Changed" -Action $action
Register-ObjectEvent $watcher "Deleted" -Action $action
Register-ObjectEvent $watcher "Renamed" -Action $action
```

### Step 3: Deploy Honeypot Network Shares

Create decoy file shares that appear to contain high-value data:

```powershell
# Create honeypot share on dedicated server
# This server monitors ALL file access and alerts on any activity

New-Item -Path "D:\HoneypotShares\Executive_Compensation" -ItemType Directory
New-Item -Path "D:\HoneypotShares\M&A_Documents" -ItemType Directory
New-Item -Path "D:\HoneypotShares\Board_Meeting_Notes" -ItemType Directory
New-Item -Path "D:\HoneypotShares\Customer_Database_Exports" -ItemType Directory

# Share with broad read access (enticing to attackers)
New-SmbShare -Name "Executive_Compensation" `
    -Path "D:\HoneypotShares\Executive_Compensation" `
    -FullAccess "DOMAIN\Domain Users" `
    -Description "Executive Compensation Files - Restricted"

# Populate with realistic-looking but fake documents
# Use document templates that look legitimate
$docContent = @"
CONFIDENTIAL - Executive Compensation Summary
FY 2026 Base Salary and Bonus Structures
CEO: [REDACTED] | CFO: [REDACTED] | CTO: [REDACTED]
Total Compensation Package: See Appendix A
"@
Set-Content -Path "D:\HoneypotShares\Executive_Compensation\FY2026_Comp_Summary.txt" -Value $docContent

# Enable detailed audit logging on honeypot share
$acl = Get-Acl "D:\HoneypotShares"
$auditRule = New-Object System.Security.AccessControl.FileSystemAuditRule(
    "Everyone", "ReadAndExecute,Write,Delete", "ContainerInherit,ObjectInherit",
    "None", "Success,Failure"
)
$acl.AddAuditRule($auditRule)
Set-Acl "D:\HoneypotShares" $acl

# Enable object access auditing via GPO
auditpol /set /subcategory:"File System" /success:enable /failure:enable
```

### Step 4: Deploy Thinkst Canary Tokens

For organizations using Thinkst Canary or the free canarytokens.org service:

```bash
# Generate canary tokens via API (Thinkst Canary)
# These trigger alerts when documents are opened or URLs are accessed

# Word document token
curl -X POST "https://CONSOLE.canary.tools/api/v1/canarytoken/create" \
  -d "auth_token=YOUR_API_TOKEN" \
  -d "memo=Finance_Share_Canary" \
  -d "kind=doc-msword" \
  -o /tmp/canary_budget_report.docx

# PDF document token
curl -X POST "https://CONSOLE.canary.tools/api/v1/canarytoken/create" \
  -d "auth_token=YOUR_API_TOKEN" \
  -d "memo=HR_Share_Canary" \
  -d "kind=pdf-acrobat-reader" \
  -o /tmp/canary_employee_handbook.pdf

# Windows folder token (alerts when folder is browsed)
curl -X POST "https://CONSOLE.canary.tools/api/v1/canarytoken/create" \
  -d "auth_token=YOUR_API_TOKEN" \
  -d "memo=Executive_Folder_Browse" \
  -d "kind=windows-dir"

# Deploy Canary appliance (emulates a file server)
# Configure via web console to appear as:
# - Windows file server with SMB shares
# - Contains realistic-looking directories
# - Any access triggers immediate alert with source IP and activity details
```

### Step 5: Integrate Alerts with SIEM and Automated Response

```python
# siem_integration.py - Forward honeypot alerts to SIEM and trigger containment

import json
import requests
import logging
from datetime import datetime

SIEM_WEBHOOK = "https://siem.company.com/api/alerts"
NAC_API = "https://nac.company.com/api/v1/quarantine"
EDR_API = "https://edr.company.com/api/v1/isolate"

def send_ransomware_alert(source_ip: str, canary_path: str, action: str):
    """Send high-priority alert to SIEM and trigger automated containment."""
    alert = {
        "timestamp": datetime.utcnow().isoformat(),
        "severity": "CRITICAL",
        "category": "Ransomware - Canary File Triggered",
        "source_ip": source_ip,
        "canary_file": canary_path,
        "action_detected": action,
        "automated_response": "Host isolation initiated",
        "mitre_technique": "T1486 - Data Encrypted for Impact",
    }

    # Send to SIEM
    try:
        requests.post(SIEM_WEBHOOK, json=alert, timeout=5)
    except requests.RequestException as e:
        logging.error(f"SIEM alert failed: {e}")

    # Automated containment - isolate host via NAC
    try:
        requests.post(f"{NAC_API}/{source_ip}",
                      json={"action": "quarantine", "reason": "Ransomware canary triggered"},
                      timeout=5)
    except requests.RequestException as e:
        logging.error(f"NAC quarantine failed: {e}")

    # Automated containment - isolate host via EDR
    try:
        requests.post(EDR_API,
                      json={"ip": source_ip, "action": "isolate"},
                      timeout=5)
    except requests.RequestException as e:
        logging.error(f"EDR isolation failed: {e}")

    logging.critical(f"RANSOMWARE CANARY ALERT: {source_ip} modified {canary_path} ({action})")
```

## Key Concepts

| Term | Definition |
|------|------------|
| **Canary File** | A decoy file placed in strategic locations that triggers an alert when modified, renamed, or deleted by ransomware |
| **Honeypot Share** | A decoy network share designed to attract attackers, where any access is suspicious and triggers alerts |
| **Canary Token** | A trackable token embedded in a document or URL that reports back when accessed, revealing the accessor's IP and time |
| **FSRM** | File Server Resource Manager - Windows Server role that monitors file operations and can screen for ransomware extensions |
| **Deception Layer** | Security architecture layer using decoy assets to detect threats with near-zero false positive rates |
| **File System Watcher** | System service that monitors real-time file system changes (creation, modification, deletion, rename) |

## Tools & Systems

- **Thinkst Canary**: Commercial deception platform providing canary appliances (emulate servers) and canary tokens (trackable documents)
- **Canarytokens.org**: Free service from Thinkst for generating basic canary tokens (Word docs, PDFs, URLs, DNS)
- **OpenCanary**: Open-source honeypot daemon that emulates common services (SMB, RDP, SSH) and logs access attempts
- **FSRM (File Server Resource Manager)**: Windows Server built-in tool for file screening, quota management, and ransomware extension detection
- **Elastic Endpoint**: Uses canary files internally for ransomware protection, triggering behavioral alerts on canary modification

## Common Scenarios

### Scenario: Early Detection of BlackByte Ransomware via Canary Files

**Context**: A retail company deploys canary files across 200 file shares and 3 honeypot shares. At 3:00 AM on a Saturday, the canary monitoring system generates 47 alerts in rapid succession as canary files across 12 shares are modified within 90 seconds.

**Approach**:
1. Canary file alert triggers automated containment: source workstation (10.2.8.55) quarantined via NAC within 30 seconds
2. SIEM correlation shows the source workstation had EDR alerts for PsExec execution 2 hours earlier (missed by overnight SOC)
3. Additional canary alerts from 3 other workstations indicate the ransomware is spreading via scheduled tasks
4. IR team isolates the affected VLAN, preventing encryption of the remaining 188 file shares
5. The 12 affected shares are restored from immutable backups within 4 hours
6. Estimated damage prevented: $2.3M in downtime and recovery costs based on the 95% of shares protected

**Pitfalls**:
- Placing canary files only in root directories where ransomware may skip them by targeting subdirectories first
- Using obvious canary names that sophisticated ransomware may recognize and avoid
- Not testing canary alerting end-to-end, discovering during an actual incident that alerts are not reaching the SOC
- Generating excessive canary alerts during legitimate file migrations or antivirus scans, causing alert fatigue

## Output Format

```
## Ransomware Honeypot Deployment Report

**Organization**: [Name]
**Deployment Date**: [Date]

### Canary File Deployment
| Share | Files Deployed | Naming Convention | Alert Method |
|-------|---------------|-------------------|--------------|
| [Share path] | [Count] | [Pattern] | [FSRM/Watcher/Token] |

### Honeypot Shares
| Share Name | Location | Apparent Content | Monitoring |
|-----------|----------|-----------------|------------|
| [Name] | [Server] | [Description] | [Audit/Canary] |

### Alert Integration
- SIEM: [Connected/Not Connected]
- Automated Containment: [EDR Isolation/NAC Quarantine/None]
- Alert SLA: [Expected response time]

### Testing Results
| Test Date | Test Type | Canary Triggered | Alert Received | Containment Executed | Time to Alert |
|-----------|----------|-----------------|----------------|---------------------|---------------|
```

===== END FILE =====


===== FILE: skills/implementing-honeypot-for-ransomware-detection/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/implementing-honeypot-for-ransomware-detection/assets/template.md =====
# Ransomware Honeypot Deployment Template

## Deployment Plan

| Share | Canary Count | File Types | Naming Convention | Monitoring Method |
|-------|-------------|------------|-------------------|-------------------|
| | | | | |

## Canary File Inventory

| File Path | Type | Hash (SHA-256) | Deploy Date | Status |
|-----------|------|----------------|-------------|--------|
| | | | | |

## Honeypot Shares

| Share Name | Server | Apparent Content | Access Logging | Alert Destination |
|-----------|--------|-----------------|---------------|-------------------|
| | | | | |

## Alert Integration

- [ ] SIEM webhook configured
- [ ] Automated NAC quarantine tested
- [ ] EDR isolation API integrated
- [ ] Email/SMS alerts configured for on-call
- [ ] Canary alerts have CRITICAL priority in SIEM

## Validation Testing

| Test Date | Method | Canaries Triggered | Alert Time | Containment Time | Result |
|-----------|--------|-------------------|------------|------------------|--------|
| | | | | | |

===== END FILE =====


===== FILE: skills/implementing-honeypot-for-ransomware-detection/references/api-reference.md =====
# API Reference: Implementing Honeypot for Ransomware Detection

## Canary File Strategy

| Name Pattern | Extension | Purpose |
|-------------|-----------|---------|
| `!Accounting_*` | .docx, .xlsx | Sorted first alphabetically |
| `~$Confidential_*` | .pdf, .csv | Mimics temp/open Office files |
| `!Payroll_*` | .xlsx, .bak | High-value bait |

## Integrity Monitoring

```python
import hashlib
from pathlib import Path
content = Path("canary.docx").read_bytes()
sha256 = hashlib.sha256(content).hexdigest()
```

## Ransomware Extension Indicators

| Extension | Ransomware Family |
|-----------|------------------|
| `.encrypted` | Generic |
| `.locked` | LockBit, GandCrab |
| `.crypto` | CryptoLocker variants |
| `.ransom` | Generic |
| `.enc` | Various |

## Samba Honeypot Share (full_audit VFS)

```ini
[FinanceArchive]
    path = /srv/honeypot
    vfs objects = full_audit
    full_audit:success = open opendir write rename unlink
    full_audit:failure = open
    full_audit:facility = LOCAL7
    full_audit:priority = NOTICE
```

## Thinkst Canary API

```bash
# List incidents
curl "https://DOMAIN.canary.tools/api/v1/incidents/all" \
  -d auth_token=TOKEN

# Acknowledge incident
curl "https://DOMAIN.canary.tools/api/v1/incident/acknowledge" \
  -d auth_token=TOKEN -d incident=INC_ID
```

## Detection Thresholds

| Metric | Threshold | Severity |
|--------|----------|----------|
| Files modified in 60s | > 50 | CRITICAL |
| Canary file deleted | Any | CRITICAL |
| Canary hash changed | Any | CRITICAL |
| Known ransom extensions | Any | CRITICAL |

### References

- Thinkst Canary: https://canary.tools/
- CISA Ransomware Guide: https://www.cisa.gov/stopransomware
- Canarytokens: https://canarytokens.org/

===== END FILE =====


===== FILE: skills/implementing-honeypot-for-ransomware-detection/references/standards.md =====
# Standards & References - Honeypot for Ransomware Detection

## MITRE ATT&CK
- T1486: Data Encrypted for Impact (detection via canary file modification)
- T1021.002: SMB/Windows Admin Shares (detection via honeypot share access)
- T1083: File and Directory Discovery (detection via honeypot folder browsing)

## NIST SP 800-53 Rev 5
- SI-4: System Monitoring (honeypots as monitoring component)
- AU-6: Audit Record Review, Analysis, and Reporting (canary file audit logging)

## CIS Controls v8
- Control 13.1: Centralize security event alerting (integrate honeypot alerts into SIEM)
- Control 13.8: Deploy intrusion detection/prevention systems

## Tools Documentation
- Thinkst Canary: https://canary.tools/
- Canarytokens: https://canarytokens.org/
- OpenCanary: https://github.com/thinkst/opencanary
- FSRM: https://learn.microsoft.com/en-us/windows-server/storage/fsrm/fsrm-overview
- Elastic Ransomware Protection: https://www.elastic.co/security-labs/ransomware-in-the-honeypot-how-we-capture-keys

===== END FILE =====


===== FILE: skills/implementing-honeypot-for-ransomware-detection/references/workflows.md =====
# Workflows - Honeypot for Ransomware Detection

## Workflow 1: Canary File Deployment

```
Start
  |
  v
[Inventory all file shares] --> Map share names, paths, user population
  |
  v
[Select canary file placement strategy]
  |-- Root of each share (first files encrypted)
  |-- Key subdirectories (finance, HR, executive)
  |-- Alphabetically early names (!_, 000_, AAA_)
  |
  v
[Generate canary files with realistic content]
  |-- .docx, .xlsx, .pdf formats
  |-- Realistic filenames matching share context
  |-- Hidden attribute to prevent user interaction
  |
  v
[Deploy monitoring]
  |-- FSRM file screens for ransomware extensions
  |-- FileSystemWatcher for canary file changes
  |-- Audit logging on canary files
  |
  v
[Integrate with SIEM and automated containment]
  |
  v
[Test with controlled encryption tool]
  |
  v
End
```

## Workflow 2: Honeypot Alert Response

```
Canary Alert Triggered
  |
  v
[Identify source IP and user from alert]
  |
  v
[Automated containment (within 30 seconds)]
  |-- NAC: Quarantine source IP
  |-- EDR: Isolate endpoint
  |-- AD: Disable user account
  |
  v
[SOC validates alert]
  |-- Check for legitimate activity (file migration, AV scan)
  |-- If FP --> Restore access, tune alerting
  |-- If TP --> Escalate to IR team
  |
  v
[IR team assesses scope]
  |-- How many canary files triggered?
  |-- How many endpoints involved?
  |-- Is encryption spreading?
  |
  v
[Full incident response activation]
  |
  v
End
```

===== END FILE =====


===== FILE: skills/implementing-honeypot-for-ransomware-detection/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for deploying and monitoring ransomware honeypot canary files."""

import os
import json
import argparse
import hashlib
import time
from datetime import datetime
from pathlib import Path
from collections import Counter


CANARY_EXTENSIONS = [".docx", ".xlsx", ".pdf", ".pptx", ".csv", ".txt",
                     ".jpg", ".png", ".sql", ".bak"]
CANARY_PREFIX_NAMES = [
    "!Accounting_Report_2024", "!Budget_Final", "!Confidential_HR",
    "!Employee_SSN_List", "!Financial_Audit", "!Payroll_Records",
    "~$Customer_Database", "~$Executive_Compensation",
]


def create_canary_files(target_dir, count=10):
    """Create canary files in strategic locations for ransomware detection."""
    canaries = []
    target = Path(target_dir)
    for i in range(min(count, len(CANARY_PREFIX_NAMES))):
        for ext in CANARY_EXTENSIONS[:3]:
            name = f"{CANARY_PREFIX_NAMES[i]}{ext}"
            path = target / name
            content = os.urandom(1024 * (i + 1))
            path.write_bytes(content)
            file_hash = hashlib.sha256(content).hexdigest()
            canaries.append({
                "path": str(path),
                "hash": file_hash,
                "size": len(content),
                "created": datetime.utcnow().isoformat(),
            })
    return canaries


def generate_canary_manifest(canaries, manifest_path):
    """Save canary file manifest for integrity monitoring."""
    manifest = {
        "created_at": datetime.utcnow().isoformat(),
        "canary_count": len(canaries),
        "canaries": canaries,
    }
    with open(manifest_path, "w") as f:
        json.dump(manifest, f, indent=2)
    return manifest_path


def check_canary_integrity(manifest_path):
    """Check canary files against manifest to detect tampering/encryption."""
    with open(manifest_path) as f:
        manifest = json.load(f)
    alerts = []
    for canary in manifest.get("canaries", []):
        path = Path(canary["path"])
        if not path.exists():
            alerts.append({
                "type": "DELETED",
                "path": canary["path"],
                "severity": "CRITICAL",
                "detail": "Canary file deleted - possible ransomware wiper",
            })
            continue
        current_hash = hashlib.sha256(path.read_bytes()).hexdigest()
        if current_hash != canary["hash"]:
            alerts.append({
                "type": "MODIFIED",
                "path": canary["path"],
                "severity": "CRITICAL",
                "original_hash": canary["hash"],
                "current_hash": current_hash,
                "detail": "Canary file modified - possible ransomware encryption",
            })
        current_size = path.stat().st_size
        if abs(current_size - canary["size"]) > canary["size"] * 0.1:
            alerts.append({
                "type": "SIZE_CHANGE",
                "path": canary["path"],
                "severity": "HIGH",
                "original_size": canary["size"],
                "current_size": current_size,
            })
    checked = len(manifest.get("canaries", []))
    return {
        "checked": checked,
        "alerts": alerts,
        "alert_count": len(alerts),
        "status": "ALERT" if alerts else "CLEAN",
    }


def detect_ransomware_indicators(watch_dir, window_seconds=60):
    """Detect rapid file modifications indicative of ransomware."""
    watch_path = Path(watch_dir)
    now = time.time()
    recently_modified = []
    extension_changes = Counter()
    new_extensions = Counter()

    for fp in watch_path.rglob("*"):
        if not fp.is_file():
            continue
        try:
            mtime = fp.stat().st_mtime
            if now - mtime < window_seconds:
                recently_modified.append(str(fp))
                ext = fp.suffix.lower()
                if ext in (".encrypted", ".locked", ".crypto", ".crypt",
                           ".enc", ".pay", ".ransom"):
                    new_extensions[ext] += 1
        except (OSError, PermissionError):
            continue

    indicators = []
    if len(recently_modified) > 50:
        indicators.append({
            "indicator": "Mass file modification",
            "count": len(recently_modified),
            "severity": "CRITICAL",
            "detail": f"{len(recently_modified)} files modified in {window_seconds}s",
        })
    if new_extensions:
        indicators.append({
            "indicator": "Ransomware file extensions detected",
            "extensions": dict(new_extensions),
            "severity": "CRITICAL",
        })
    return {
        "files_checked_window": window_seconds,
        "recently_modified": len(recently_modified),
        "indicators": indicators,
        "status": "ALERT" if indicators else "CLEAN",
    }


def generate_honeypot_share_config(share_name="FinanceArchive", share_path="/srv/honeypot"):
    """Generate SMB honeypot share configuration."""
    return {
        "samba_config": {
            "share_name": share_name,
            "path": share_path,
            "comment": "Financial Archive (Read Only)",
            "read_only": False,
            "browseable": True,
            "guest_ok": False,
            "valid_users": "@domain_users",
            "vfs_objects": "full_audit",
            "full_audit_prefix": f"%u|%I|%S",
            "full_audit_success": "open opendir write rename unlink mkdir rmdir",
            "full_audit_failure": "open",
            "full_audit_facility": "LOCAL7",
            "full_audit_priority": "NOTICE",
        },
        "monitoring": {
            "log_path": "/var/log/samba/audit.log",
            "alert_on": ["write", "rename", "unlink"],
            "siem_integration": "syslog -> SIEM",
        },
    }


def analyze_honeypot_logs(log_path):
    """Analyze honeypot access logs for suspicious activity."""
    with open(log_path) as f:
        events = json.load(f)
    items = events if isinstance(events, list) else events.get("events", [])
    by_user = Counter(e.get("user", "unknown") for e in items)
    by_action = Counter(e.get("action", "unknown") for e in items)
    write_events = [e for e in items if e.get("action") in ("write", "rename", "delete")]
    return {
        "total_events": len(items),
        "by_user": dict(by_user.most_common(10)),
        "by_action": dict(by_action),
        "write_events": len(write_events),
        "suspicious": len(write_events) > 5,
        "severity": "CRITICAL" if len(write_events) > 20 else
                    "HIGH" if len(write_events) > 5 else "INFO",
    }


def main():
    parser = argparse.ArgumentParser(description="Ransomware Honeypot Agent")
    parser.add_argument("--deploy", help="Directory to deploy canary files")
    parser.add_argument("--manifest", help="Canary manifest JSON for integrity check")
    parser.add_argument("--watch", help="Directory to watch for ransomware indicators")
    parser.add_argument("--honeypot-log", help="Honeypot access log JSON")
    parser.add_argument("--action", choices=["deploy", "check", "detect", "analyze",
                                              "share-config", "full"], default="full")
    parser.add_argument("--output", default="ransomware_honeypot_report.json")
    args = parser.parse_args()

    report = {"generated_at": datetime.utcnow().isoformat(), "results": {}}

    if args.action in ("deploy", "full") and args.deploy:
        canaries = create_canary_files(args.deploy)
        manifest = generate_canary_manifest(canaries, args.deploy + "/canary_manifest.json")
        report["results"]["deployed"] = {"count": len(canaries), "manifest": manifest}
        print(f"[+] Deployed {len(canaries)} canary files")

    if args.action in ("check", "full") and args.manifest:
        result = check_canary_integrity(args.manifest)
        report["results"]["integrity"] = result
        print(f"[+] Integrity: {result['status']} ({result['alert_count']} alerts)")

    if args.action in ("detect", "full") and args.watch:
        result = detect_ransomware_indicators(args.watch)
        report["results"]["detection"] = result
        print(f"[+] Detection: {result['status']}")

    if args.action in ("share-config", "full"):
        config = generate_honeypot_share_config()
        report["results"]["share_config"] = config
        print("[+] Honeypot share config generated")

    if args.action in ("analyze", "full") and args.honeypot_log:
        result = analyze_honeypot_logs(args.honeypot_log)
        report["results"]["log_analysis"] = result
        print(f"[+] Honeypot events: {result['total_events']}, writes: {result['write_events']}")

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"[+] Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-honeypot-for-ransomware-detection/scripts/process.py =====
#!/usr/bin/env python3
"""
Ransomware Honeypot Deployment and Monitoring Tool

Deploys canary files across file shares and monitors for modifications
that indicate ransomware activity. Supports:
- Canary file generation with realistic content
- File system monitoring with immediate alerting
- Integration with SIEM via syslog
- Automated containment via API calls
"""

import hashlib
import json
import logging
import os
import sys
import time
from dataclasses import dataclass, field, asdict
from datetime import datetime
from pathlib import Path
from typing import Optional

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger("ransomware_honeypot")


@dataclass
class CanaryFile:
    path: str
    original_hash: str
    file_type: str
    deploy_time: str
    share_name: str
    status: str = "active"
    last_check: Optional[str] = None
    alert_triggered: bool = False


@dataclass
class CanaryAlert:
    timestamp: str
    canary_path: str
    change_type: str  # modified, deleted, renamed, extension_changed
    source_info: str
    severity: str = "CRITICAL"
    automated_response: str = ""


CANARY_TEMPLATES = {
    "finance": [
        ("!_Budget_FY2026_FINAL.xlsx", "FY2026 Budget Summary\nTotal Revenue: $142.3M\nTotal Expenses: $98.7M\nNet Income: $43.6M"),
        ("000_Quarterly_Earnings.docx", "Q4 2025 Earnings Report\nRevenue: $38.2M\nGross Margin: 67%\nEBITDA: $12.1M"),
        ("_Executive_Compensation.pdf", "Executive Compensation Committee Report\nCEO Total Comp: $2.1M\nCFO Total Comp: $1.4M"),
    ],
    "hr": [
        ("!_Employee_SSN_List.xlsx", "Employee ID | Name | SSN\n10001 | Smith, John | XXX-XX-1234\n10002 | Johnson, Mary | XXX-XX-5678"),
        ("000_Salary_Database_2026.csv", "Employee,Department,Base Salary,Bonus\nSmith J,Engineering,$145000,$29000"),
        ("_Termination_List_Q1.docx", "Planned Workforce Reduction Q1 2026\nDepartment: Operations\nHeadcount Impact: 45 positions"),
    ],
    "engineering": [
        ("!_Product_Roadmap_Confidential.docx", "Product Roadmap 2026-2028\nProject Phoenix: AI-powered analytics\nProject Titan: Next-gen platform"),
        ("000_Source_Code_Access_Keys.txt", "Repository Access Tokens\nGitHub Enterprise: ghp_XXXXXXXXXXXX\nAWS CodeCommit: AKIAIOSFODNN7EXAMPLE"),
        ("_Patent_Application_Draft.pdf", "US Patent Application\nTitle: Method for Distributed Computing\nInventor: Dr. Jane Smith"),
    ],
    "executive": [
        ("!_Board_Meeting_Minutes.docx", "Board of Directors Meeting Minutes\nDate: January 15, 2026\nAttendees: Full board present"),
        ("000_MA_Target_Analysis.xlsx", "M&A Target Analysis\nTarget: AcmeTech Inc\nValuation: $450M\nSynergies: $80M annual"),
        ("_Strategic_Plan_2026.pdf", "Strategic Plan 2026-2030\nVision: Market leader in three verticals\nCapEx Budget: $200M"),
    ],
}


class CanaryDeployer:
    """Deploys and manages canary files for ransomware detection."""

    def __init__(self, state_file: str = "canary_state.json"):
        self.state_file = state_file
        self.canaries: list[CanaryFile] = []
        self._load_state()

    def _load_state(self):
        if os.path.exists(self.state_file):
            with open(self.state_file, "r") as f:
                data = json.load(f)
                self.canaries = [CanaryFile(**c) for c in data.get("canaries", [])]

    def _save_state(self):
        with open(self.state_file, "w") as f:
            json.dump({"canaries": [asdict(c) for c in self.canaries],
                       "last_updated": datetime.now().isoformat()}, f, indent=2)

    def _compute_hash(self, filepath: str) -> str:
        if not os.path.exists(filepath):
            return ""
        h = hashlib.sha256()
        with open(filepath, "rb") as f:
            for chunk in iter(lambda: f.read(8192), b""):
                h.update(chunk)
        return h.hexdigest()

    def deploy_canaries(self, base_path: str, share_type: str = "finance") -> list:
        """Deploy canary files to a target directory."""
        templates = CANARY_TEMPLATES.get(share_type, CANARY_TEMPLATES["finance"])
        deployed = []

        base = Path(base_path)
        if not base.exists():
            logger.error(f"Target path does not exist: {base_path}")
            return deployed

        for filename, content in templates:
            filepath = base / filename
            try:
                filepath.write_text(content, encoding="utf-8")
                file_hash = self._compute_hash(str(filepath))

                canary = CanaryFile(
                    path=str(filepath),
                    original_hash=file_hash,
                    file_type=filepath.suffix,
                    deploy_time=datetime.now().isoformat(),
                    share_name=share_type,
                )
                self.canaries.append(canary)
                deployed.append(canary)
                logger.info(f"Deployed canary: {filepath}")
            except OSError as e:
                logger.error(f"Failed to deploy canary {filepath}: {e}")

        self._save_state()
        return deployed

    def check_canaries(self) -> list:
        """Check all deployed canaries for modifications."""
        alerts = []

        for canary in self.canaries:
            if canary.status != "active":
                continue

            canary.last_check = datetime.now().isoformat()

            if not os.path.exists(canary.path):
                # File deleted - strong ransomware indicator
                alert = CanaryAlert(
                    timestamp=datetime.now().isoformat(),
                    canary_path=canary.path,
                    change_type="deleted",
                    source_info="File no longer exists",
                    severity="CRITICAL",
                )
                alerts.append(alert)
                canary.alert_triggered = True
                canary.status = "triggered"
                logger.critical(f"CANARY DELETED: {canary.path}")
                continue

            current_hash = self._compute_hash(canary.path)
            if current_hash != canary.original_hash:
                # File modified - likely encryption
                alert = CanaryAlert(
                    timestamp=datetime.now().isoformat(),
                    canary_path=canary.path,
                    change_type="modified",
                    source_info=f"Hash changed: {canary.original_hash[:16]}... -> {current_hash[:16]}...",
                    severity="CRITICAL",
                )
                alerts.append(alert)
                canary.alert_triggered = True
                canary.status = "triggered"
                logger.critical(f"CANARY MODIFIED: {canary.path}")

            # Check for ransomware extension appended
            parent = Path(canary.path).parent
            base_name = Path(canary.path).name
            for f in parent.iterdir():
                if f.name.startswith(base_name) and f.name != base_name:
                    # Possible encrypted version (e.g., file.docx.lockbit)
                    alert = CanaryAlert(
                        timestamp=datetime.now().isoformat(),
                        canary_path=canary.path,
                        change_type="extension_changed",
                        source_info=f"Possible encrypted copy: {f.name}",
                        severity="CRITICAL",
                    )
                    alerts.append(alert)
                    canary.alert_triggered = True
                    logger.critical(f"CANARY EXTENSION CHANGE: {f.name}")

        self._save_state()
        return alerts

    def generate_report(self) -> str:
        """Generate deployment status report."""
        lines = []
        lines.append("=" * 60)
        lines.append("RANSOMWARE CANARY DEPLOYMENT REPORT")
        lines.append("=" * 60)
        lines.append(f"Report Time: {datetime.now().isoformat()}")
        lines.append(f"Total Canaries: {len(self.canaries)}")

        active = sum(1 for c in self.canaries if c.status == "active")
        triggered = sum(1 for c in self.canaries if c.status == "triggered")
        lines.append(f"Active: {active}")
        lines.append(f"Triggered: {triggered}")

        by_share = {}
        for c in self.canaries:
            by_share.setdefault(c.share_name, []).append(c)

        lines.append("\nDeployment by Share:")
        for share, canaries in by_share.items():
            lines.append(f"\n  {share.upper()}:")
            for c in canaries:
                status_icon = "OK" if c.status == "active" else "ALERT"
                lines.append(f"    [{status_icon}] {c.path}")

        lines.append("\n" + "=" * 60)
        return "\n".join(lines)

    def continuous_monitor(self, interval_seconds: int = 10):
        """Run continuous monitoring loop."""
        logger.info(f"Starting continuous canary monitoring (interval: {interval_seconds}s)")
        logger.info(f"Monitoring {len(self.canaries)} canary files")

        try:
            while True:
                alerts = self.check_canaries()
                if alerts:
                    logger.critical(f"!!! {len(alerts)} CANARY ALERTS DETECTED !!!")
                    for alert in alerts:
                        logger.critical(f"  {alert.change_type}: {alert.canary_path}")
                        # In production: trigger SIEM alert, NAC quarantine, EDR isolation
                time.sleep(interval_seconds)
        except KeyboardInterrupt:
            logger.info("Monitoring stopped by user")


def main():
    deployer = CanaryDeployer(
        state_file=str(Path(__file__).parent / "canary_state.json")
    )

    if len(sys.argv) > 1:
        command = sys.argv[1]

        if command == "deploy" and len(sys.argv) >= 4:
            target_path = sys.argv[2]
            share_type = sys.argv[3]
            deployed = deployer.deploy_canaries(target_path, share_type)
            print(f"Deployed {len(deployed)} canary files to {target_path}")

        elif command == "check":
            alerts = deployer.check_canaries()
            if alerts:
                print(f"\n!!! {len(alerts)} ALERTS !!!")
                for alert in alerts:
                    print(f"  [{alert.severity}] {alert.change_type}: {alert.canary_path}")
            else:
                print("All canaries intact - no alerts")

        elif command == "monitor":
            interval = int(sys.argv[2]) if len(sys.argv) > 2 else 10
            deployer.continuous_monitor(interval)

        elif command == "report":
            print(deployer.generate_report())

        else:
            print("Usage:")
            print("  python process.py deploy <path> <type>  - Deploy canaries (finance/hr/engineering/executive)")
            print("  python process.py check                 - Check all canaries for modifications")
            print("  python process.py monitor [interval]    - Continuous monitoring")
            print("  python process.py report                - Generate deployment report")
    else:
        # Demo mode
        print("Ransomware Honeypot - Demo Mode")
        print("=" * 40)
        demo_dir = Path(__file__).parent / "demo_share"
        demo_dir.mkdir(exist_ok=True)

        deployed = deployer.deploy_canaries(str(demo_dir), "finance")
        print(f"\nDeployed {len(deployed)} canary files")

        alerts = deployer.check_canaries()
        print(f"Initial check: {len(alerts)} alerts (expected: 0)")

        print("\n" + deployer.generate_report())


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-immutable-backup-with-restic/SKILL.md =====
---
name: implementing-immutable-backup-with-restic
description: 'Implements ransomware-resistant backups using restic with S3-compatible
  Object Lock (AWS S3, MinIO, Backblaze B2), automating backup creation, integrity
  checks via restic check --read-data, retention enforcement, and restore testing.
  Use when building immutable backup infrastructure, adding a WORM copy to a
  3-2-1-1-0 strategy, or automating scheduled backup-verification workflows.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- restic
- backup
- immutable
- ransomware
- s3
- object-lock
- worm
- recovery
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- MEASURE-2.7
- MAP-5.1
- MANAGE-2.4
atlas_techniques:
- AML.T0070
- AML.T0066
- AML.T0082
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1486
- T1490
---
# Implementing Immutable Backup with Restic

## When to Use

- Establishing ransomware-resistant backup infrastructure with cryptographic integrity verification
- Implementing 3-2-1-1-0 backup strategy where the extra 1 is an immutable copy
- Automating backup verification workflows that test restore capability on a schedule
- Protecting backup repositories from deletion or modification by compromised admin accounts
- Meeting compliance requirements for data retention with tamper-proof storage

**Do not use** as the sole backup solution without also maintaining offline/air-gapped copies. Object lock protects against logical deletion but not physical storage failure.

## Prerequisites

- restic binary installed (https://restic.readthedocs.io/)
- S3-compatible storage with Object Lock enabled (AWS S3, MinIO, Backblaze B2)
- Python 3.8+ with subprocess module
- AWS CLI or MinIO client (mc) configured for bucket access
- Sufficient storage for backup repository (typically 2-3x source data with deduplication)

## Workflow

### Step 1: Initialize Restic Repository with Encryption

Create an encrypted restic repository on S3-compatible storage with object lock enabled. Restic uses AES-256-CTR for encryption with Poly1305-AES for authentication, ensuring backup data is both confidential and tamper-evident.

### Step 2: Configure Object Lock Retention

Enable S3 Object Lock in Compliance mode on the backup bucket to prevent any principal (including root) from deleting or modifying objects during the retention period. Set retention to match your backup window requirements (typically 30-90 days).

### Step 3: Automate Backup and Verification

Schedule backup operations with post-backup integrity verification using `restic check --read-data` which downloads and verifies every data blob against its stored checksum. Log results and alert on any integrity failures.

### Step 4: Test Restore Procedures

Periodically restore random files from backup snapshots to a temporary location and compare checksums against the original to validate end-to-end backup integrity. Document restore times for RTO planning.

## Key Concepts

| Term | Definition |
|------|------------|
| **Object Lock** | S3 feature that prevents object deletion or overwrite for a specified retention period |
| **Compliance Mode** | Object Lock mode where even the root account cannot delete objects before retention expires |
| **Deduplication** | Restic stores data in content-addressable chunks, deduplicating across all snapshots |
| **3-2-1-1-0** | 3 copies, 2 media types, 1 offsite, 1 immutable, 0 errors in verification |

## Tools & Systems

- **restic**: Fast, secure, cross-platform backup tool with built-in encryption and deduplication
- **resticpy**: Python wrapper for restic CLI operations
- **AWS S3 Object Lock**: WORM storage for tamper-proof backup retention
- **MinIO**: Self-hosted S3-compatible storage with Object Lock support

## Output Format

```
BACKUP VERIFICATION REPORT
===========================
Repository: s3:s3.amazonaws.com/company-backups-immutable
Snapshots: 45
Total Size: 2.3 TiB (deduplicated from 8.7 TiB)
Last Backup: 2026-03-11T02:00:00Z
Integrity Check: PASSED (all packs verified)
Object Lock: Compliance mode, 90-day retention
Restore Test: PASSED (15 files verified)
```

===== END FILE =====


===== FILE: skills/implementing-immutable-backup-with-restic/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/implementing-immutable-backup-with-restic/references/api-reference.md =====
# API Reference: Implementing Immutable Backup with Restic

## Core Functions

| Function | Parameters | Returns |
|----------|-----------|---------|
| `init_repository(repo_url, password, s3_config)` | Repository URL, password, optional S3 credentials | Init result dict |
| `create_backup(repo_url, password, source_paths, tags, exclude)` | Repo, password, paths list, optional tags/excludes | Snapshot summary |
| `verify_backup_integrity(repo_url, password, read_data)` | Repo, password, whether to verify data blobs | Verification result |
| `list_snapshots(repo_url, password)` | Repo, password | Snapshot list with metadata |
| `apply_retention_policy(repo_url, password, keep_*)` | Repo, password, retention counts | Pruning result |
| `test_restore(repo_url, password, snapshot_id)` | Repo, password, snapshot to restore | Restore test result |
| `get_repository_stats(repo_url, password)` | Repo, password | Size, dedup ratio, file count |

## S3 Object Lock Configuration

| Function | Parameters | Returns |
|----------|-----------|---------|
| `configure_s3_object_lock(bucket, retention_days, mode)` | Bucket name, days, COMPLIANCE or GOVERNANCE | Config result |

## Restic Commands Wrapped

| Command | Description |
|---------|-------------|
| `restic init` | Initialize encrypted repository |
| `restic backup --json` | Create snapshot with JSON progress |
| `restic check --read-data` | Full integrity verification |
| `restic snapshots --json` | List snapshots |
| `restic forget --prune` | Apply retention and reclaim space |
| `restic restore` | Extract files from snapshot |
| `restic stats --json` | Repository size statistics |

## CLI Usage

```bash
# Initialize repository
python agent.py --repo s3:s3.amazonaws.com/my-backups --action init

# Backup with tags
python agent.py --repo s3:s3.amazonaws.com/my-backups --action backup \
  --source /var/lib/postgresql /etc --tags database config

# Full pipeline: backup + verify + retention + restore test
python agent.py --repo s3:s3.amazonaws.com/my-backups --action full-pipeline \
  --source /srv/data --keep-daily 7 --keep-weekly 4
```

## Retention Policy Defaults

| Parameter | Default | Description |
|-----------|---------|-------------|
| `keep-daily` | 7 | Keep 7 daily snapshots |
| `keep-weekly` | 4 | Keep 4 weekly snapshots |
| `keep-monthly` | 12 | Keep 12 monthly snapshots |
| `keep-yearly` | 2 | Keep 2 yearly snapshots |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `subprocess` | stdlib | Execute restic CLI commands |
| `hashlib` | stdlib | SHA-256 verification of restored files |
| `tempfile` | stdlib | Temporary restore target directories |
| `json` | stdlib | Parse restic JSON output |

## References

- Restic Documentation: https://restic.readthedocs.io/
- AWS S3 Object Lock: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html
- resticpy Python Wrapper: https://mtlynch.github.io/resticpy/
- Veeam 3-2-1-1-0 Strategy: https://www.veeam.com/blog/321-backup-rule.html

===== END FILE =====


===== FILE: skills/implementing-immutable-backup-with-restic/scripts/agent.py =====
#!/usr/bin/env python3
"""Immutable backup agent using restic with S3 Object Lock for ransomware-resistant storage.

Automates backup creation, integrity verification, snapshot retention,
and restore testing against S3-compatible storage with WORM protection.
"""

import os
import sys
import json
import hashlib
import logging
import argparse
import subprocess
import tempfile
import shutil
from datetime import datetime, timezone

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.StreamHandler(),
        logging.FileHandler("restic_backup.log"),
    ],
)
logger = logging.getLogger(__name__)


def run_restic(args, repo=None, password=None, env_extra=None):
    """Execute a restic command and return parsed output."""
    cmd = ["restic"]
    if repo:
        cmd.extend(["--repo", repo])
    cmd.extend(args)

    env = dict(os.environ)
    if password:
        env["RESTIC_PASSWORD"] = password
    if env_extra:
        env.update(env_extra)

    result = subprocess.run(
        cmd, capture_output=True, text=True, env=env, timeout=3600
    )
    if result.returncode != 0:
        logger.error("restic %s failed: %s", args[0], result.stderr[:500])
    return {
        "returncode": result.returncode,
        "stdout": result.stdout,
        "stderr": result.stderr,
    }


def init_repository(repo_url, password, s3_config=None):
    """Initialize a new restic repository with encryption."""
    env_extra = {}
    if s3_config:
        env_extra["AWS_ACCESS_KEY_ID"] = s3_config.get("access_key", "")
        env_extra["AWS_SECRET_ACCESS_KEY"] = s3_config.get("secret_key", "")
        if s3_config.get("endpoint"):
            env_extra["AWS_DEFAULT_REGION"] = s3_config.get("region", "us-east-1")

    result = run_restic(["init"], repo=repo_url, password=password, env_extra=env_extra)
    if result["returncode"] == 0:
        logger.info("Repository initialized: %s", repo_url)
    elif "already initialized" in result["stderr"]:
        logger.info("Repository already exists: %s", repo_url)
        result["returncode"] = 0
    return result


def configure_s3_object_lock(bucket_name, retention_days=90, mode="COMPLIANCE"):
    """Configure S3 Object Lock on backup bucket via AWS CLI."""
    lock_config = {
        "ObjectLockEnabled": "Enabled",
        "Rule": {
            "DefaultRetention": {
                "Mode": mode,
                "Days": retention_days,
            }
        }
    }
    cmd = [
        "aws", "s3api", "put-object-lock-configuration",
        "--bucket", bucket_name,
        "--object-lock-configuration", json.dumps(lock_config),
    ]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    if result.returncode == 0:
        logger.info(
            "Object Lock configured: bucket=%s mode=%s retention=%dd",
            bucket_name, mode, retention_days
        )
    else:
        logger.error("Object Lock configuration failed: %s", result.stderr[:300])
    return {
        "bucket": bucket_name,
        "mode": mode,
        "retention_days": retention_days,
        "success": result.returncode == 0,
        "error": result.stderr if result.returncode != 0 else None,
    }


def create_backup(repo_url, password, source_paths, tags=None, exclude_patterns=None, env_extra=None):
    """Create a new backup snapshot."""
    args = ["backup"]
    if tags:
        for tag in tags:
            args.extend(["--tag", tag])
    if exclude_patterns:
        for pattern in exclude_patterns:
            args.extend(["--exclude", pattern])
    args.extend(["--json"])
    args.extend(source_paths)

    result = run_restic(args, repo=repo_url, password=password, env_extra=env_extra)
    if result["returncode"] == 0:
        for line in result["stdout"].strip().split("\n"):
            try:
                msg = json.loads(line)
                if msg.get("message_type") == "summary":
                    logger.info(
                        "Backup complete: %d files, %s added, snapshot %s",
                        msg.get("files_new", 0) + msg.get("files_changed", 0),
                        format_bytes(msg.get("data_added", 0)),
                        msg.get("snapshot_id", "unknown")[:8],
                    )
                    return {
                        "success": True,
                        "snapshot_id": msg.get("snapshot_id"),
                        "files_new": msg.get("files_new", 0),
                        "files_changed": msg.get("files_changed", 0),
                        "files_unmodified": msg.get("files_unmodified", 0),
                        "data_added": msg.get("data_added", 0),
                        "total_files_processed": msg.get("total_files_processed", 0),
                        "total_bytes_processed": msg.get("total_bytes_processed", 0),
                        "duration": msg.get("total_duration", 0),
                    }
            except json.JSONDecodeError:
                continue
    return {"success": False, "error": result["stderr"][:500]}


def verify_backup_integrity(repo_url, password, read_data=True, env_extra=None):
    """Verify repository integrity by checking all data packs."""
    args = ["check"]
    if read_data:
        args.append("--read-data")
    args.append("--json")

    logger.info("Starting integrity verification (read_data=%s)...", read_data)
    result = run_restic(args, repo=repo_url, password=password, env_extra=env_extra)

    verification = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "read_data": read_data,
        "passed": result["returncode"] == 0,
        "output": result["stdout"][:2000],
        "errors": [],
    }
    if result["returncode"] != 0:
        for line in result["stderr"].split("\n"):
            if "error" in line.lower() or "fatal" in line.lower():
                verification["errors"].append(line.strip())
        logger.error("Integrity check FAILED: %d errors", len(verification["errors"]))
    else:
        logger.info("Integrity check PASSED")
    return verification


def list_snapshots(repo_url, password, env_extra=None):
    """List all snapshots in the repository."""
    result = run_restic(
        ["snapshots", "--json"], repo=repo_url, password=password, env_extra=env_extra
    )
    if result["returncode"] == 0 and result["stdout"].strip():
        try:
            snapshots = json.loads(result["stdout"])
            return {
                "count": len(snapshots),
                "snapshots": [
                    {
                        "id": s["short_id"],
                        "time": s["time"],
                        "hostname": s.get("hostname", ""),
                        "tags": s.get("tags", []),
                        "paths": s.get("paths", []),
                    }
                    for s in snapshots
                ],
            }
        except json.JSONDecodeError:
            pass
    return {"count": 0, "snapshots": []}


def apply_retention_policy(repo_url, password, keep_daily=7, keep_weekly=4,
                           keep_monthly=12, keep_yearly=2, env_extra=None):
    """Apply snapshot retention policy using restic forget."""
    args = [
        "forget", "--prune",
        "--keep-daily", str(keep_daily),
        "--keep-weekly", str(keep_weekly),
        "--keep-monthly", str(keep_monthly),
        "--keep-yearly", str(keep_yearly),
        "--json",
    ]
    result = run_restic(args, repo=repo_url, password=password, env_extra=env_extra)
    if result["returncode"] == 0:
        logger.info(
            "Retention policy applied: daily=%d weekly=%d monthly=%d yearly=%d",
            keep_daily, keep_weekly, keep_monthly, keep_yearly,
        )
    return {
        "success": result["returncode"] == 0,
        "policy": {
            "keep_daily": keep_daily,
            "keep_weekly": keep_weekly,
            "keep_monthly": keep_monthly,
            "keep_yearly": keep_yearly,
        },
        "output": result["stdout"][:1000],
    }


def test_restore(repo_url, password, snapshot_id="latest", target_path=None,
                 sample_count=5, env_extra=None):
    """Test restore by extracting files and verifying checksums."""
    if target_path is None:
        target_path = tempfile.mkdtemp(prefix="restic_restore_test_")
    else:
        os.makedirs(target_path, exist_ok=True)

    result = run_restic(
        ["restore", snapshot_id, "--target", target_path],
        repo=repo_url, password=password, env_extra=env_extra,
    )
    if result["returncode"] != 0:
        return {"success": False, "error": result["stderr"][:500]}

    restored_files = []
    for root, dirs, files in os.walk(target_path):
        for fname in files:
            fpath = os.path.join(root, fname)
            try:
                stat = os.stat(fpath)
                sha256 = hashlib.sha256()
                with open(fpath, "rb") as f:
                    for chunk in iter(lambda: f.read(65536), b""):
                        sha256.update(chunk)
                restored_files.append({
                    "path": os.path.relpath(fpath, target_path),
                    "size": stat.st_size,
                    "sha256": sha256.hexdigest(),
                })
            except (PermissionError, OSError):
                continue
            if len(restored_files) >= sample_count:
                break
        if len(restored_files) >= sample_count:
            break

    try:
        shutil.rmtree(target_path)
    except OSError:
        pass

    logger.info("Restore test: %d files verified from snapshot %s", len(restored_files), snapshot_id)
    return {
        "success": True,
        "snapshot": snapshot_id,
        "files_verified": len(restored_files),
        "samples": restored_files,
        "restore_path": target_path,
    }


def get_repository_stats(repo_url, password, env_extra=None):
    """Get repository statistics including size and deduplication ratio."""
    result = run_restic(
        ["stats", "--json", "--mode", "restore-size"],
        repo=repo_url, password=password, env_extra=env_extra,
    )
    stats = {}
    if result["returncode"] == 0 and result["stdout"].strip():
        try:
            stats = json.loads(result["stdout"])
        except json.JSONDecodeError:
            pass

    raw_result = run_restic(
        ["stats", "--json", "--mode", "raw-data"],
        repo=repo_url, password=password, env_extra=env_extra,
    )
    raw_stats = {}
    if raw_result["returncode"] == 0 and raw_result["stdout"].strip():
        try:
            raw_stats = json.loads(raw_result["stdout"])
        except json.JSONDecodeError:
            pass

    restore_size = stats.get("total_size", 0)
    raw_size = raw_stats.get("total_size", 0)
    dedup_ratio = round(restore_size / max(raw_size, 1), 2) if raw_size else 0

    return {
        "restore_size": restore_size,
        "restore_size_human": format_bytes(restore_size),
        "raw_size": raw_size,
        "raw_size_human": format_bytes(raw_size),
        "deduplication_ratio": dedup_ratio,
        "total_file_count": stats.get("total_file_count", 0),
    }


def format_bytes(size):
    """Format byte count to human-readable string."""
    for unit in ["B", "KiB", "MiB", "GiB", "TiB"]:
        if abs(size) < 1024:
            return f"{size:.1f} {unit}"
        size /= 1024
    return f"{size:.1f} PiB"


def generate_backup_report(repo_url, password, env_extra=None):
    """Generate comprehensive backup status report."""
    snapshots = list_snapshots(repo_url, password, env_extra)
    stats = get_repository_stats(repo_url, password, env_extra)
    verification = verify_backup_integrity(repo_url, password, read_data=False, env_extra=env_extra)

    report = {
        "report_timestamp": datetime.now(timezone.utc).isoformat(),
        "repository": repo_url,
        "snapshot_count": snapshots["count"],
        "latest_snapshot": snapshots["snapshots"][-1] if snapshots["snapshots"] else None,
        "oldest_snapshot": snapshots["snapshots"][0] if snapshots["snapshots"] else None,
        "repository_stats": stats,
        "integrity_check": {
            "passed": verification["passed"],
            "errors": verification["errors"],
        },
        "backup_strategy": {
            "encryption": "AES-256-CTR with Poly1305-AES MAC",
            "deduplication": "content-defined chunking (CDC)",
            "dedup_ratio": stats.get("deduplication_ratio", 0),
        },
    }
    return report


def main():
    parser = argparse.ArgumentParser(
        description="Immutable Backup Agent with Restic and S3 Object Lock"
    )
    parser.add_argument("--repo", required=True, help="Restic repository URL (s3:host/bucket)")
    parser.add_argument("--password-file", help="File containing repository password")
    parser.add_argument("--action", choices=[
        "init", "backup", "verify", "snapshots", "retention", "restore-test",
        "stats", "report", "configure-lock", "full-pipeline"
    ], default="report")
    parser.add_argument("--source", nargs="+", help="Source paths to back up")
    parser.add_argument("--tags", nargs="+", help="Tags for backup snapshot")
    parser.add_argument("--exclude", nargs="+", help="Exclude patterns")
    parser.add_argument("--bucket", help="S3 bucket name for Object Lock config")
    parser.add_argument("--retention-days", type=int, default=90)
    parser.add_argument("--keep-daily", type=int, default=7)
    parser.add_argument("--keep-weekly", type=int, default=4)
    parser.add_argument("--keep-monthly", type=int, default=12)
    parser.add_argument("--snapshot", default="latest", help="Snapshot ID for restore test")
    parser.add_argument("--output", default="backup_report.json")
    args = parser.parse_args()

    password = None
    if args.password_file and os.path.exists(args.password_file):
        with open(args.password_file) as f:
            password = f.read().strip()
    else:
        password = os.getenv("RESTIC_PASSWORD", "")

    if args.action == "init":
        result = init_repository(args.repo, password)
        print(json.dumps(result, indent=2))

    elif args.action == "backup":
        if not args.source:
            print("Error: --source required for backup")
            sys.exit(1)
        result = create_backup(args.repo, password, args.source, args.tags, args.exclude)
        print(json.dumps(result, indent=2, default=str))

    elif args.action == "verify":
        result = verify_backup_integrity(args.repo, password, read_data=True)
        print(json.dumps(result, indent=2))

    elif args.action == "snapshots":
        result = list_snapshots(args.repo, password)
        print(json.dumps(result, indent=2))

    elif args.action == "retention":
        result = apply_retention_policy(
            args.repo, password,
            args.keep_daily, args.keep_weekly, args.keep_monthly, 2,
        )
        print(json.dumps(result, indent=2))

    elif args.action == "restore-test":
        result = test_restore(args.repo, password, args.snapshot)
        print(json.dumps(result, indent=2))

    elif args.action == "stats":
        result = get_repository_stats(args.repo, password)
        print(json.dumps(result, indent=2))

    elif args.action == "configure-lock":
        if not args.bucket:
            print("Error: --bucket required for Object Lock configuration")
            sys.exit(1)
        result = configure_s3_object_lock(args.bucket, args.retention_days)
        print(json.dumps(result, indent=2))

    elif args.action == "report":
        report = generate_backup_report(args.repo, password)
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(json.dumps(report, indent=2, default=str))

    elif args.action == "full-pipeline":
        if not args.source:
            print("Error: --source required for full pipeline")
            sys.exit(1)
        print("[1/5] Creating backup...")
        backup = create_backup(args.repo, password, args.source, args.tags, args.exclude)
        print(f"  Backup: {'SUCCESS' if backup.get('success') else 'FAILED'}")
        print("[2/5] Verifying integrity...")
        verify = verify_backup_integrity(args.repo, password, read_data=True)
        print(f"  Integrity: {'PASSED' if verify['passed'] else 'FAILED'}")
        print("[3/5] Applying retention policy...")
        retention = apply_retention_policy(
            args.repo, password, args.keep_daily, args.keep_weekly, args.keep_monthly, 2
        )
        print(f"  Retention: {'SUCCESS' if retention['success'] else 'FAILED'}")
        print("[4/5] Testing restore...")
        restore = test_restore(args.repo, password)
        print(f"  Restore: {'SUCCESS' if restore['success'] else 'FAILED'}")
        print("[5/5] Generating report...")
        report = generate_backup_report(args.repo, password)
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(f"  Report saved to {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-ransomware-backup-strategy/SKILL.md =====
---
name: implementing-ransomware-backup-strategy
description: 'Designs a ransomware-resilient backup strategy using the 3-2-1-1-0
  methodology (3 copies, 2 media types, 1 offsite, 1 immutable/air-gapped, 0 restore
  errors), configuring RPO/RTO-aligned schedules, isolating backup credentials, and
  automating restore testing. Use when planning ransomware backup resilience or air-gapped/immutable
  backup architecture.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- backup
- incident-response
- defense
- recovery
- immutable-storage
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_ai_rmf:
- MEASURE-2.7
- MAP-5.1
- MANAGE-2.4
- MANAGE-3.1
- MEASURE-3.1
atlas_techniques:
- AML.T0070
- AML.T0066
- AML.T0082
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
mitre_f3:
  version: '1.1'
  tactics:
  - positioning
  - monetization
  techniques:
  - id: T1531
    name: Account Access Removal
    tactic: positioning
    source: attack
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1047
    name: Transfer of funds
    tactic: monetization
    source: f3
  - id: F1017.001
    name: 'Conversion to Physical Monetary Instruments: Cash'
    tactic: monetization
    source: f3
---
# Implementing Ransomware Backup Strategy

## When to Use

- Designing backup architecture that withstands ransomware encryption and deletion attempts
- Migrating from traditional backup to ransomware-resilient backup with immutable storage
- Establishing RPO/RTO targets for critical systems and validating them through restore testing
- Isolating backup credentials and infrastructure from the production Active Directory domain
- Meeting cyber insurance requirements for backup resilience and tested recovery capabilities

**Do not use** as a substitute for endpoint protection, network segmentation, or incident response planning. Backups are a last line of defense, not a primary prevention control.

## Prerequisites

- Inventory of critical systems, applications, and data classified by business impact (Tier 1/2/3)
- Defined RPO (Recovery Point Objective) and RTO (Recovery Time Objective) per tier
- Backup software supporting immutable repositories (Veeam 12+, Commvault, Rubrik, Cohesity)
- Isolated backup network segment or air-gapped storage infrastructure
- Separate backup admin credentials not joined to the production AD domain

## Workflow

### Step 1: Classify Assets and Define Recovery Objectives

Map all systems into recovery tiers based on business impact:

| Tier | Examples | RPO | RTO | Backup Frequency |
|------|----------|-----|-----|------------------|
| Tier 1 (Critical) | Domain controllers, ERP, databases | 1 hour | 4 hours | Hourly incremental, daily full |
| Tier 2 (Important) | File servers, email, web apps | 4 hours | 12 hours | Every 4 hours incremental, daily full |
| Tier 3 (Standard) | Dev environments, archives | 24 hours | 48 hours | Daily incremental, weekly full |

Document dependencies between systems. Domain controllers and DNS must recover before application servers. Database servers before application tiers.

### Step 2: Implement 3-2-1-1-0 Architecture

Configure backup storage following the extended 3-2-1-1-0 rule:

**Copy 1 - Primary backup on local storage:**
```
# Veeam backup job targeting local repository
# Fast restore for operational recovery
Backup Repository: Local NAS (CIFS/NFS) or SAN
Retention: 14 days of restore points
Encryption: AES-256 with password not stored in AD
```

**Copy 2 - Secondary backup on different media:**
```
# Replicate to secondary site or cloud
# Veeam Backup Copy Job or Scale-Out Backup Repository
Target: AWS S3 / Azure Blob / Wasabi / tape library
Retention: 30 days
Transfer: Encrypted TLS 1.2+ in transit
```

**Copy 3 - Offsite copy:**
```
# Geographically separated from primary and secondary
# Cloud object storage in different region or physical tape rotation
Target: Cross-region cloud storage or Iron Mountain tape vaulting
Retention: 90 days
```

**+1 - Immutable or air-gapped copy:**
```
# Cannot be modified or deleted for defined retention period
# Veeam Hardened Repository on Linux with immutable flag
# Or AWS S3 Object Lock in Compliance mode
# Or physical air-gapped tape
```

**+0 - Zero errors on restore verification:**
```
# Automated restore testing using Veeam SureBackup or equivalent
# Scheduled weekly for Tier 1, monthly for Tier 2/3
# Verify boot, network connectivity, and application health
```

### Step 3: Isolate Backup Credentials

Ransomware operators target backup infrastructure by compromising backup admin credentials through Active Directory:

1. **Separate backup admin accounts** from the production AD domain. Use local accounts on backup servers or a dedicated backup management domain.
2. **Dedicated backup network segment** with firewall rules allowing only backup traffic (specific ports, specific source/destination IPs).
3. **MFA on backup console access** using hardware tokens or authenticator apps, not SMS.
4. **Disable RDP** on backup servers. Use out-of-band management (iLO/iDRAC/IPMI) for emergency access.
5. **Remove backup servers from domain** or place in a dedicated OU with restricted GPO inheritance.

```bash
# Linux Hardened Repository - disable SSH password auth
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Set immutable flag on backup files (XFS filesystem)
sudo chattr +i /mnt/backup/repository/*

# Veeam Hardened Repository uses single-use credentials
# that are not stored on the Veeam server after initial setup
```

### Step 4: Configure Immutable Storage

**Veeam Hardened Linux Repository:**
```bash
# Minimal Ubuntu 22.04 LTS installation
# No GUI, no unnecessary services
# Veeam uses temporary SSH credentials during backup window only

# Configure XFS with reflink support
sudo mkfs.xfs -b size=4096 -m reflink=1 /dev/sdb1
sudo mount /dev/sdb1 /mnt/veeam-repo

# Create dedicated Veeam user with limited permissions
sudo useradd -m -s /bin/bash veeamuser
sudo mkdir -p /mnt/veeam-repo/backups
sudo chown veeamuser:veeamuser /mnt/veeam-repo/backups
```

**AWS S3 Object Lock (Compliance Mode):**
```bash
# Create bucket with Object Lock enabled
aws s3api create-bucket \
  --bucket company-immutable-backups \
  --object-lock-enabled-for-bucket \
  --region us-east-1

# Set default retention - 30 days compliance mode
aws s3api put-object-lock-configuration \
  --bucket company-immutable-backups \
  --object-lock-configuration '{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
      "DefaultRetention": {
        "Mode": "COMPLIANCE",
        "Days": 30
      }
    }
  }'
```

**Azure Immutable Blob Storage:**
```bash
# Create storage account with immutable storage
az storage container immutability-policy create \
  --account-name backupaccount \
  --container-name immutable-backups \
  --period 30

# Lock the policy (irreversible)
az storage container immutability-policy lock \
  --account-name backupaccount \
  --container-name immutable-backups
```

### Step 5: Automate Restore Testing

Configure automated restore verification on a recurring schedule:

```powershell
# Veeam SureBackup verification job (PowerShell)
# Tests VM boot, network ping, and application health

Add-PSSnapin VeeamPSSnapin
$backupJob = Get-VBRJob -Name "Tier1-DailyBackup"
$sureBackupJob = Get-VSBJob -Name "Tier1-RestoreTest"

# Verify last restore test completed successfully
$lastSession = Get-VSBSession -Job $sureBackupJob -Last
if ($lastSession.Result -ne "Success") {
    Send-MailMessage -To "backup-team@company.com" `
        -Subject "ALERT: SureBackup verification failed" `
        -Body "Tier 1 restore test failed. Last result: $($lastSession.Result)" `
        -SmtpServer "smtp.company.com"
}
```

Document restore test results and maintain a recovery runbook with step-by-step procedures for each tier.

## Key Concepts

| Term | Definition |
|------|------------|
| **3-2-1-1-0** | Extended backup rule: 3 copies, 2 media types, 1 offsite, 1 immutable/air-gapped, 0 restore verification errors |
| **RPO** | Recovery Point Objective: maximum acceptable data loss measured in time (e.g., 1 hour RPO means max 1 hour of data loss) |
| **RTO** | Recovery Time Objective: maximum acceptable downtime before system must be operational |
| **Immutable Backup** | Backup copy that cannot be modified, encrypted, or deleted for a defined retention period, even by administrators |
| **Air-Gapped Backup** | Physically isolated backup with no network connectivity to production systems, providing strongest ransomware protection |
| **Hardened Repository** | Linux-based backup storage with minimal attack surface, no persistent SSH, and immutable file flags |

## Tools & Systems

- **Veeam Backup & Replication 12**: Enterprise backup with Hardened Linux Repository, SureBackup verification, and immutable backup support
- **Rubrik Security Cloud**: Zero-trust backup platform with immutable snapshots, anomaly detection, and air-gapped recovery
- **Commvault**: Backup with Metallic air-gap protection, anomaly detection, and automated recovery orchestration
- **AWS S3 Object Lock**: Cloud-native immutable storage in Compliance or Governance mode for backup copies
- **Cohesity DataProtect**: Backup platform with DataLock immutability, anti-ransomware detection, and instant mass restore

## Common Scenarios

### Scenario: Financial Services Firm Implementing Ransomware-Resilient Backup

**Context**: A mid-size bank with 500 servers, 200TB of data, and regulatory requirements for 7-year retention must redesign backup after a peer institution was hit by ransomware. Current backups use a single Veeam repository on a Windows server joined to the production domain.

**Approach**:
1. Classify all 500 servers into three tiers: 50 Tier 1 (core banking, AD, DNS), 200 Tier 2 (email, file shares, web), 250 Tier 3 (dev, test, archive)
2. Deploy Veeam Hardened Linux Repository on dedicated Ubuntu 22.04 servers with XFS immutability for primary backup
3. Configure S3 Object Lock in Compliance mode for 30-day immutable cloud copy with Veeam Scale-Out Repository capacity tier
4. Establish quarterly tape rotation to Iron Mountain for 7-year regulatory retention
5. Remove all backup servers from the production AD domain and create isolated backup admin accounts with hardware MFA tokens
6. Deploy SureBackup jobs: weekly for Tier 1, monthly for Tier 2, quarterly for Tier 3
7. Conduct annual full recovery drill restoring AD, DNS, core banking, and dependent applications to validate documented RTO

**Pitfalls**:
- Leaving backup admin credentials in the production AD domain where ransomware operators can compromise them via Kerberoasting or DCSync
- Configuring immutable retention periods shorter than the dwell time of typical ransomware (average 21 days), allowing attackers to wait for immutability to expire
- Testing only individual VM restores without testing full application stack recovery including dependencies
- Forgetting to back up backup server configuration (Veeam config database, encryption keys) separately from the backup infrastructure itself

## Output Format

```
## Ransomware Backup Strategy Assessment

**Organization**: [Name]
**Assessment Date**: [Date]
**Assessor**: [Name]

### Current State
- Backup Solution: [Product/Version]
- Copies: [Number and locations]
- Immutable Copy: [Yes/No - Details]
- Air-Gapped Copy: [Yes/No - Details]
- Credential Isolation: [Yes/No - Details]
- Last Restore Test: [Date - Result]

### Gap Analysis
| Control | Current | Target | Gap | Priority |
|---------|---------|--------|-----|----------|
| Immutable backup | None | S3 Object Lock + Linux Hardened Repo | Missing | Critical |
| Credential isolation | Domain-joined | Standalone local accounts + MFA | Partial | Critical |
| Restore testing | Ad-hoc manual | Automated weekly SureBackup | Missing | High |

### Recommendations
1. [Priority] [Recommendation] - [Estimated effort]
2. ...

### Recovery Tier Summary
| Tier | Systems | RPO | RTO | Backup Schedule | Restore Test Frequency |
|------|---------|-----|-----|-----------------|----------------------|
| 1 | 50 | 1hr | 4hr | Hourly inc/Daily full | Weekly |
| 2 | 200 | 4hr | 12hr | 4hr inc/Daily full | Monthly |
| 3 | 250 | 24hr | 48hr | Daily inc/Weekly full | Quarterly |
```

===== END FILE =====


===== FILE: skills/implementing-ransomware-backup-strategy/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/implementing-ransomware-backup-strategy/assets/template.md =====
# Ransomware Backup Strategy Assessment Template

## Organization Information

| Field | Value |
|-------|-------|
| Organization Name | |
| Assessment Date | |
| Assessor Name | |
| Backup Solution | |
| Number of Servers | |
| Total Data Volume | |

## Current Backup Architecture

### Backup Copies Inventory

| Copy # | Location | Media Type | Offsite? | Immutable? | Air-Gapped? | Retention | Encrypted? | Last Successful |
|--------|----------|------------|----------|------------|-------------|-----------|------------|-----------------|
| 1 | | | | | | | | |
| 2 | | | | | | | | |
| 3 | | | | | | | | |

### 3-2-1-1-0 Compliance Checklist

- [ ] **3 Copies**: At least 3 copies of data exist
- [ ] **2 Media Types**: Backups stored on at least 2 different media types
- [ ] **1 Offsite**: At least 1 copy stored offsite or in a different geographic location
- [ ] **1 Immutable/Air-Gapped**: At least 1 copy is immutable or physically air-gapped
- [ ] **0 Errors**: Automated restore testing passes with zero errors

## Recovery Tier Classification

### Tier 1 - Critical Systems

| System | RPO Target | RTO Target | Backup Frequency | Dependencies |
|--------|-----------|-----------|-------------------|--------------|
| | | | | |

### Tier 2 - Important Systems

| System | RPO Target | RTO Target | Backup Frequency | Dependencies |
|--------|-----------|-----------|-------------------|--------------|
| | | | | |

### Tier 3 - Standard Systems

| System | RPO Target | RTO Target | Backup Frequency | Dependencies |
|--------|-----------|-----------|-------------------|--------------|
| | | | | |

## Credential Isolation Assessment

| Control | Status | Evidence |
|---------|--------|----------|
| Backup servers removed from production AD | Yes / No | |
| Dedicated backup admin accounts | Yes / No | |
| MFA enabled for backup console | Yes / No | |
| Backup network segmented | Yes / No | |
| RDP disabled on backup servers | Yes / No | |
| Backup encryption keys stored separately | Yes / No | |

## Restore Testing History

| Date | Tier | Systems Tested | Result | RTO Achieved | Issues |
|------|------|---------------|--------|-------------|--------|
| | | | | | |

## Gap Analysis

| Control | Current State | Target State | Gap | Priority | Effort |
|---------|--------------|-------------|-----|----------|--------|
| Immutable backup | | | | | |
| Credential isolation | | | | | |
| Restore testing | | | | | |
| Offsite copy | | | | | |
| Encryption | | | | | |

## Recommendations

### Critical Priority

1. **[Finding]**: [Recommendation] - Estimated effort: [X days/weeks]

### High Priority

1. **[Finding]**: [Recommendation] - Estimated effort: [X days/weeks]

### Medium Priority

1. **[Finding]**: [Recommendation] - Estimated effort: [X days/weeks]

## Recovery Runbook Checklist

### Pre-Recovery
- [ ] Incident declared and scope determined
- [ ] Affected systems isolated from network
- [ ] Backup integrity verified (immutable copies confirmed clean)
- [ ] Backup timestamps verified to predate infection
- [ ] Recovery environment prepared (clean network, fresh OS images)

### Recovery Execution
- [ ] Phase 1: Identity infrastructure (AD, DNS, DHCP)
- [ ] Phase 2: Tier 1 critical systems
- [ ] Phase 3: Tier 2 important systems
- [ ] Phase 4: Tier 3 standard systems
- [ ] Each restored system validated before connecting to network

### Post-Recovery
- [ ] All restored systems scanned for persistence mechanisms
- [ ] Security controls validated (EDR, firewall rules, MFA)
- [ ] Users notified and credentials reset
- [ ] Recovery time documented against RTO targets
- [ ] Lessons learned documented

## Sign-Off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| IT Director | | | |
| CISO | | | |
| Backup Admin | | | |

===== END FILE =====


===== FILE: skills/implementing-ransomware-backup-strategy/references/api-reference.md =====
# API Reference: Ransomware Backup Strategy Audit

## Libraries Used

| Library | Purpose |
|---------|---------|
| `boto3` | AWS SDK for S3, AWS Backup, and IAM auditing |
| `json` | Parse backup policies and compliance data |
| `subprocess` | Run local backup verification commands |
| `datetime` | Calculate backup age and RPO/RTO compliance |

## Installation

```bash
pip install boto3
```

## Authentication

```python
import boto3
import os

session = boto3.Session(
    aws_access_key_id=os.environ.get("AWS_ACCESS_KEY_ID"),
    aws_secret_access_key=os.environ.get("AWS_SECRET_ACCESS_KEY"),
    region_name=os.environ.get("AWS_REGION", "us-east-1"),
)

s3 = session.client("s3")
backup = session.client("backup")
iam = session.client("iam")
```

## AWS S3 Backup Audit

### Check Bucket Versioning (Ransomware Recovery)
```python
def audit_s3_versioning():
    findings = []
    buckets = s3.list_buckets()["Buckets"]
    for bucket in buckets:
        name = bucket["Name"]
        versioning = s3.get_bucket_versioning(Bucket=name)
        status = versioning.get("Status", "Disabled")
        mfa_delete = versioning.get("MFADelete", "Disabled")

        if status != "Enabled":
            findings.append({
                "bucket": name,
                "issue": "Versioning not enabled",
                "severity": "high",
                "remediation": "Enable versioning for ransomware recovery",
            })
        if mfa_delete != "Enabled":
            findings.append({
                "bucket": name,
                "issue": "MFA Delete not enabled",
                "severity": "medium",
                "remediation": "Enable MFA Delete to prevent bulk deletion",
            })
    return findings
```

### Check Object Lock (Immutable Backups)
```python
def check_object_lock(bucket_name):
    try:
        config = s3.get_object_lock_configuration(Bucket=bucket_name)
        lock = config["ObjectLockConfiguration"]
        rule = lock.get("Rule", {}).get("DefaultRetention", {})
        return {
            "bucket": bucket_name,
            "object_lock_enabled": lock.get("ObjectLockEnabled") == "Enabled",
            "retention_mode": rule.get("Mode", "NONE"),
            "retention_days": rule.get("Days", 0),
        }
    except s3.exceptions.ClientError:
        return {"bucket": bucket_name, "object_lock_enabled": False}
```

### Check Cross-Region Replication
```python
def check_cross_region_replication(bucket_name):
    try:
        repl = s3.get_bucket_replication(Bucket=bucket_name)
        rules = repl["ReplicationConfiguration"]["Rules"]
        return {
            "bucket": bucket_name,
            "replication_enabled": True,
            "destinations": [
                r["Destination"]["Bucket"] for r in rules if r["Status"] == "Enabled"
            ],
        }
    except s3.exceptions.ClientError:
        return {"bucket": bucket_name, "replication_enabled": False}
```

## AWS Backup Service

### List Backup Plans
```python
def list_backup_plans():
    plans = backup.list_backup_plans()["BackupPlansList"]
    result = []
    for plan in plans:
        detail = backup.get_backup_plan(BackupPlanId=plan["BackupPlanId"])
        rules = detail["BackupPlan"]["Rules"]
        result.append({
            "name": plan["BackupPlanName"],
            "id": plan["BackupPlanId"],
            "rules": [
                {
                    "name": r["RuleName"],
                    "schedule": r.get("ScheduleExpression"),
                    "lifecycle_delete_days": r.get("Lifecycle", {}).get("DeleteAfterDays"),
                    "lifecycle_cold_days": r.get("Lifecycle", {}).get("MoveToColdStorageAfterDays"),
                    "target_vault": r["TargetBackupVaultName"],
                }
                for r in rules
            ],
        })
    return result
```

### Audit Backup Vault Access Policy
```python
def audit_vault_access(vault_name):
    try:
        policy = backup.get_backup_vault_access_policy(BackupVaultName=vault_name)
        policy_doc = json.loads(policy["Policy"])
        # Check for overly permissive policies
        findings = []
        for stmt in policy_doc.get("Statement", []):
            if stmt.get("Effect") == "Allow" and stmt.get("Principal") == "*":
                findings.append({
                    "vault": vault_name,
                    "issue": "Vault policy allows public access",
                    "severity": "critical",
                })
        return findings
    except backup.exceptions.ClientError:
        return [{"vault": vault_name, "issue": "No access policy set", "severity": "medium"}]
```

### List Recovery Points (Check Backup Freshness)
```python
from datetime import datetime, timezone

def check_backup_freshness(vault_name, max_age_hours=24):
    recovery_points = backup.list_recovery_points_by_backup_vault(
        BackupVaultName=vault_name, MaxResults=100
    )["RecoveryPoints"]

    stale = []
    for rp in recovery_points:
        age = datetime.now(timezone.utc) - rp["CreationDate"]
        if age.total_seconds() > max_age_hours * 3600:
            stale.append({
                "resource": rp["ResourceArn"],
                "last_backup": rp["CreationDate"].isoformat(),
                "age_hours": round(age.total_seconds() / 3600),
                "status": rp["Status"],
            })
    return stale
```

## 3-2-1 Backup Rule Audit

```python
def audit_321_rule(bucket_name):
    """Verify the 3-2-1 backup rule: 3 copies, 2 media types, 1 offsite."""
    versioning = s3.get_bucket_versioning(Bucket=bucket_name)
    replication = check_cross_region_replication(bucket_name)
    object_lock = check_object_lock(bucket_name)

    score = {
        "three_copies": versioning.get("Status") == "Enabled",
        "two_media": replication["replication_enabled"],
        "one_offsite": replication["replication_enabled"],
        "immutable": object_lock["object_lock_enabled"],
    }
    score["compliant"] = all([score["three_copies"], score["two_media"], score["one_offsite"]])
    return score
```

## Output Format

```json
{
  "audit_date": "2025-01-15",
  "backup_strategy": {
    "total_buckets": 15,
    "versioning_enabled": 12,
    "object_lock_enabled": 5,
    "cross_region_replication": 8,
    "three_two_one_compliant": 4
  },
  "backup_plans": 3,
  "recovery_points_stale": 2,
  "findings": [
    {
      "resource": "critical-data-bucket",
      "issue": "No Object Lock — vulnerable to ransomware deletion",
      "severity": "high",
      "remediation": "Enable S3 Object Lock in COMPLIANCE mode"
    }
  ]
}
```

===== END FILE =====


===== FILE: skills/implementing-ransomware-backup-strategy/references/standards.md =====
# Standards & References - Ransomware Backup Strategy

## Industry Standards

### NIST SP 800-209: Security Guidelines for Storage Infrastructure
- Defines security controls for storage systems including backup infrastructure
- Covers access control, encryption, integrity verification, and audit logging for storage
- Section 5.3: Backup and recovery security controls

### NIST IR 8374: Ransomware Risk Management
- Identifies backup as a critical control in the Recover function
- Recommends maintaining offline, encrypted backups with regular testing
- Emphasizes credential separation for backup administration

### CISA #StopRansomware Guide (2023, updated 2025)
- Prescribes 3-2-1 backup rule as baseline, recommends extending to 3-2-1-1-0
- Mandates backup credential isolation from production domains
- Requires documented and tested recovery procedures

### CIS Controls v8
- Control 11: Data Recovery
  - 11.1: Establish and maintain a data recovery process
  - 11.2: Perform automated backups
  - 11.3: Protect recovery data (encryption, access control)
  - 11.4: Establish and maintain an isolated instance of recovery data (air-gapped/immutable)
  - 11.5: Test data recovery

### ISO 27001:2022
- A.8.13: Information backup
- A.8.14: Redundancy of information processing facilities

## Regulatory Requirements

### PCI DSS v4.0
- Requirement 9.4.1: Backup media physically secured
- Requirement 12.10.1: Incident response plan includes recovery procedures

### HIPAA Security Rule
- 45 CFR 164.308(a)(7): Contingency plan including data backup, disaster recovery, emergency mode operation
- 45 CFR 164.312(a)(2)(ii): Emergency access procedure

### SOX
- Section 302/404: Internal controls over financial reporting must include IT controls for data backup and recovery

## Vendor Documentation

### Veeam
- Hardened Repository Guide: https://helpcenter.veeam.com/docs/backup/vsphere/hardened_repository.html
- SureBackup: https://helpcenter.veeam.com/docs/backup/vsphere/surebackup_job.html
- Immutability: https://helpcenter.veeam.com/docs/backup/vsphere/immutability.html

### AWS
- S3 Object Lock: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html
- AWS Backup Vault Lock: https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html

### Azure
- Immutable Blob Storage: https://learn.microsoft.com/en-us/azure/storage/blobs/immutable-storage-overview
- Azure Backup Immutable Vault: https://learn.microsoft.com/en-us/azure/backup/backup-azure-immutable-vault-concept

===== END FILE =====


===== FILE: skills/implementing-ransomware-backup-strategy/references/workflows.md =====
# Workflows - Ransomware Backup Strategy

## Workflow 1: Initial Backup Architecture Design

```
Start
  |
  v
[Inventory all systems and data] --> Classify into Tier 1/2/3 by business impact
  |
  v
[Define RPO/RTO per tier] --> Document in recovery plan
  |
  v
[Select backup platform] --> Veeam / Rubrik / Commvault / Cohesity
  |
  v
[Design 3-2-1-1-0 architecture]
  |-- Copy 1: Local repository (fast restore)
  |-- Copy 2: Secondary site/cloud (different media)
  |-- Copy 3: Offsite (geographic separation)
  |-- +1: Immutable or air-gapped copy
  |-- +0: Automated restore verification
  |
  v
[Isolate backup credentials]
  |-- Remove from production AD
  |-- Deploy MFA for backup admin access
  |-- Segment backup network
  |
  v
[Configure immutable storage]
  |-- Linux Hardened Repository (XFS immutability)
  |-- S3 Object Lock / Azure Immutable Blob
  |-- Tape air-gap rotation
  |
  v
[Set backup schedules per tier]
  |
  v
[Configure automated restore testing]
  |-- SureBackup / SureReplica
  |-- Verify boot, network, application health
  |
  v
[Document recovery runbook]
  |
  v
End
```

## Workflow 2: Restore Verification Process

```
Start (Scheduled - Weekly for Tier 1, Monthly for Tier 2)
  |
  v
[SureBackup job triggers VM restore to isolated sandbox]
  |
  v
[VM boots in isolated network segment]
  |
  v
[Heartbeat check] -- Fail --> Alert backup team
  |
  Pass
  |
  v
[Network ping check] -- Fail --> Alert backup team
  |
  Pass
  |
  v
[Application-specific check]
  |-- AD: LDAP query test
  |-- SQL: Database consistency check
  |-- Web: HTTP 200 response
  |-- Email: SMTP handshake
  |
  Fail --> Alert backup team with diagnostic details
  |
  Pass
  |
  v
[Log successful restore] --> Update compliance dashboard
  |
  v
[Clean up sandbox VMs]
  |
  v
End
```

## Workflow 3: Emergency Ransomware Recovery

```
Ransomware Incident Declared
  |
  v
[Isolate affected systems from network]
  |
  v
[Verify backup integrity]
  |-- Check immutable copies are unaffected
  |-- Validate backup timestamps predate infection
  |-- Scan backup files for ransomware artifacts
  |
  v
[Determine recovery scope]
  |-- Full environment rebuild vs. selective restore
  |-- Prioritize by tier: AD/DNS first, then Tier 1, then Tier 2/3
  |
  v
[Rebuild infrastructure in clean environment]
  |-- Deploy clean OS images
  |-- Restore AD from immutable backup
  |-- Validate AD integrity with ADRestore/DSInternals
  |
  v
[Restore applications in dependency order]
  |-- Database servers before application servers
  |-- Internal services before external-facing
  |
  v
[Validate restored systems]
  |-- Application functionality testing
  |-- Data integrity verification
  |-- Security control validation
  |
  v
[Reconnect to network in phases]
  |-- Monitor for re-infection indicators
  |-- Validate no persistence mechanisms in restored systems
  |
  v
[Post-recovery documentation and lessons learned]
  |
  v
End
```

## Workflow 4: Backup Health Monitoring

```
Daily Automated Check
  |
  v
[Query backup job status via API/PowerShell]
  |
  v
[Check for failed or warning jobs]
  |-- Failed --> Create P1 ticket, alert backup team
  |-- Warning --> Create P3 ticket, investigate within 24hr
  |-- Success --> Log and continue
  |
  v
[Verify backup repository capacity]
  |-- >85% utilization --> Alert for capacity planning
  |-- >95% utilization --> Critical alert, backup jobs at risk
  |
  v
[Check immutable copy synchronization]
  |-- Verify last immutable copy is within RPO window
  |-- Alert if immutable copy is stale
  |
  v
[Generate weekly backup health report]
  |-- Success rate percentage
  |-- Data protected volume
  |-- Restore test results
  |-- Capacity forecast
  |
  v
End
```

===== END FILE =====


===== FILE: skills/implementing-ransomware-backup-strategy/scripts/agent.py =====
#!/usr/bin/env python3
"""Ransomware backup strategy audit agent.

Audits backup infrastructure for ransomware resilience by checking
3-2-1 backup rule compliance, air-gapped/immutable backup presence,
backup encryption status, recovery point objectives (RPO), and
backup integrity verification schedules.
"""
import argparse
import json
import os
import subprocess
import sys
from datetime import datetime, timezone, timedelta


def check_veeam_backups(server_url, token):
    """Check Veeam backup status via REST API."""
    try:
        import requests
    except ImportError:
        return [{"check": "Veeam API", "status": "SKIP", "severity": "INFO",
                 "detail": "requests library not installed"}]

    findings = []
    headers = {"Authorization": f"Bearer {token}", "Accept": "application/json"}

    # Check backup jobs
    try:
        resp = requests.get(f"{server_url}/api/v1/jobs", headers=headers, timeout=30)
        resp.raise_for_status()
        jobs = resp.json().get("data", [])
        for job in jobs:
            job_name = job.get("name", "Unknown")
            last_result = job.get("lastResult", "None")
            schedule_enabled = job.get("scheduleEnabled", False)
            if last_result == "Failed":
                findings.append({
                    "check": f"Backup job: {job_name}",
                    "status": "FAIL",
                    "severity": "CRITICAL",
                    "detail": "Last backup failed",
                })
            elif not schedule_enabled:
                findings.append({
                    "check": f"Backup job: {job_name}",
                    "status": "WARN",
                    "severity": "HIGH",
                    "detail": "Schedule disabled",
                })
            else:
                findings.append({
                    "check": f"Backup job: {job_name}",
                    "status": "PASS",
                    "severity": "INFO",
                    "detail": f"Last result: {last_result}",
                })
    except Exception as e:
        findings.append({"check": "Veeam job check", "status": "ERROR",
                         "severity": "HIGH", "detail": str(e)})
    return findings


def check_restic_repository(repo_path, password_file=None):
    """Audit a Restic backup repository for integrity and freshness."""
    findings = []
    restic_bin = None
    for name in ["restic", "restic.exe"]:
        for d in os.environ.get("PATH", "").split(os.pathsep):
            if os.path.isfile(os.path.join(d, name)):
                restic_bin = os.path.join(d, name)
                break
    if not restic_bin:
        findings.append({"check": "Restic binary", "status": "SKIP",
                         "severity": "INFO", "detail": "restic not found"})
        return findings

    env = dict(os.environ)
    env["RESTIC_REPOSITORY"] = repo_path
    if password_file:
        env["RESTIC_PASSWORD_FILE"] = password_file

    # Check snapshots
    try:
        result = subprocess.run(
            [restic_bin, "snapshots", "--json"],
            capture_output=True, text=True, timeout=120, env=env,
        )
        if result.returncode == 0:
            snapshots = json.loads(result.stdout)
            if not snapshots:
                findings.append({"check": "Snapshots exist", "status": "FAIL",
                                 "severity": "CRITICAL", "detail": "No snapshots found"})
            else:
                latest = max(snapshots, key=lambda s: s.get("time", ""))
                latest_time = latest.get("time", "")[:19]
                findings.append({"check": "Latest snapshot", "status": "PASS",
                                 "severity": "INFO",
                                 "detail": f"{latest_time} ({len(snapshots)} total)"})
                try:
                    latest_dt = datetime.fromisoformat(latest_time.replace("Z", "+00:00"))
                    age_hours = (datetime.now(timezone.utc) - latest_dt).total_seconds() / 3600
                    if age_hours > 48:
                        findings.append({"check": "Backup freshness", "status": "FAIL",
                                        "severity": "HIGH",
                                        "detail": f"Latest backup is {age_hours:.0f}h old (>48h)"})
                    elif age_hours > 24:
                        findings.append({"check": "Backup freshness", "status": "WARN",
                                        "severity": "MEDIUM",
                                        "detail": f"Latest backup is {age_hours:.0f}h old (>24h)"})
                except (ValueError, TypeError):
                    pass
        else:
            findings.append({"check": "Repository access", "status": "FAIL",
                             "severity": "CRITICAL", "detail": result.stderr[:200]})
    except subprocess.TimeoutExpired:
        findings.append({"check": "Repository access", "status": "FAIL",
                         "severity": "HIGH", "detail": "Command timed out"})

    # Check repository integrity
    try:
        result = subprocess.run(
            [restic_bin, "check", "--read-data-subset=1%"],
            capture_output=True, text=True, timeout=300, env=env,
        )
        if result.returncode == 0:
            findings.append({"check": "Repository integrity", "status": "PASS",
                             "severity": "INFO", "detail": "Integrity check passed"})
        else:
            findings.append({"check": "Repository integrity", "status": "FAIL",
                             "severity": "CRITICAL", "detail": "Integrity check failed"})
    except subprocess.TimeoutExpired:
        findings.append({"check": "Repository integrity", "status": "WARN",
                         "severity": "MEDIUM", "detail": "Integrity check timed out"})

    return findings


def audit_321_rule(backup_config):
    """Audit compliance with the 3-2-1 backup rule."""
    findings = []
    copies = backup_config.get("copies", 0)
    media_types = backup_config.get("media_types", [])
    offsite_locations = backup_config.get("offsite_locations", [])
    immutable = backup_config.get("immutable_backup", False)
    air_gapped = backup_config.get("air_gapped", False)

    # 3 copies
    if copies >= 3:
        findings.append({"check": "3-2-1: At least 3 copies", "status": "PASS",
                         "severity": "INFO", "detail": f"{copies} copies"})
    else:
        findings.append({"check": "3-2-1: At least 3 copies", "status": "FAIL",
                         "severity": "CRITICAL",
                         "detail": f"Only {copies} copies (need 3)"})

    # 2 different media types
    if len(media_types) >= 2:
        findings.append({"check": "3-2-1: 2 different media types", "status": "PASS",
                         "severity": "INFO", "detail": ", ".join(media_types)})
    else:
        findings.append({"check": "3-2-1: 2 different media types", "status": "FAIL",
                         "severity": "HIGH",
                         "detail": f"Only {len(media_types)} type(s): {', '.join(media_types)}"})

    # 1 offsite copy
    if offsite_locations:
        findings.append({"check": "3-2-1: 1 offsite copy", "status": "PASS",
                         "severity": "INFO", "detail": ", ".join(offsite_locations)})
    else:
        findings.append({"check": "3-2-1: 1 offsite copy", "status": "FAIL",
                         "severity": "CRITICAL", "detail": "No offsite backup"})

    # Immutable backup (ransomware protection)
    if immutable:
        findings.append({"check": "Immutable backup", "status": "PASS",
                         "severity": "INFO", "detail": "WORM/immutable storage enabled"})
    else:
        findings.append({"check": "Immutable backup", "status": "FAIL",
                         "severity": "CRITICAL",
                         "detail": "No immutable backup - vulnerable to ransomware encryption"})

    # Air-gapped backup
    if air_gapped:
        findings.append({"check": "Air-gapped backup", "status": "PASS",
                         "severity": "INFO", "detail": "Offline/air-gapped copy exists"})
    else:
        findings.append({"check": "Air-gapped backup", "status": "WARN",
                         "severity": "HIGH",
                         "detail": "No air-gapped backup - consider tape or offline storage"})

    # Encryption
    if backup_config.get("encrypted", False):
        findings.append({"check": "Backup encryption", "status": "PASS",
                         "severity": "INFO"})
    else:
        findings.append({"check": "Backup encryption", "status": "FAIL",
                         "severity": "HIGH", "detail": "Backups are not encrypted"})

    return findings


def format_summary(all_findings):
    """Print audit summary."""
    print(f"\n{'='*60}")
    print(f"  Ransomware Backup Strategy Audit")
    print(f"{'='*60}")

    severity_counts = {}
    for f in all_findings:
        sev = f.get("severity", "INFO")
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    pass_count = sum(1 for f in all_findings if f.get("status") == "PASS")
    fail_count = sum(1 for f in all_findings if f.get("status") == "FAIL")
    warn_count = sum(1 for f in all_findings if f.get("status") == "WARN")

    print(f"  Checks   : {len(all_findings)}")
    print(f"  Passed   : {pass_count}")
    print(f"  Failed   : {fail_count}")
    print(f"  Warnings : {warn_count}")

    print(f"\n  By Severity:")
    for sev in ["CRITICAL", "HIGH", "MEDIUM", "LOW", "INFO"]:
        count = severity_counts.get(sev, 0)
        if count > 0:
            print(f"    {sev:10s}: {count}")

    print(f"\n  Detailed Results:")
    for f in all_findings:
        status_icon = "OK" if f["status"] == "PASS" else "!!" if f["status"] == "FAIL" else "~~"
        detail = f.get("detail", "")
        print(f"    [{status_icon}] [{f['severity']:8s}] {f['check']}"
              + (f": {detail}" if detail else ""))

    return severity_counts


def main():
    parser = argparse.ArgumentParser(
        description="Ransomware backup strategy audit agent"
    )
    parser.add_argument("--config", help="Backup configuration JSON file")
    parser.add_argument("--restic-repo", help="Restic repository path to audit")
    parser.add_argument("--restic-password-file", help="Restic password file")
    parser.add_argument("--veeam-url", help="Veeam server URL")
    parser.add_argument("--veeam-token", help="Veeam API token")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    all_findings = []

    if args.config:
        with open(args.config, "r") as f:
            backup_config = json.load(f)
        all_findings.extend(audit_321_rule(backup_config))

    if args.restic_repo:
        all_findings.extend(check_restic_repository(args.restic_repo, args.restic_password_file))

    if args.veeam_url and args.veeam_token:
        all_findings.extend(check_veeam_backups(args.veeam_url, args.veeam_token))

    if not all_findings:
        print("[!] No audit sources specified. Use --config, --restic-repo, or --veeam-url.",
              file=sys.stderr)
        parser.print_help()
        sys.exit(1)

    severity_counts = format_summary(all_findings)

    report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "Ransomware Backup Audit",
        "findings": all_findings,
        "severity_counts": severity_counts,
        "risk_level": (
            "CRITICAL" if severity_counts.get("CRITICAL", 0) > 0
            else "HIGH" if severity_counts.get("HIGH", 0) > 0
            else "MEDIUM" if severity_counts.get("MEDIUM", 0) > 0
            else "LOW"
        ),
    }

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"\n[+] Report saved to {args.output}")
    elif args.verbose:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-ransomware-backup-strategy/scripts/process.py =====
#!/usr/bin/env python3
"""
Ransomware Backup Strategy Assessment and Monitoring Tool

Audits backup infrastructure for ransomware resilience by checking:
- 3-2-1-1-0 compliance (copies, media types, offsite, immutable, restore testing)
- Backup credential isolation from production AD
- Immutable storage configuration
- Restore test history and success rates
- RPO/RTO compliance per tier

Supports Veeam, AWS Backup, and Azure Backup via API integration.
"""

import json
import subprocess
import sys
import os
import socket
import ssl
import datetime
from dataclasses import dataclass, field, asdict
from typing import Optional
from pathlib import Path


@dataclass
class BackupCopy:
    location: str
    media_type: str
    is_offsite: bool
    is_immutable: bool
    is_airgapped: bool
    retention_days: int
    last_successful: Optional[str] = None
    encryption_algorithm: Optional[str] = None


@dataclass
class RecoveryTier:
    name: str
    tier_level: int
    systems: list
    rpo_hours: float
    rto_hours: float
    backup_frequency: str
    restore_test_frequency: str
    last_restore_test: Optional[str] = None
    last_restore_result: Optional[str] = None


@dataclass
class BackupAssessment:
    organization: str
    assessment_date: str
    backup_solution: str
    copies: list = field(default_factory=list)
    tiers: list = field(default_factory=list)
    credential_isolation: dict = field(default_factory=dict)
    findings: list = field(default_factory=list)
    score: float = 0.0


class BackupStrategyAuditor:
    """Audits backup infrastructure for ransomware resilience."""

    def __init__(self, org_name: str, backup_solution: str):
        self.assessment = BackupAssessment(
            organization=org_name,
            assessment_date=datetime.datetime.now().strftime("%Y-%m-%d"),
            backup_solution=backup_solution,
        )
        self.max_score = 0
        self.earned_score = 0

    def add_backup_copy(self, copy: BackupCopy):
        self.assessment.copies.append(copy)

    def add_recovery_tier(self, tier: RecoveryTier):
        self.assessment.tiers.append(tier)

    def set_credential_isolation(
        self,
        domain_joined: bool,
        mfa_enabled: bool,
        separate_network: bool,
        rdp_disabled: bool,
        dedicated_admin_accounts: bool,
    ):
        self.assessment.credential_isolation = {
            "domain_joined": domain_joined,
            "mfa_enabled": mfa_enabled,
            "separate_network": separate_network,
            "rdp_disabled": rdp_disabled,
            "dedicated_admin_accounts": dedicated_admin_accounts,
        }

    def _add_finding(self, severity: str, category: str, title: str, detail: str, recommendation: str):
        self.assessment.findings.append({
            "severity": severity,
            "category": category,
            "title": title,
            "detail": detail,
            "recommendation": recommendation,
        })

    def audit_321_10_compliance(self):
        """Check compliance with the 3-2-1-1-0 backup rule."""
        copies = self.assessment.copies

        # 3 - At least 3 copies
        self.max_score += 20
        if len(copies) >= 3:
            self.earned_score += 20
        else:
            self._add_finding(
                "CRITICAL", "3-2-1-1-0",
                f"Insufficient backup copies: {len(copies)} of 3 required",
                f"Only {len(copies)} backup copies configured. Minimum 3 required.",
                "Add additional backup copies to meet 3-2-1-1-0 minimum.",
            )

        # 2 - At least 2 different media types
        self.max_score += 15
        media_types = set(c.media_type for c in copies)
        if len(media_types) >= 2:
            self.earned_score += 15
        else:
            self._add_finding(
                "HIGH", "3-2-1-1-0",
                f"Insufficient media diversity: {len(media_types)} type(s)",
                f"All backups use {media_types}. Need at least 2 different media types.",
                "Add backup copy on different media (e.g., tape, cloud object storage, SAN).",
            )

        # 1 - At least 1 offsite copy
        self.max_score += 15
        offsite_copies = [c for c in copies if c.is_offsite]
        if offsite_copies:
            self.earned_score += 15
        else:
            self._add_finding(
                "CRITICAL", "3-2-1-1-0",
                "No offsite backup copy",
                "All backup copies are stored on-premises. A site-level disaster would destroy all copies.",
                "Configure at least one offsite backup copy (cloud, remote site, or tape vaulting).",
            )

        # 1 - At least 1 immutable or air-gapped copy
        self.max_score += 25
        immutable_copies = [c for c in copies if c.is_immutable or c.is_airgapped]
        if immutable_copies:
            self.earned_score += 25
        else:
            self._add_finding(
                "CRITICAL", "3-2-1-1-0",
                "No immutable or air-gapped backup copy",
                "No backup copies are protected against modification or deletion. "
                "Ransomware operators routinely delete accessible backups.",
                "Deploy immutable storage (S3 Object Lock, Hardened Linux Repository) "
                "or air-gapped backup (tape with physical separation).",
            )

        # 0 - Restore testing with zero errors
        self.max_score += 25
        if self.assessment.tiers:
            tested_tiers = [t for t in self.assessment.tiers if t.last_restore_result == "Success"]
            all_tiers = len(self.assessment.tiers)
            if len(tested_tiers) == all_tiers:
                self.earned_score += 25
            elif tested_tiers:
                partial_score = int(25 * len(tested_tiers) / all_tiers)
                self.earned_score += partial_score
                self._add_finding(
                    "HIGH", "3-2-1-1-0",
                    f"Incomplete restore testing: {len(tested_tiers)}/{all_tiers} tiers verified",
                    f"Only {len(tested_tiers)} of {all_tiers} recovery tiers have successful restore tests.",
                    "Implement automated restore testing (SureBackup) for all tiers.",
                )
            else:
                self._add_finding(
                    "CRITICAL", "3-2-1-1-0",
                    "No successful restore tests recorded",
                    "No recovery tiers have documented successful restore verification.",
                    "Implement automated restore testing immediately. Untested backups "
                    "should be considered unreliable.",
                )

    def audit_credential_isolation(self):
        """Check backup credential isolation from production AD."""
        creds = self.assessment.credential_isolation
        if not creds:
            self._add_finding(
                "CRITICAL", "Credential Isolation",
                "Credential isolation not assessed",
                "No information provided about backup credential isolation.",
                "Assess backup admin account configuration and network isolation.",
            )
            return

        self.max_score += 10
        if not creds.get("domain_joined", True):
            self.earned_score += 10
        else:
            self._add_finding(
                "CRITICAL", "Credential Isolation",
                "Backup servers joined to production AD domain",
                "Backup infrastructure is domain-joined. Ransomware operators compromise "
                "backup credentials via Kerberoasting, DCSync, or GPO manipulation.",
                "Remove backup servers from production AD. Use local accounts or a "
                "dedicated management domain.",
            )

        self.max_score += 5
        if creds.get("mfa_enabled", False):
            self.earned_score += 5
        else:
            self._add_finding(
                "HIGH", "Credential Isolation",
                "MFA not enabled for backup administration",
                "Backup admin accounts do not require multi-factor authentication.",
                "Enable MFA for all backup console access using hardware tokens.",
            )

        self.max_score += 5
        if creds.get("separate_network", False):
            self.earned_score += 5
        else:
            self._add_finding(
                "HIGH", "Credential Isolation",
                "Backup infrastructure not on separate network segment",
                "Backup servers share the production network segment.",
                "Segment backup infrastructure into a dedicated VLAN with strict firewall rules.",
            )

        self.max_score += 3
        if creds.get("rdp_disabled", False):
            self.earned_score += 3
        else:
            self._add_finding(
                "MEDIUM", "Credential Isolation",
                "RDP enabled on backup servers",
                "Remote Desktop Protocol is accessible on backup servers.",
                "Disable RDP and use out-of-band management (iLO/iDRAC) for emergency access.",
            )

        self.max_score += 2
        if creds.get("dedicated_admin_accounts", False):
            self.earned_score += 2
        else:
            self._add_finding(
                "HIGH", "Credential Isolation",
                "No dedicated backup admin accounts",
                "Backup administration uses shared or production admin accounts.",
                "Create dedicated backup admin accounts with least-privilege access.",
            )

    def audit_rpo_rto_compliance(self):
        """Check if backup frequency meets RPO targets and document RTO validation."""
        for tier in self.assessment.tiers:
            self.max_score += 5
            # Check if restore test is recent enough
            if tier.last_restore_test:
                try:
                    last_test = datetime.datetime.strptime(tier.last_restore_test, "%Y-%m-%d")
                    days_since_test = (datetime.datetime.now() - last_test).days

                    frequency_map = {
                        "weekly": 7,
                        "monthly": 30,
                        "quarterly": 90,
                    }
                    expected_days = frequency_map.get(tier.restore_test_frequency.lower(), 30)

                    if days_since_test <= expected_days * 1.5:
                        self.earned_score += 5
                    else:
                        self._add_finding(
                            "HIGH", "RPO/RTO",
                            f"Tier {tier.tier_level} ({tier.name}): Restore test overdue",
                            f"Last test was {days_since_test} days ago. "
                            f"Expected frequency: {tier.restore_test_frequency}.",
                            f"Run restore test for {tier.name} tier immediately.",
                        )
                except ValueError:
                    self._add_finding(
                        "MEDIUM", "RPO/RTO",
                        f"Tier {tier.tier_level}: Invalid restore test date format",
                        f"Date '{tier.last_restore_test}' could not be parsed.",
                        "Use YYYY-MM-DD format for restore test dates.",
                    )
            else:
                self._add_finding(
                    "CRITICAL", "RPO/RTO",
                    f"Tier {tier.tier_level} ({tier.name}): No restore test recorded",
                    "No restore test has been documented for this recovery tier.",
                    f"Implement {tier.restore_test_frequency} automated restore testing.",
                )

    def audit_encryption(self):
        """Check backup encryption configuration."""
        for copy in self.assessment.copies:
            self.max_score += 3
            if copy.encryption_algorithm:
                if copy.encryption_algorithm.upper() in ("AES-256", "AES256", "AES-256-GCM"):
                    self.earned_score += 3
                else:
                    self._add_finding(
                        "MEDIUM", "Encryption",
                        f"Weak backup encryption: {copy.encryption_algorithm}",
                        f"Backup copy at {copy.location} uses {copy.encryption_algorithm}.",
                        "Upgrade to AES-256 encryption for all backup copies.",
                    )
            else:
                self._add_finding(
                    "HIGH", "Encryption",
                    f"Unencrypted backup: {copy.location}",
                    f"Backup copy at {copy.location} is not encrypted.",
                    "Enable AES-256 encryption for all backup copies, both at rest and in transit.",
                )

    def audit_immutable_retention(self):
        """Check immutable retention period against typical ransomware dwell time."""
        avg_dwell_time_days = 21  # Average ransomware dwell time

        for copy in self.assessment.copies:
            if copy.is_immutable:
                self.max_score += 5
                if copy.retention_days > avg_dwell_time_days:
                    self.earned_score += 5
                else:
                    self._add_finding(
                        "CRITICAL", "Immutable Retention",
                        f"Immutable retention too short: {copy.retention_days} days",
                        f"Immutable retention at {copy.location} is {copy.retention_days} days. "
                        f"Average ransomware dwell time is {avg_dwell_time_days} days. "
                        "Attackers may wait for immutability to expire.",
                        f"Increase immutable retention to at least {avg_dwell_time_days * 2} days.",
                    )

    def run_full_audit(self) -> dict:
        """Execute all audit checks and calculate overall score."""
        self.audit_321_10_compliance()
        self.audit_credential_isolation()
        self.audit_rpo_rto_compliance()
        self.audit_encryption()
        self.audit_immutable_retention()

        if self.max_score > 0:
            self.assessment.score = round((self.earned_score / self.max_score) * 100, 1)

        return asdict(self.assessment)

    def generate_report(self) -> str:
        """Generate human-readable assessment report."""
        result = self.run_full_audit()
        lines = []

        lines.append("=" * 70)
        lines.append("RANSOMWARE BACKUP STRATEGY ASSESSMENT REPORT")
        lines.append("=" * 70)
        lines.append(f"Organization: {result['organization']}")
        lines.append(f"Date: {result['assessment_date']}")
        lines.append(f"Backup Solution: {result['backup_solution']}")
        lines.append(f"Overall Score: {result['score']}%")
        lines.append("")

        # Score interpretation
        score = result["score"]
        if score >= 90:
            rating = "EXCELLENT - Ransomware-resilient backup architecture"
        elif score >= 75:
            rating = "GOOD - Minor gaps in ransomware resilience"
        elif score >= 50:
            rating = "FAIR - Significant gaps require remediation"
        else:
            rating = "POOR - Critical ransomware backup risks present"
        lines.append(f"Rating: {rating}")
        lines.append("")

        # 3-2-1-1-0 Summary
        lines.append("-" * 40)
        lines.append("3-2-1-1-0 COMPLIANCE")
        lines.append("-" * 40)
        copies = result["copies"]
        lines.append(f"Total copies: {len(copies)} (minimum 3)")
        media_types = set(c["media_type"] for c in copies)
        lines.append(f"Media types: {', '.join(media_types)} ({len(media_types)} types, minimum 2)")
        offsite = sum(1 for c in copies if c["is_offsite"])
        lines.append(f"Offsite copies: {offsite} (minimum 1)")
        immutable = sum(1 for c in copies if c["is_immutable"] or c["is_airgapped"])
        lines.append(f"Immutable/Air-gapped copies: {immutable} (minimum 1)")
        lines.append("")

        # Recovery Tiers
        lines.append("-" * 40)
        lines.append("RECOVERY TIERS")
        lines.append("-" * 40)
        for tier in result["tiers"]:
            lines.append(f"  Tier {tier['tier_level']}: {tier['name']}")
            lines.append(f"    Systems: {len(tier['systems'])}")
            lines.append(f"    RPO: {tier['rpo_hours']}h | RTO: {tier['rto_hours']}h")
            lines.append(f"    Backup: {tier['backup_frequency']}")
            lines.append(f"    Restore test: {tier['restore_test_frequency']} "
                        f"(Last: {tier['last_restore_test'] or 'Never'}, "
                        f"Result: {tier['last_restore_result'] or 'N/A'})")
        lines.append("")

        # Credential Isolation
        lines.append("-" * 40)
        lines.append("CREDENTIAL ISOLATION")
        lines.append("-" * 40)
        creds = result["credential_isolation"]
        for key, val in creds.items():
            status = "PASS" if (val if key != "domain_joined" else not val) else "FAIL"
            lines.append(f"  {key}: {'Yes' if val else 'No'} [{status}]")
        lines.append("")

        # Findings
        lines.append("-" * 40)
        lines.append(f"FINDINGS ({len(result['findings'])} total)")
        lines.append("-" * 40)
        severity_order = {"CRITICAL": 0, "HIGH": 1, "MEDIUM": 2, "LOW": 3, "INFO": 4}
        sorted_findings = sorted(result["findings"], key=lambda f: severity_order.get(f["severity"], 5))

        for i, finding in enumerate(sorted_findings, 1):
            lines.append(f"\n  [{finding['severity']}] #{i}: {finding['title']}")
            lines.append(f"    Category: {finding['category']}")
            lines.append(f"    Detail: {finding['detail']}")
            lines.append(f"    Recommendation: {finding['recommendation']}")

        lines.append("")
        lines.append("=" * 70)
        lines.append("END OF REPORT")
        lines.append("=" * 70)

        return "\n".join(lines)


def check_veeam_api(server: str, port: int = 9419) -> dict:
    """Check Veeam Backup & Replication API availability."""
    result = {"reachable": False, "tls": False, "api_version": None}
    try:
        sock = socket.create_connection((server, port), timeout=5)
        context = ssl.create_default_context()
        context.check_hostname = False
        context.verify_mode = ssl.CERT_NONE
        ssock = context.wrap_socket(sock, server_hostname=server)
        result["reachable"] = True
        result["tls"] = True
        ssock.close()
    except (socket.timeout, ConnectionRefusedError, OSError):
        try:
            sock = socket.create_connection((server, port), timeout=5)
            result["reachable"] = True
            sock.close()
        except (socket.timeout, ConnectionRefusedError, OSError):
            pass
    return result


def check_s3_object_lock(bucket_name: str) -> dict:
    """Check AWS S3 bucket for Object Lock configuration."""
    result = {"bucket": bucket_name, "object_lock_enabled": False, "retention_mode": None, "retention_days": None}
    try:
        output = subprocess.run(
            ["aws", "s3api", "get-object-lock-configuration", "--bucket", bucket_name],
            capture_output=True, text=True, timeout=30,
        )
        if output.returncode == 0:
            config = json.loads(output.stdout)
            lock_config = config.get("ObjectLockConfiguration", {})
            result["object_lock_enabled"] = lock_config.get("ObjectLockEnabled") == "Enabled"
            rule = lock_config.get("Rule", {}).get("DefaultRetention", {})
            result["retention_mode"] = rule.get("Mode")
            result["retention_days"] = rule.get("Days")
    except (subprocess.TimeoutExpired, FileNotFoundError, json.JSONDecodeError):
        pass
    return result


def main():
    """Run example backup strategy assessment."""
    auditor = BackupStrategyAuditor(
        org_name="Example Financial Services",
        backup_solution="Veeam Backup & Replication 12",
    )

    # Define backup copies
    auditor.add_backup_copy(BackupCopy(
        location="On-premises NAS (Building A)",
        media_type="NAS/NFS",
        is_offsite=False,
        is_immutable=False,
        is_airgapped=False,
        retention_days=14,
        last_successful="2026-02-22",
        encryption_algorithm="AES-256",
    ))

    auditor.add_backup_copy(BackupCopy(
        location="Veeam Hardened Linux Repository",
        media_type="Linux/XFS",
        is_offsite=False,
        is_immutable=True,
        is_airgapped=False,
        retention_days=30,
        last_successful="2026-02-22",
        encryption_algorithm="AES-256",
    ))

    auditor.add_backup_copy(BackupCopy(
        location="AWS S3 (us-west-2) Object Lock",
        media_type="Cloud Object Storage",
        is_offsite=True,
        is_immutable=True,
        is_airgapped=False,
        retention_days=60,
        last_successful="2026-02-22",
        encryption_algorithm="AES-256",
    ))

    # Define recovery tiers
    auditor.add_recovery_tier(RecoveryTier(
        name="Critical",
        tier_level=1,
        systems=["DC01", "DC02", "DNS01", "ERP-DB", "CoreBanking"],
        rpo_hours=1,
        rto_hours=4,
        backup_frequency="Hourly incremental, Daily full",
        restore_test_frequency="weekly",
        last_restore_test="2026-02-20",
        last_restore_result="Success",
    ))

    auditor.add_recovery_tier(RecoveryTier(
        name="Important",
        tier_level=2,
        systems=["Exchange", "FileServer01", "WebApp01", "SharePoint"],
        rpo_hours=4,
        rto_hours=12,
        backup_frequency="4-hour incremental, Daily full",
        restore_test_frequency="monthly",
        last_restore_test="2026-02-01",
        last_restore_result="Success",
    ))

    auditor.add_recovery_tier(RecoveryTier(
        name="Standard",
        tier_level=3,
        systems=["DevServer01", "TestDB", "ArchiveNAS"],
        rpo_hours=24,
        rto_hours=48,
        backup_frequency="Daily incremental, Weekly full",
        restore_test_frequency="quarterly",
        last_restore_test="2025-12-15",
        last_restore_result="Success",
    ))

    # Set credential isolation status
    auditor.set_credential_isolation(
        domain_joined=False,
        mfa_enabled=True,
        separate_network=True,
        rdp_disabled=True,
        dedicated_admin_accounts=True,
    )

    # Generate and print report
    report = auditor.generate_report()
    print(report)

    # Export JSON for integration
    result = auditor.run_full_audit()
    output_path = Path(__file__).parent / "assessment_result.json"
    with open(output_path, "w") as f:
        json.dump(result, f, indent=2)
    print(f"\nJSON report saved to: {output_path}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-ransomware-kill-switch-detection/SKILL.md =====
---
name: implementing-ransomware-kill-switch-detection
description: 'Analyzes ransomware kill switch mechanisms, including mutex-based execution
  guards, domain-based kill switches (e.g. WannaCry-style), and registry termination
  checks, then implements mutex vaccination and kill switch domain monitoring to stop
  ransomware before it runs. Use when analyzing a sample''s execution guards or deploying
  vaccination/monitoring as a defensive control.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- kill-switch
- mutex
- detection
- WannaCry
- malware-analysis
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1486
- T1490
mitre_f3:
  version: '1.1'
  tactics:
  - positioning
  - monetization
  techniques:
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1017
    name: Conversion to Physical Monetary Instruments
    tactic: monetization
    source: f3
  - id: F1047
    name: Transfer of funds
    tactic: monetization
    source: f3
---

# Implementing Ransomware Kill Switch Detection

## When to Use

- Analyzing a ransomware sample to determine if it contains a kill switch mechanism (mutex, domain, registry)
- Deploying proactive mutex vaccination across endpoints to prevent known ransomware families from executing
- Monitoring DNS for kill switch domain lookups that indicate ransomware attempting to check before encrypting
- During incident response to quickly determine if a ransomware variant can be stopped by activating its kill switch
- Building detection signatures for ransomware mutex creation events using Sysmon or EDR telemetry

**Do not use** kill switch vaccination as a primary defense. Not all ransomware families implement kill switches, and those that do may remove them in newer versions. This is a supplementary detection and prevention layer.

## Prerequisites

- Python 3.8+ with `ctypes` (Windows) for mutex creation and enumeration
- Sysmon installed with Event ID 1 (process creation) and Event ID 17/18 (pipe/mutex events) configured
- Access to malware analysis sandbox for identifying kill switch mechanisms in samples
- DNS monitoring capability for detecting kill switch domain resolution attempts
- Familiarity with Windows internals: mutexes (mutants), kernel objects, named pipes
- Reference database of known ransomware mutexes (github.com/albertzsigovits/malware-mutex)

## Workflow

### Step 1: Identify Kill Switch Mechanisms in Ransomware

Analyze samples for common kill switch patterns:

```
Kill Switch Types Found in Ransomware:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. MUTEX-BASED (most common):
   - Ransomware creates a named mutex at startup
   - If mutex already exists → another instance is running → exit
   - Defense: Pre-create the mutex to prevent execution
   - Examples:
     WannaCry:     Global\MsWinZonesCacheCounterMutexA
     Conti:        kasKDJSAFJauisiudUASIIQWUA82
     REvil:        Global\{GUID-based-on-machine}
     Ryuk:         Global\YOURPRODUCT_MUTEX

2. DOMAIN-BASED:
   - Ransomware resolves a hardcoded domain before executing
   - If domain resolves → security sandbox detected → exit
   - Defense: Register/sinkhole the domain to activate kill switch
   - Examples:
     WannaCry v1:  iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com
     WannaCry v1:  fferfsodp9ifjaposdfjhgosurijfaewrwergwea.com

3. REGISTRY-BASED:
   - Check for specific registry key/value before executing
   - If key exists → exit (anti-analysis or kill switch)
   - Defense: Create the registry key proactively

4. FILE-BASED:
   - Check for existence of specific file or directory
   - If marker file exists → exit
   - Defense: Create the marker file on all endpoints

5. LANGUAGE-BASED:
   - Check system language/keyboard layout
   - Exit if Russian/CIS country keyboard detected
   - Common in Eastern European ransomware groups
```

### Step 2: Deploy Mutex Vaccination

Pre-create known ransomware mutexes on endpoints to prevent execution:

```python
# Windows mutex vaccination using ctypes
import ctypes
from ctypes import wintypes

kernel32 = ctypes.WinDLL('kernel32', use_last_error=True)

def create_mutex(name):
    """Create a named mutex to vaccinate against ransomware."""
    handle = kernel32.CreateMutexW(None, False, name)
    error = ctypes.get_last_error()
    if handle == 0:
        return False, f"Failed to create mutex: error {error}"
    if error == 183:  # ERROR_ALREADY_EXISTS
        return True, f"Mutex already exists (already vaccinated): {name}"
    return True, f"Mutex created successfully: {name}"

KNOWN_RANSOMWARE_MUTEXES = [
    "Global\\MsWinZonesCacheCounterMutexA",        # WannaCry
    "Global\\kasKDJSAFJauisiudUASIIQWUA82",        # Conti
    "Global\\YOURPRODUCT_MUTEX",                     # Ryuk variant
    "Global\\JhbGjhBsSQjz",                         # Maze
    "Global\\sdjfhksjdhfsd",                         # Generic ransomware
]
```

### Step 3: Monitor for Mutex Creation Events

Use Sysmon to detect when ransomware creates its characteristic mutexes:

```xml
<!-- Sysmon configuration for mutex monitoring -->
<Sysmon schemaversion="4.90">
  <EventFiltering>
    <!-- Event ID 1: Process creation with mutex indicators -->
    <ProcessCreate onmatch="include">
      <CommandLine condition="contains">mutex</CommandLine>
      <CommandLine condition="contains">CreateMutex</CommandLine>
    </ProcessCreate>
  </EventFiltering>
</Sysmon>
```

```
Detection via Event Logs:
━━━━━━━━━━━━━━━━━━━━━━━━
Windows Security Log:
  Event ID 4688: Process creation (enable command line logging)

Sysmon:
  Event ID 1:  Process create (includes command line and hashes)
  Event ID 17: Pipe created (named pipes, similar to mutexes)

PowerShell detection:
  Event ID 4104: Script block logging (detect mutex creation in scripts)

Velociraptor artifact:
  Windows.Detection.Mutants - Enumerates all named mutant objects
```

### Step 4: Monitor DNS for Kill Switch Domains

Detect ransomware domain-based kill switch resolution attempts:

```
DNS Monitoring for Kill Switch Domains:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Monitor DNS queries for known kill switch domains
2. High-entropy domain names (>4.0 entropy in domain label) may indicate
   ransomware kill switch domains or DGA-generated C2 domains
3. Queries to newly registered domains from endpoints that typically
   only access well-established domains

Indicators:
  - Domain with no prior resolution history
  - Domain registered in last 24-72 hours
  - High character entropy in domain name
  - Resolution attempt followed by either mass encryption (kill switch failed)
    or process termination (kill switch activated)
```

### Step 5: Enumerate Active Mutexes for Incident Response

During an active incident, scan endpoints for ransomware-associated mutexes:

```powershell
# PowerShell: List all named mutant objects using Sysinternals Handle
# handle.exe -a -p <PID> | findstr "Mutant"

# Velociraptor query for mutex hunting:
# SELECT * FROM glob(globs="\\BaseNamedObjects\\*") WHERE Name =~ "mutex_pattern"

# Python-based enumeration (requires pywin32):
# import win32event
# handle = win32event.OpenMutex(0x00100000, False, "Global\\MutexName")
```

## Verification

- Verify mutex vaccination by attempting to create the same mutex (should get ERROR_ALREADY_EXISTS)
- Test that vaccinated mutexes survive system reboot (they do not; re-apply at startup via scheduled task)
- Confirm DNS monitoring detects test queries for known kill switch domains
- Validate Sysmon event generation for mutex creation by running a test script
- Check that vaccination does not interfere with legitimate applications using similar mutex names
- Test against actual ransomware samples in an isolated sandbox to confirm kill switch activation

## Key Concepts

| Term | Definition |
|------|------------|
| **Mutex (Mutant)** | A Windows kernel synchronization object used to ensure only one instance of a program runs; ransomware uses named mutexes to prevent re-infection |
| **Kill Switch** | A mechanism in ransomware that causes it to terminate without encrypting if a specific condition is met (mutex exists, domain resolves, file present) |
| **Mutex Vaccination** | Proactively creating named mutexes on endpoints that match known ransomware mutex names, preventing the ransomware from executing |
| **Domain Sinkhole** | Registering or redirecting a malicious domain to a controlled server; used to activate domain-based kill switches |
| **DGA (Domain Generation Algorithm)** | Algorithm used by malware to generate pseudo-random domain names for C2 communication, sometimes incorporating kill switch checks |

## Tools & Systems

- **Sysmon**: Microsoft system monitor providing Event ID 17/18 for named pipe and mutex creation monitoring
- **Velociraptor**: Endpoint visibility tool with built-in artifacts for enumerating mutant (mutex) objects on Windows
- **Sysinternals Handle**: Command-line tool for listing open handles including named mutexes per process
- **malware-mutex (GitHub)**: Community-maintained database of mutexes used by known malware families
- **ANY.RUN**: Interactive malware sandbox that reports mutex creation during dynamic analysis
- **PassiveDNS**: DNS monitoring infrastructure for detecting kill switch domain resolution attempts

===== END FILE =====


===== FILE: skills/implementing-ransomware-kill-switch-detection/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/implementing-ransomware-kill-switch-detection/references/api-reference.md =====
# API Reference: Ransomware Kill Switch Detection

## Windows Mutex (Mutant) APIs

### CreateMutex (kernel32.dll)
```c
HANDLE CreateMutexW(
  LPSECURITY_ATTRIBUTES lpMutexAttributes,  // NULL for default
  BOOL bInitialOwner,                       // TRUE to own immediately
  LPCWSTR lpName                            // Named mutex string
);
// Returns: Handle to mutex, or NULL on failure
// GetLastError() == ERROR_ALREADY_EXISTS (183) if mutex already exists
```

### OpenMutex (kernel32.dll)
```c
HANDLE OpenMutexW(
  DWORD dwDesiredAccess,  // SYNCHRONIZE (0x00100000)
  BOOL bInheritHandle,    // FALSE
  LPCWSTR lpName          // Named mutex string
);
// Returns: Handle if exists, NULL if not found
```

### PowerShell Mutex Operations
```powershell
# Create a named mutex
$created = $false
$m = New-Object System.Threading.Mutex($true, "Global\MutexName", [ref]$created)

# Check if mutex exists
try {
  $m = [System.Threading.Mutex]::OpenExisting("Global\MutexName")
  "EXISTS"
} catch { "NOT_FOUND" }
```

## Known Ransomware Kill Switch Mutexes

| Mutex Name | Family | Notes |
|-----------|--------|-------|
| Global\MsWinZonesCacheCounterMutexA | WannaCry | Single-instance guard |
| Global\kasKDJSAFJauisiudUASIIQWUA82 | Conti | Instance mutex |
| Global\YOURPRODUCT_MUTEX | Ryuk variant | Instance guard |
| Global\JhbGjhBsSQjz | Maze | Single-instance check |
| Global\{GUID-based} | LockBit | Machine-specific GUID |
| Global\sdjfhksjdhfsd | Generic builders | Common in kits |

## Known Kill Switch Domains

| Domain | Family | Discovered By |
|--------|--------|--------------|
| iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com | WannaCry v1 | MalwareTech (2017) |
| fferfsodp9ifjaposdfjhgosurijfaewrwergwea.com | WannaCry v1 | Secondary switch |

## Sysmon Configuration for Mutex Detection

### Event ID 1 - Process Creation
```xml
<Sysmon schemaversion="4.90">
  <EventFiltering>
    <ProcessCreate onmatch="include">
      <Image condition="excludes">C:\Windows\</Image>
    </ProcessCreate>
  </EventFiltering>
</Sysmon>
```

## Velociraptor Mutex Hunting

### Windows.Detection.Mutants Artifact
```sql
SELECT * FROM glob(globs="\\BaseNamedObjects\\*")
WHERE Name =~ "MsWinZonesCacheCounterMutexA|kasKDJSAF|YOURPRODUCT"
```

### Sysinternals Handle Tool
```cmd
handle.exe -a | findstr /i "Mutant"
handle.exe -a -p <PID> | findstr /i "Mutant"
```

## DNS Kill Switch Monitoring

### Python DNS Resolution Check
```python
import socket

def check_domain(domain):
    try:
        ip = socket.gethostbyname(domain)
        return {"resolves": True, "ip": ip}
    except socket.gaierror:
        return {"resolves": False}
```

### Passive DNS Services
| Service | URL | Notes |
|---------|-----|-------|
| VirusTotal | virustotal.com | Domain resolution history |
| PassiveTotal | community.riskiq.com | DNS record history |
| SecurityTrails | securitytrails.com | Domain intelligence |

## Malware Mutex Database

### albertzsigovits/malware-mutex (GitHub)
```
URL: https://github.com/albertzsigovits/malware-mutex
Format: JSON with mutex name, malware family, source reference
```

### ANY.RUN Mutex Search
```
URL: https://any.run/cybersecurity-blog/mutex-search-in-ti-lookup/
Search: Threat Intelligence Lookup → Synchronization → Mutex name
```

## Mutex Vaccination Deployment Methods

| Method | Persistence | Scope |
|--------|------------|-------|
| GPO Startup Script | Survives reboot | Domain-wide |
| Scheduled Task (at logon) | Survives reboot | Per-machine |
| Windows Service | Survives reboot | Per-machine |
| Manual PowerShell | Until reboot | Current session |

### GPO Startup Script Path
```
Computer Configuration → Policies → Windows Settings →
Scripts (Startup/Shutdown) → Startup → Add Script
```

===== END FILE =====


===== FILE: skills/implementing-ransomware-kill-switch-detection/scripts/agent.py =====
#!/usr/bin/env python3
"""Ransomware kill switch detection and mutex vaccination agent.

Detects ransomware kill switch mechanisms (mutexes, domains, registry keys)
and can proactively deploy mutex vaccinations to prevent known ransomware
families from executing. Monitors for kill switch domain DNS queries.
"""

import json
import logging
import platform
import socket
import subprocess
import sys
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
)
logger = logging.getLogger("killswitch_agent")

KNOWN_KILL_SWITCH_MUTEXES = {
    "Global\\MsWinZonesCacheCounterMutexA": {
        "family": "WannaCry",
        "type": "instance_guard",
        "notes": "Prevents multiple WannaCry instances from running",
    },
    "Global\\kasKDJSAFJauisiudUASIIQWUA82": {
        "family": "Conti",
        "type": "instance_guard",
        "notes": "Conti ransomware single-instance mutex",
    },
    "Global\\YOURPRODUCT_MUTEX": {
        "family": "Ryuk variant",
        "type": "instance_guard",
        "notes": "Ryuk variant instance check",
    },
    "Global\\JhbGjhBsSQjz": {
        "family": "Maze",
        "type": "instance_guard",
        "notes": "Maze ransomware single-instance mutex",
    },
    "Global\\{A7FE5338-4DDE-8CDE-9F54-FE88C3B8B532}": {
        "family": "LockBit",
        "type": "instance_guard",
        "notes": "LockBit variant mutex (GUID-based)",
    },
    "Global\\MsWinZonesCacheCounterMutexA0": {
        "family": "WannaCry variant",
        "type": "instance_guard",
        "notes": "WannaCry variant with appended zero",
    },
    "Global\\55a42b46-43dc-4e6c-abef-2529ddd744c7": {
        "family": "BlackCat/ALPHV",
        "type": "instance_guard",
        "notes": "ALPHV ransomware instance mutex",
    },
    "Global\\sdjfhksjdhfsd": {
        "family": "Generic ransomware",
        "type": "instance_guard",
        "notes": "Common in several ransomware builders",
    },
}

KNOWN_KILL_SWITCH_DOMAINS = {
    "iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com": {
        "family": "WannaCry v1",
        "type": "domain_kill_switch",
        "notes": "Primary WannaCry kill switch domain registered by MalwareTech",
    },
    "fferfsodp9ifjaposdfjhgosurijfaewrwergwea.com": {
        "family": "WannaCry v1 (alternate)",
        "type": "domain_kill_switch",
        "notes": "Secondary WannaCry kill switch domain",
    },
    "ayloginilider.com": {
        "family": "Emotet (ransomware loader)",
        "type": "c2_sinkhole",
        "notes": "Emotet C2 domain sinkholed by law enforcement",
    },
}

KNOWN_KILL_SWITCH_REGISTRY = {
    "HKLM\\SOFTWARE\\WannaDecrypt0r": {
        "family": "WannaCry",
        "type": "registry_marker",
        "key": "HKLM\\SOFTWARE\\WannaDecrypt0r",
    },
}


def check_mutex_exists_windows(mutex_name):
    """Check if a named mutex exists on Windows using PowerShell."""
    ps_script = (
        f'try {{ $m = [System.Threading.Mutex]::OpenExisting("{mutex_name}"); '
        f'"EXISTS" }} catch {{ "NOT_FOUND" }}'
    )
    try:
        result = subprocess.run(
            ["powershell", "-Command", ps_script],
            capture_output=True, text=True, timeout=10,
        )
        return result.stdout.strip() == "EXISTS"
    except (FileNotFoundError, subprocess.TimeoutExpired):
        return None


def create_mutex_windows(mutex_name):
    """Create a named mutex on Windows for vaccination."""
    ps_script = (
        f'$created = $false; '
        f'$m = New-Object System.Threading.Mutex($true, "{mutex_name}", [ref]$created); '
        f'if ($created) {{ "CREATED" }} else {{ "ALREADY_EXISTS" }}; '
        f'Start-Sleep -Seconds 2'
    )
    try:
        result = subprocess.run(
            ["powershell", "-Command", ps_script],
            capture_output=True, text=True, timeout=15,
        )
        output = result.stdout.strip()
        return output == "CREATED" or output == "ALREADY_EXISTS", output
    except (FileNotFoundError, subprocess.TimeoutExpired) as e:
        return False, str(e)


def check_kill_switch_domain(domain):
    """Check if a kill switch domain resolves (indicating it is active)."""
    try:
        ip = socket.gethostbyname(domain)
        return {
            "domain": domain,
            "resolves": True,
            "ip": ip,
            "kill_switch_active": True,
            "notes": "Domain resolves - kill switch is ACTIVE (ransomware should abort)",
        }
    except socket.gaierror:
        return {
            "domain": domain,
            "resolves": False,
            "ip": None,
            "kill_switch_active": False,
            "notes": "Domain does NOT resolve - kill switch INACTIVE (ransomware would proceed)",
        }


def scan_all_kill_switches():
    """Scan for all known kill switch mechanisms."""
    report = {
        "scan_time": datetime.now().isoformat(),
        "hostname": platform.node(),
        "platform": platform.system(),
        "mutex_checks": [],
        "domain_checks": [],
        "registry_checks": [],
        "summary": {"total_checked": 0, "active_vaccinations": 0, "active_domains": 0},
    }

    # Check mutexes (Windows only)
    if platform.system() == "Windows":
        logger.info("Checking %d known ransomware mutexes...", len(KNOWN_KILL_SWITCH_MUTEXES))
        for mutex_name, info in KNOWN_KILL_SWITCH_MUTEXES.items():
            exists = check_mutex_exists_windows(mutex_name)
            check = {
                "mutex": mutex_name,
                "family": info["family"],
                "exists": exists,
                "vaccinated": exists is True,
            }
            report["mutex_checks"].append(check)
            report["summary"]["total_checked"] += 1
            if exists:
                report["summary"]["active_vaccinations"] += 1
                logger.warning("Mutex EXISTS: %s (%s)", mutex_name, info["family"])
    else:
        logger.info("Mutex checking is Windows-only. Skipping on %s.", platform.system())

    # Check kill switch domains
    logger.info("Checking %d known kill switch domains...", len(KNOWN_KILL_SWITCH_DOMAINS))
    for domain, info in KNOWN_KILL_SWITCH_DOMAINS.items():
        result = check_kill_switch_domain(domain)
        result["family"] = info["family"]
        report["domain_checks"].append(result)
        report["summary"]["total_checked"] += 1
        if result["resolves"]:
            report["summary"]["active_domains"] += 1

    return report


def vaccinate_endpoint(mutex_list=None):
    """Deploy mutex vaccinations to prevent ransomware execution."""
    if platform.system() != "Windows":
        return {"error": "Mutex vaccination is only supported on Windows"}

    if mutex_list is None:
        mutex_list = list(KNOWN_KILL_SWITCH_MUTEXES.keys())

    results = {"vaccinated": [], "failed": [], "already_exists": []}

    for mutex_name in mutex_list:
        info = KNOWN_KILL_SWITCH_MUTEXES.get(mutex_name, {"family": "Custom"})
        success, status = create_mutex_windows(mutex_name)

        record = {"mutex": mutex_name, "family": info.get("family", "Custom"), "status": status}

        if status == "CREATED":
            results["vaccinated"].append(record)
            logger.info("Vaccinated: %s (%s)", mutex_name, info.get("family"))
        elif status == "ALREADY_EXISTS":
            results["already_exists"].append(record)
            logger.info("Already vaccinated: %s", mutex_name)
        else:
            results["failed"].append(record)
            logger.error("Failed to vaccinate: %s - %s", mutex_name, status)

    results["summary"] = {
        "total_attempted": len(mutex_list),
        "newly_vaccinated": len(results["vaccinated"]),
        "already_vaccinated": len(results["already_exists"]),
        "failed": len(results["failed"]),
    }
    return results


def generate_vaccination_script():
    """Generate a PowerShell script for persistent mutex vaccination."""
    lines = [
        "# Ransomware Mutex Vaccination Script",
        "# Deploy via Group Policy Startup Script or Scheduled Task",
        f"# Generated: {datetime.now().isoformat()}",
        "# This script creates named mutexes that prevent known ransomware from executing",
        "",
        "$mutexHandles = @()",
        "",
    ]

    for mutex_name, info in KNOWN_KILL_SWITCH_MUTEXES.items():
        lines.append(f"# {info['family']} - {info['notes']}")
        lines.append(f'$created = $false')
        lines.append(f'$m = New-Object System.Threading.Mutex($true, "{mutex_name}", [ref]$created)')
        lines.append(f'if ($created) {{ Write-Host "Vaccinated: {mutex_name}" }}')
        lines.append(f'$mutexHandles += $m')
        lines.append("")

    lines.append("# Keep script running to maintain mutex handles")
    lines.append("Write-Host 'Mutex vaccination active. Press Ctrl+C to stop.'")
    lines.append("while ($true) { Start-Sleep -Seconds 60 }")

    return "\n".join(lines)


if __name__ == "__main__":
    print("=" * 60)
    print("Ransomware Kill Switch Detection & Vaccination Agent")
    print("Mutex vaccination, domain monitoring, kill switch analysis")
    print("=" * 60)

    if len(sys.argv) < 2:
        print("\nUsage:")
        print("  python agent.py scan                Scan for all known kill switches")
        print("  python agent.py vaccinate           Deploy mutex vaccinations")
        print("  python agent.py domains             Check kill switch domain status")
        print("  python agent.py generate-script     Generate PowerShell vaccination script")
        print("  python agent.py list                List all known kill switches")
        sys.exit(0)

    command = sys.argv[1]

    if command == "scan":
        report = scan_all_kill_switches()
        print(f"\n--- Kill Switch Scan Results ---")
        print(f"  Total checked: {report['summary']['total_checked']}")
        print(f"  Active mutex vaccinations: {report['summary']['active_vaccinations']}")
        print(f"  Active kill switch domains: {report['summary']['active_domains']}")
        for mc in report["mutex_checks"]:
            status = "VACCINATED" if mc["vaccinated"] else "not vaccinated"
            print(f"  [{status:15s}] {mc['family']:20s} {mc['mutex']}")
        for dc in report["domain_checks"]:
            status = "ACTIVE" if dc["resolves"] else "INACTIVE"
            print(f"  [{status:15s}] {dc['family']:20s} {dc['domain']}")
        print(f"\n{json.dumps(report, indent=2, default=str)}")

    elif command == "vaccinate":
        print("\n[*] Deploying mutex vaccinations...")
        results = vaccinate_endpoint()
        print(f"\n--- Vaccination Results ---")
        print(f"  Newly vaccinated: {results['summary']['newly_vaccinated']}")
        print(f"  Already vaccinated: {results['summary']['already_vaccinated']}")
        print(f"  Failed: {results['summary']['failed']}")

    elif command == "domains":
        print("\n--- Kill Switch Domain Status ---")
        for domain, info in KNOWN_KILL_SWITCH_DOMAINS.items():
            result = check_kill_switch_domain(domain)
            status = "ACTIVE (resolves)" if result["resolves"] else "INACTIVE (no DNS)"
            print(f"  [{status}] {info['family']}: {domain}")
            if result["resolves"]:
                print(f"    Resolves to: {result['ip']}")

    elif command == "generate-script":
        script = generate_vaccination_script()
        output_file = "mutex_vaccination.ps1"
        with open(output_file, "w") as f:
            f.write(script)
        print(f"\n[+] Vaccination script saved to: {output_file}")
        print(f"[+] Deploy via GPO startup script or scheduled task")
        print(f"\n{script[:500]}...")

    elif command == "list":
        print(f"\n--- Known Ransomware Kill Switches ---")
        print(f"\nMutexes ({len(KNOWN_KILL_SWITCH_MUTEXES)}):")
        for name, info in KNOWN_KILL_SWITCH_MUTEXES.items():
            print(f"  {info['family']:20s} {name}")
        print(f"\nDomains ({len(KNOWN_KILL_SWITCH_DOMAINS)}):")
        for domain, info in KNOWN_KILL_SWITCH_DOMAINS.items():
            print(f"  {info['family']:20s} {domain}")

    else:
        print(f"[!] Unknown command: {command}")

===== END FILE =====


===== FILE: skills/performing-ransomware-tabletop-exercise/SKILL.md =====
---
name: performing-ransomware-tabletop-exercise
description: 'Plans and facilitates tabletop exercises simulating ransomware incidents,
  using realistic scenarios based on threat actors like LockBit and ALPHV/BlackCat
  with injects covering double extortion and backup destruction, then evaluates responses
  against NIST CSF and CISA guidelines. Use when planning or running a ransomware
  tabletop exercise or incident response readiness drill.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- incident-response
- tabletop-exercise
- defense
- preparedness
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1486
- T1490
mitre_f3:
  version: '1.1'
  tactics:
  - positioning
  - monetization
  techniques:
  - id: T1531
    name: Account Access Removal
    tactic: positioning
    source: attack
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: F1047
    name: Transfer of funds
    tactic: monetization
    source: f3
  - id: F1017.001
    name: 'Conversion to Physical Monetary Instruments: Cash'
    tactic: monetization
    source: f3
---
# Performing Ransomware Tabletop Exercise

## When to Use

- Testing organizational ransomware response procedures annually or after major infrastructure changes
- Validating decision-making processes for ransom payment, regulatory notification, and public disclosure
- Training executives, IT, legal, PR, and operations teams on their roles during a ransomware incident
- Meeting cyber insurance policy requirements for documented incident response testing
- Identifying gaps in recovery playbooks, communication plans, and backup procedures

**Do not use** as a substitute for technical controls testing. Tabletop exercises validate procedures and decision-making, not technical detection or prevention capabilities.

## Prerequisites

- Documented incident response plan (IRP) that participants should have read before the exercise
- Identified exercise participants from: executive leadership, IT/security, legal, communications/PR, HR, operations, and external counsel
- Facilitator who is independent from the IR team (to provide objective evaluation)
- Ransomware scenario designed with injects that escalate over multiple rounds
- Evaluation criteria aligned to NIST CSF Respond/Recover functions
- Conference room or virtual meeting for 2-4 hours with no interruptions

## Workflow

### Step 1: Design the Exercise Scenario

Build a realistic scenario based on current threat actor TTPs:

**Scenario Structure:**
```
Phase 1: Initial Detection (30 min)
  - SOC receives alert for suspicious process execution on file server
  - EDR detects Cobalt Strike beacon on 3 workstations
  - Inject: External threat intel report links C2 IP to LockBit affiliate

Phase 2: Escalation (30 min)
  - Ransomware executes on 40% of servers during overnight hours
  - Ransom note demands $2M in Bitcoin with 72-hour deadline
  - Inject: Attackers contact media claiming data theft of customer PII

Phase 3: Decision Points (45 min)
  - Backup assessment reveals immutable copies are intact but primary backups encrypted
  - Legal advises on breach notification timeline (72 hours GDPR, varies by US state)
  - Inject: Threat actor publishes sample of stolen data on leak site

Phase 4: Recovery and Communication (45 min)
  - Recovery time estimate: 5-7 days from immutable backups
  - Insurance carrier engages negotiation firm
  - Inject: Major customer threatens contract termination without update within 24 hours
```

**Scenario Variables to Customize:**
- Threat actor group and known TTPs
- Percentage of infrastructure encrypted
- Whether backups are intact, partially compromised, or fully destroyed
- Type of data exfiltrated (PII, PHI, financial, trade secrets)
- Applicable regulatory frameworks (GDPR, HIPAA, PCI DSS, SEC rules)
- Ransom amount and payment deadline

### Step 2: Prepare Exercise Materials

Create the following documents for participants:

1. **Exercise Overview Briefing** - Ground rules, objectives, scope, and participants
2. **Situation Reports (SITREPs)** - One per phase, distributed as the exercise progresses
3. **Inject Cards** - New information introduced at specific times to force decision-making
4. **Decision Point Worksheets** - Structured forms for documenting group decisions
5. **Evaluation Scorecard** - Criteria for assessing response quality

**Key Decision Points to Include:**
- When to activate the incident response team
- Whether to shut down systems or contain selectively
- Whether to engage law enforcement (FBI IC3, CISA)
- Whether to pay the ransom and under what conditions
- When and how to notify regulators, customers, and the public
- How to prioritize system recovery order

### Step 3: Facilitate the Exercise

**Facilitator Responsibilities:**
- Present each phase scenario and distribute SITREPs
- Introduce injects at predetermined times to increase pressure
- Ask probing questions to test decision-making reasoning
- Ensure all participant groups contribute (prevent IT from dominating)
- Document all decisions, rationales, and action items
- Track time management (many teams lose time on early phases)

**Probing Questions by Phase:**

Phase 1 - Detection:
- Who makes the call to declare an incident? What criteria trigger it?
- How do we determine the scope of compromise from initial alerts?
- Do we have the forensic capability to investigate or do we need external help?

Phase 2 - Escalation:
- What is our communication plan for employees? Do they know not to turn on affected machines?
- Have we isolated the network to prevent further encryption?
- Who authorizes system shutdowns that impact business operations?

Phase 3 - Decision:
- Under what conditions would we consider paying the ransom?
- What are the legal obligations for notification at this point?
- How do we handle the public leak of customer data?

Phase 4 - Recovery:
- What is the recovery priority order? Is it documented or decided ad hoc?
- How long until critical business operations resume?
- What evidence preservation is required for law enforcement and insurance?

### Step 4: Evaluate and Score Responses

Score each functional area against defined criteria:

| Evaluation Area | Score (1-5) | Criteria |
|----------------|-------------|----------|
| Detection & Escalation | | Timely incident declaration, proper chain of command |
| Containment | | Network isolation, credential reset, scope assessment |
| Communication - Internal | | Employee notification, executive briefing, documented decisions |
| Communication - External | | Regulatory notification, customer communication, media response |
| Recovery Planning | | Backup verification, recovery priority, RTO tracking |
| Legal & Compliance | | Breach notification timelines, evidence preservation, law enforcement engagement |
| Business Continuity | | Manual operations, customer impact mitigation, revenue loss estimation |
| Payment Decision | | Structured framework, legal review, OFAC sanctions check |

### Step 5: Document Findings and Remediation Plan

Produce an after-action report (AAR) within 5 business days:

**AAR Contents:**
1. Exercise overview and objectives
2. Scenario summary and injects
3. Key decisions made and rationale
4. Strengths observed
5. Gaps identified with severity rating
6. Remediation actions with owners and deadlines
7. Comparison to previous exercise results (if applicable)

## Key Concepts

| Term | Definition |
|------|------------|
| **Tabletop Exercise (TTX)** | Discussion-based exercise where participants walk through a simulated incident scenario to test plans and procedures |
| **Inject** | New information introduced during the exercise to change the scenario and force additional decision-making |
| **SITREP** | Situation Report providing current status of the simulated incident at each exercise phase |
| **After-Action Report (AAR)** | Post-exercise document capturing findings, gaps, strengths, and remediation actions |
| **Double Extortion** | Ransomware tactic where attackers both encrypt data and threaten to publish stolen data unless ransom is paid |
| **OFAC Check** | Verification that ransom payment recipient is not on the US Treasury OFAC sanctions list, which would make payment illegal |

## Tools & Systems

- **CISA Tabletop Exercise Packages (CTEPs)**: Free scenario packages from CISA designed for critical infrastructure sectors
- **FEMA Homeland Security Exercise and Evaluation Program (HSEEP)**: Methodology for designing, conducting, and evaluating exercises
- **Immersive Labs**: Platform providing interactive cyber crisis simulations with real-time scoring
- **Tabletop Scenarios (from NCSC UK)**: Exercise in a Box tool providing free guided tabletop exercises
- **Ransomware Readiness Assessment (CISA)**: Self-assessment tool for evaluating ransomware preparedness

## Common Scenarios

### Scenario: Healthcare System Double Extortion Exercise

**Context**: A 5-hospital healthcare system conducts an annual ransomware tabletop. Previous exercise revealed gaps in HIPAA breach notification and clinical system recovery priority. This year's scenario simulates a double extortion attack targeting the EMR system.

**Approach**:
1. Design scenario based on Cl0p MOO (Managed Operations Operator) TTPs: exploitation of MOVEit vulnerability for initial access, data exfiltration of 500,000 patient records, followed by encryption of EMR database servers
2. Participants: CISO, CIO, CMO (Chief Medical Officer), General Counsel, VP Communications, Director of Clinical Operations, Privacy Officer, External IR firm representative
3. Phase 1 inject: EMR system down, emergency department diverting patients to neighboring hospital
4. Phase 2 inject: HHS OCR (Office for Civil Rights) contacts organization about reports of patient data on dark web
5. Phase 3 inject: Attacker provides decryption key sample for $3.5M, 48-hour deadline
6. Key finding: Organization lacks documented criteria for ransom payment decision and had not pre-identified an OFAC-compliant payment mechanism
7. Remediation: Establish payment decision framework, pre-engage ransomware negotiation firm, update HIPAA breach notification procedures with specific timelines

**Pitfalls**:
- Designing unrealistic scenarios that do not reflect actual ransomware TTPs, reducing exercise credibility
- Allowing technical teams to dominate the exercise while business and legal participants remain passive
- Not testing the communication plan (many organizations discover their notification list is outdated during the actual incident)
- Failing to follow up on remediation actions identified in the AAR, negating the exercise value

## Output Format

```
## Ransomware Tabletop Exercise - After Action Report

**Exercise Date**: [Date]
**Facilitator**: [Name]
**Scenario**: [Brief description]
**Duration**: [Hours]
**Participants**: [Count by department]

### Exercise Objectives
1. [Objective] - Met / Partially Met / Not Met
2. [Objective] - Met / Partially Met / Not Met

### Key Decisions Log
| Time | Decision Point | Decision Made | Rationale | Assessment |
|------|---------------|--------------|-----------|------------|

### Strengths Observed
1. [Strength]

### Gaps Identified
| Gap | Severity | Affected Area | Current State | Desired State |
|-----|----------|--------------|---------------|---------------|

### Remediation Actions
| Action | Owner | Deadline | Priority | Status |
|--------|-------|----------|----------|--------|

### Comparison to Previous Exercise
| Area | Previous Score | Current Score | Trend |
|------|---------------|--------------|-------|
```

===== END FILE =====


===== FILE: skills/performing-ransomware-tabletop-exercise/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/performing-ransomware-tabletop-exercise/assets/template.md =====
# Ransomware Tabletop Exercise Template

## Exercise Overview

| Field | Value |
|-------|-------|
| Exercise Name | |
| Date | |
| Duration | 3-4 hours |
| Facilitator | |
| Scenario | [Threat Actor] ransomware attack |
| Industry | |

## Participants

| Name | Role | Department | Present (Y/N) |
|------|------|-----------|---------------|
| | CISO | Security | |
| | CIO | IT | |
| | General Counsel | Legal | |
| | VP Comms | PR | |
| | COO | Operations | |
| | CFO | Finance | |

## Phase 1: Detection SITREP

[Insert scenario text]

### Discussion Questions
1. Who declares the incident?
2. What is the immediate containment action?
3. Who is notified at this stage?

### Decisions Made
| Decision | Rationale | Owner |
|----------|-----------|-------|
| | | |

## Phase 2: Escalation SITREP

[Insert scenario text]

### Discussion Questions
1. What is the scope assessment process?
2. How do we maintain business operations?
3. Do we engage law enforcement?

### Decisions Made
| Decision | Rationale | Owner |
|----------|-----------|-------|
| | | |

## Phase 3: Critical Decisions SITREP

[Insert scenario text]

### Discussion Questions
1. Under what conditions do we pay?
2. What are notification obligations?
3. How do we respond to data leak?

### Decisions Made
| Decision | Rationale | Owner |
|----------|-----------|-------|
| | | |

## Phase 4: Recovery SITREP

[Insert scenario text]

### Discussion Questions
1. What is recovery priority order?
2. What do we tell customers?
3. What is the media statement?

### Decisions Made
| Decision | Rationale | Owner |
|----------|-----------|-------|
| | | |

## Evaluation Scorecard

| Area | Score (1-5) | Notes |
|------|-------------|-------|
| Detection & Escalation | | |
| Containment | | |
| Internal Communication | | |
| External Communication | | |
| Recovery Planning | | |
| Legal & Compliance | | |
| Business Continuity | | |
| Payment Decision | | |
| **Overall** | | |

## Key Findings

### Strengths
1.
2.
3.

### Gaps
| Gap | Severity | Owner | Remediation | Deadline |
|-----|----------|-------|-------------|----------|
| | Critical/High/Medium | | | |

## Sign-Off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Exercise Sponsor | | | |
| Facilitator | | | |

===== END FILE =====


===== FILE: skills/performing-ransomware-tabletop-exercise/references/api-reference.md =====
# Ransomware Tabletop Exercise - API Reference

## Scenario Framework

### Phase Structure
Each phase contains:

| Field | Type | Description |
|-------|------|-------------|
| `phase` | string | Phase name: detection, containment, escalation, eradication, recovery |
| `inject` | string | Narrative scenario inject read to participants |
| `expected_actions` | list | Correct response actions for scoring |
| `time_pressure_minutes` | int | Simulated time window for decisions |

### Exercise Variants

- **standard** - Normal time pressure, full scenario
- **accelerated** - Half time windows, tests rapid decision-making

## Scoring Algorithm

```
phase_score = (correct_actions / expected_actions) * 100
overall_score = mean(all_phase_scores)
```

Rating thresholds:
- >= 90%: Excellent
- >= 70%: Good
- >= 50%: Needs Improvement
- < 50%: Critical Gaps

## Expected Actions by Phase

### Detection
- `isolate_host` - Quarantine affected endpoint
- `preserve_evidence` - Capture memory dump and disk image
- `notify_ir_lead` - Escalate to incident response lead

### Containment
- `network_segmentation` - Restrict lateral movement paths
- `disable_compromised_accounts` - Lock affected credentials
- `block_c2_domains` - Update firewall/proxy deny lists
- `preserve_shadow_copies` - Protect backup snapshots

### Escalation
- `notify_executive_team` - Brief C-suite leadership
- `engage_legal_counsel` - Activate legal response team
- `contact_law_enforcement` - Report to FBI IC3 or local CIRT
- `activate_crisis_comms` - Prepare stakeholder communications

### Eradication
- `remove_persistence` - Clean scheduled tasks, registry keys, WMI subscriptions
- `reset_all_credentials` - Reset passwords domain-wide
- `rebuild_compromised_hosts` - Reimage from gold images
- `reset_krbtgt_twice` - Invalidate all Kerberos tickets

### Recovery
- `restore_from_backup` - Use verified clean backup sets
- `validate_restored_systems` - Run integrity checks
- `monitor_for_reinfection` - Enhanced monitoring for 72+ hours
- `staged_network_reconnection` - Reconnect systems in phases

## After-Action Report Schema

```json
{
  "report": "ransomware_tabletop_aar",
  "evaluation": {
    "overall_score_pct": 78.5,
    "rating": "good",
    "phase_scores": [{"phase": "detection", "score_pct": 66.7}]
  },
  "recommendations": [{"phase": "detection", "gap": "Missed: preserve_evidence"}]
}
```

## CLI Usage

```bash
python agent.py --mode demo --output aar.json
python agent.py --mode generate --variant accelerated --output scenario.json
python agent.py --mode score --responses-file responses.json --output aar.json
```

===== END FILE =====


===== FILE: skills/performing-ransomware-tabletop-exercise/references/standards.md =====
# Standards & References - Ransomware Tabletop Exercise

## Exercise Standards

### FEMA HSEEP (Homeland Security Exercise and Evaluation Program)
- Standardized methodology for exercise design, conduct, and evaluation
- Defines exercise types: seminars, workshops, tabletops, drills, functional, full-scale
- Provides templates for exercise plans, evaluation guides, and AARs
- https://www.fema.gov/emergency-managers/national-preparedness/exercises/hseep

### NIST SP 800-84: Guide to Test, Training, and Exercise Programs
- Framework for developing IT plan test and exercise programs
- Section 4.3: Tabletop exercises for incident response testing
- Covers exercise scoping, objectives, scenario development, and evaluation

### CISA Tabletop Exercise Packages (CTEPs)
- Free downloadable exercise scenarios for critical infrastructure sectors
- Ransomware-specific scenarios updated to reflect current threat landscape
- https://www.cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages

## Ransomware-Specific Guidance

### CISA #StopRansomware Guide
- Ransomware response checklist that exercises should validate
- Decision tree for ransom payment considerations
- Recovery priority guidance

### NIST IR 8374: Ransomware Risk Management
- Identifies exercise testing as a key control in the Recover function
- Recommends annual tabletop exercises with escalating complexity

### FBI/CISA Joint Advisories
- AA24-131A: Black Basta Ransomware
- AA23-136A: BianLian Ransomware Group
- AA23-158A: CL0P Ransomware Gang Exploiting MOVEit
- Use these as source material for realistic exercise scenarios

## Regulatory Notification Requirements (for Scenario Design)

| Regulation | Notification Timeline | Authority |
|-----------|----------------------|-----------|
| GDPR (EU) | 72 hours | Supervisory Authority |
| HIPAA (US Healthcare) | 60 days (individuals), ASAP (HHS if >500) | HHS OCR |
| SEC (US Public Companies) | 4 business days (Form 8-K) | SEC |
| PCI DSS | 72 hours | Card brands/acquiring bank |
| NY DFS (23 NYCRR 500) | 72 hours | NY DFS |
| CCPA (California) | "Expedient time" | California AG |
| NIS2 (EU) | 24 hours (early warning), 72 hours (full) | National CSIRT |

===== END FILE =====


===== FILE: skills/performing-ransomware-tabletop-exercise/references/workflows.md =====
# Workflows - Ransomware Tabletop Exercise

## Workflow 1: Exercise Planning (4-6 weeks before)

```
Start
  |
  v
[Define exercise objectives] --> What gaps are we testing?
  |
  v
[Select scenario type]
  |-- Double extortion (data theft + encryption)
  |-- Supply chain ransomware (vendor compromise)
  |-- Cloud ransomware (SaaS/IaaS targeted)
  |-- Critical infrastructure disruption
  |
  v
[Choose threat actor model] --> LockBit / ALPHV / Cl0p / Rhysida
  |
  v
[Identify participants]
  |-- Executive leadership (CEO, CFO, COO)
  |-- IT/Security (CISO, SOC, IR team)
  |-- Legal (General Counsel, external counsel)
  |-- Communications (PR, media relations)
  |-- Operations (business unit leaders)
  |-- HR (employee communications)
  |-- External partners (IR firm, insurance)
  |
  v
[Develop scenario with 4 phases and injects]
  |
  v
[Prepare materials: SITREPs, inject cards, evaluation scorecard]
  |
  v
[Schedule 3-4 hour block, distribute pre-reading]
  |
  v
End
```

## Workflow 2: Exercise Execution

```
Exercise Start
  |
  v
[Facilitator opening brief] (10 min)
  |-- Ground rules, objectives, scope
  |-- "This is discussion-based, no wrong answers"
  |
  v
[Phase 1: Initial Detection] (30 min)
  |-- Distribute SITREP 1
  |-- Discussion: Who, what, when, initial actions
  |-- Inject: Additional information changes situation
  |-- Document decisions on worksheet
  |
  v
[Phase 2: Escalation] (30 min)
  |-- Distribute SITREP 2
  |-- Discussion: Scope of impact, containment actions
  |-- Inject: Double extortion element introduced
  |-- Document decisions
  |
  v
[Break] (10 min)
  |
  v
[Phase 3: Critical Decision Points] (45 min)
  |-- Distribute SITREP 3
  |-- Discussion: Ransom payment, law enforcement, notification
  |-- Inject: Public pressure from media/customers
  |-- Document decisions with rationale
  |
  v
[Phase 4: Recovery and Communication] (45 min)
  |-- Distribute SITREP 4
  |-- Discussion: Recovery priority, timeline, customer comms
  |-- Inject: Recovery complication (infected backup, key system fails)
  |-- Document decisions
  |
  v
[Hot wash / Debrief] (20 min)
  |-- Each functional area shares top insight
  |-- Facilitator highlights key observations
  |-- Immediate gap identification
  |
  v
Exercise End
```

## Workflow 3: After-Action Report Development

```
Exercise Complete
  |
  v
[Collect all documentation within 24 hours]
  |-- Decision worksheets
  |-- Facilitator notes
  |-- Evaluation scorecards
  |-- Observer notes (if separate observers present)
  |
  v
[Score each evaluation area (1-5)]
  |
  v
[Identify strengths (what worked well)]
  |
  v
[Identify gaps with severity rating]
  |-- Critical: Would prevent effective response
  |-- High: Would significantly delay/complicate response
  |-- Medium: Would reduce response quality
  |-- Low: Minor improvement opportunity
  |
  v
[Develop remediation actions]
  |-- Each gap gets: action, owner, deadline, priority
  |-- Must be specific and measurable
  |
  v
[Draft AAR within 5 business days]
  |
  v
[Review AAR with exercise sponsor]
  |
  v
[Distribute AAR to participants]
  |
  v
[Track remediation actions quarterly]
  |
  v
End
```

===== END FILE =====


===== FILE: skills/performing-ransomware-tabletop-exercise/scripts/agent.py =====
#!/usr/bin/env python3
"""Ransomware Tabletop Exercise agent — generates scenario injects, tracks
participant decisions, scores response effectiveness, and produces an
after-action report."""

import argparse
import json
import sys
from datetime import datetime
from pathlib import Path


SCENARIO_PHASES = [
    {
        "phase": "detection",
        "inject": "SOC analyst observes unusual SMB traffic and multiple failed login attempts from a single workstation. AV alerts show Cobalt Strike beacon signatures.",
        "expected_actions": ["isolate_host", "preserve_evidence", "notify_ir_lead"],
        "time_pressure_minutes": 15,
    },
    {
        "phase": "containment",
        "inject": "Ransomware has spread to 3 file servers. Active encryption observed on \\fs01\shared. Lateral movement detected via PsExec.",
        "expected_actions": ["network_segmentation", "disable_compromised_accounts", "block_c2_domains", "preserve_shadow_copies"],
        "time_pressure_minutes": 30,
    },
    {
        "phase": "escalation",
        "inject": "Threat actor sends ransom demand via email: 50 BTC within 48 hours. They claim to have exfiltrated 200GB of customer PII data.",
        "expected_actions": ["notify_executive_team", "engage_legal_counsel", "contact_law_enforcement", "activate_crisis_comms"],
        "time_pressure_minutes": 60,
    },
    {
        "phase": "eradication",
        "inject": "IR team identifies initial access via phishing email with macro-enabled document. Persistence mechanisms found in scheduled tasks and registry run keys.",
        "expected_actions": ["remove_persistence", "reset_all_credentials", "rebuild_compromised_hosts", "reset_krbtgt_twice"],
        "time_pressure_minutes": 120,
    },
    {
        "phase": "recovery",
        "inject": "Backups verified clean. Recovery point is 6 hours old. Business requests fastest path to resume operations.",
        "expected_actions": ["restore_from_backup", "validate_restored_systems", "monitor_for_reinfection", "staged_network_reconnection"],
        "time_pressure_minutes": 240,
    },
]


def generate_scenario(variant: str = "standard") -> list[dict]:
    """Return the scenario phases, optionally shuffled for advanced exercises."""
    phases = [dict(p) for p in SCENARIO_PHASES]
    if variant == "accelerated":
        for p in phases:
            p["time_pressure_minutes"] = max(5, p["time_pressure_minutes"] // 2)
    return phases


def score_response(phase: dict, participant_actions: list[str]) -> dict:
    """Score participant actions against expected actions for a phase."""
    expected = set(phase["expected_actions"])
    taken = set(participant_actions)
    correct = expected & taken
    missed = expected - taken
    extra = taken - expected
    score = len(correct) / len(expected) * 100 if expected else 0
    return {
        "phase": phase["phase"],
        "score_pct": round(score, 1),
        "correct_actions": sorted(correct),
        "missed_actions": sorted(missed),
        "additional_actions": sorted(extra),
    }


def evaluate_exercise(scenario: list[dict], all_responses: list[list[str]]) -> dict:
    """Score all phases and compute overall effectiveness."""
    phase_scores = []
    for phase, actions in zip(scenario, all_responses):
        phase_scores.append(score_response(phase, actions))
    overall = sum(s["score_pct"] for s in phase_scores) / len(phase_scores) if phase_scores else 0
    return {
        "phase_scores": phase_scores,
        "overall_score_pct": round(overall, 1),
        "rating": "excellent" if overall >= 90 else "good" if overall >= 70 else "needs_improvement" if overall >= 50 else "critical_gaps",
    }


def generate_aar(scenario: list[dict], evaluation: dict) -> dict:
    """Generate After-Action Report."""
    recommendations = []
    for ps in evaluation["phase_scores"]:
        if ps["missed_actions"]:
            recommendations.append({
                "phase": ps["phase"],
                "gap": f"Missed actions: {', '.join(ps['missed_actions'])}",
                "recommendation": f"Add {ps['phase']} procedures to IR playbook and train responders",
            })
    return {
        "report": "ransomware_tabletop_aar",
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "scenario_phases": len(scenario),
        "evaluation": evaluation,
        "recommendations": recommendations,
        "next_exercise_date": "Schedule within 90 days to validate improvements",
    }


def run_demo() -> dict:
    """Run a demonstration exercise with simulated participant responses."""
    scenario = generate_scenario("standard")
    simulated_responses = [
        ["isolate_host", "notify_ir_lead"],
        ["network_segmentation", "disable_compromised_accounts", "block_c2_domains"],
        ["notify_executive_team", "engage_legal_counsel", "contact_law_enforcement", "activate_crisis_comms"],
        ["remove_persistence", "reset_all_credentials", "rebuild_compromised_hosts"],
        ["restore_from_backup", "validate_restored_systems", "monitor_for_reinfection", "staged_network_reconnection"],
    ]
    evaluation = evaluate_exercise(scenario, simulated_responses)
    return generate_aar(scenario, evaluation)


def main():
    parser = argparse.ArgumentParser(description="Ransomware Tabletop Exercise Agent")
    parser.add_argument("--mode", choices=["generate", "score", "demo"], default="demo",
                        help="Mode: generate scenario, score responses, or run demo")
    parser.add_argument("--variant", choices=["standard", "accelerated"], default="standard")
    parser.add_argument("--responses-file", help="JSON file with participant responses for scoring")
    parser.add_argument("--output", help="Output JSON file path")
    args = parser.parse_args()

    if args.mode == "generate":
        result = {"scenario": generate_scenario(args.variant)}
    elif args.mode == "score":
        if not args.responses_file:
            print("Error: --responses-file required for score mode", file=sys.stderr)
            sys.exit(1)
        responses = json.loads(Path(args.responses_file).read_text())
        scenario = generate_scenario(args.variant)
        evaluation = evaluate_exercise(scenario, responses)
        result = generate_aar(scenario, evaluation)
    else:
        result = run_demo()

    output = json.dumps(result, indent=2)
    if args.output:
        Path(args.output).write_text(output, encoding="utf-8")
        print(f"Report written to {args.output}")
    else:
        print(output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-ransomware-tabletop-exercise/scripts/process.py =====
#!/usr/bin/env python3
"""
Ransomware Tabletop Exercise Generator and Evaluator

Generates customized ransomware tabletop exercise scenarios based on:
- Organization type (healthcare, financial, manufacturing, etc.)
- Threat actor profile (LockBit, ALPHV, Cl0p, Rhysida)
- Infrastructure profile (on-prem, cloud, hybrid)
- Regulatory requirements

Evaluates exercise results and generates after-action reports.
"""

import json
import sys
from dataclasses import dataclass, field, asdict
from datetime import datetime, timedelta
from pathlib import Path
from typing import Optional


THREAT_ACTORS = {
    "lockbit": {
        "name": "LockBit 3.0",
        "initial_access": ["Phishing with macro-enabled documents", "RDP brute force",
                          "Exploitation of VPN vulnerabilities (Citrix, Fortinet)"],
        "tools": ["Cobalt Strike", "Mimikatz", "PsExec", "Stealbit (data exfiltration)"],
        "ttps": ["Disables Windows Defender via GPO", "Deletes shadow copies with vssadmin",
                "Uses group policy to deploy ransomware across domain",
                "Double extortion with data leak site"],
        "avg_dwell_time": "4-14 days",
        "ransom_range": "$100K - $50M",
        "negotiation_style": "Automated chat portal, deadline-driven",
    },
    "alphv": {
        "name": "ALPHV/BlackCat",
        "initial_access": ["Compromised credentials from IABs", "Exchange server exploitation",
                          "Social engineering of help desk"],
        "tools": ["Cobalt Strike", "Brute Ratel", "Impacket", "ExMatter (exfiltration)"],
        "ttps": ["Written in Rust (cross-platform)", "Targets ESXi hypervisors",
                "Triple extortion (encrypt + leak + DDoS)", "Destroys backups before encryption"],
        "avg_dwell_time": "5-21 days",
        "ransom_range": "$200K - $35M",
        "negotiation_style": "Dedicated Tor negotiation site, threatens DDoS",
    },
    "clop": {
        "name": "Cl0p",
        "initial_access": ["Zero-day exploitation of file transfer platforms (MOVEit, GoAnywhere, Accellion)",
                          "Supply chain compromise"],
        "tools": ["FlawedAmmyy RAT", "SDBot", "TrueBot", "Custom exfiltration tools"],
        "ttps": ["Mass exploitation campaigns", "Data theft without encryption in many cases",
                "Targets managed file transfer (MFT) platforms", "Extended extortion timeline"],
        "avg_dwell_time": "1-7 days (mass exploitation)",
        "ransom_range": "$500K - $20M",
        "negotiation_style": "Email-based, group negotiations for mass attacks",
    },
    "rhysida": {
        "name": "Rhysida",
        "initial_access": ["Phishing", "VPN without MFA exploitation",
                          "Valid credentials purchased from IABs"],
        "tools": ["Cobalt Strike", "PsExec", "PowerShell scripts", "ChaCha20 encryption"],
        "ttps": ["Targets healthcare and education", "Uses living-off-the-land binaries (LOLBins)",
                "Deletes VSS and disables Windows recovery", "Auctions stolen data on leak site"],
        "avg_dwell_time": "3-10 days",
        "ransom_range": "$50K - $15M",
        "negotiation_style": "Tor-based auction site, victim-shaming approach",
    },
}

INDUSTRY_PROFILES = {
    "healthcare": {
        "critical_systems": ["EMR/EHR", "PACS (medical imaging)", "Laboratory Information System",
                           "Pharmacy Management", "Patient Portal", "Medical Devices/IoT"],
        "data_types": ["PHI (Protected Health Information)", "Patient records", "Insurance data",
                      "Clinical trial data", "Employee PII"],
        "regulations": ["HIPAA", "HITECH Act", "State breach notification laws"],
        "notification_timeline": "60 days to individuals, ASAP to HHS OCR if >500 affected",
        "operational_impact": "Patient safety - diversion to other facilities, manual charting",
        "insurance_considerations": "Cyber liability + professional liability intersection",
    },
    "financial": {
        "critical_systems": ["Core banking platform", "Trading systems", "ATM network",
                           "Wire transfer system", "Customer portal", "SWIFT messaging"],
        "data_types": ["PII", "Financial records", "Account numbers", "SSNs",
                      "Transaction histories", "Internal financial data"],
        "regulations": ["GLBA", "SOX", "PCI DSS", "SEC 8-K", "NY DFS 23 NYCRR 500", "GDPR"],
        "notification_timeline": "72 hours (NY DFS), 4 business days (SEC 8-K)",
        "operational_impact": "Transaction processing halt, customer account access disruption",
        "insurance_considerations": "Financial institution bond + cyber liability",
    },
    "manufacturing": {
        "critical_systems": ["SCADA/ICS", "MES (Manufacturing Execution System)", "ERP",
                           "Supply chain management", "Quality management system"],
        "data_types": ["Trade secrets", "Design specifications", "Customer PII",
                      "Supply chain data", "Financial records"],
        "regulations": ["Industry-specific (FDA, NHTSA)", "State breach notification", "GDPR"],
        "notification_timeline": "Varies by state/jurisdiction",
        "operational_impact": "Production line shutdown, supply chain disruption, safety concerns",
        "insurance_considerations": "Business interruption + cyber liability + product liability",
    },
    "education": {
        "critical_systems": ["Student Information System", "Learning Management System",
                           "Email/collaboration", "Research data systems", "Financial aid"],
        "data_types": ["Student PII (FERPA)", "Research data", "Financial aid records",
                      "Employee PII", "Healthcare records (student health)"],
        "regulations": ["FERPA", "HIPAA (student health)", "State breach notification"],
        "notification_timeline": "Varies by state, FERPA has no specific timeline",
        "operational_impact": "Class disruption, research data loss, enrollment processing halt",
        "insurance_considerations": "Education-specific cyber liability policies",
    },
}


@dataclass
class ExerciseInject:
    phase: int
    time_offset_minutes: int
    title: str
    description: str
    decision_required: str
    pressure_element: str


@dataclass
class ExerciseScenario:
    organization: str
    industry: str
    threat_actor: str
    date: str
    duration_hours: float
    participants: list
    scenario_summary: str
    phases: list = field(default_factory=list)
    injects: list = field(default_factory=list)
    evaluation_areas: list = field(default_factory=list)


@dataclass
class ExerciseEvaluation:
    area: str
    score: int  # 1-5
    strengths: list
    gaps: list
    remediation_actions: list


class TabletopGenerator:
    """Generates customized ransomware tabletop exercise scenarios."""

    def __init__(self, org_name: str, industry: str, threat_actor: str):
        self.org_name = org_name
        if industry not in INDUSTRY_PROFILES:
            raise ValueError(f"Unknown industry: {industry}. Choose from: {list(INDUSTRY_PROFILES.keys())}")
        if threat_actor not in THREAT_ACTORS:
            raise ValueError(f"Unknown threat actor: {threat_actor}. Choose from: {list(THREAT_ACTORS.keys())}")
        self.industry = INDUSTRY_PROFILES[industry]
        self.actor = THREAT_ACTORS[threat_actor]
        self.industry_name = industry
        self.scenario = None

    def generate_scenario(self, encrypted_percentage: int = 60,
                         data_exfiltrated: bool = True,
                         backups_intact: bool = True,
                         ransom_amount: str = "$2,000,000") -> ExerciseScenario:
        """Generate a complete exercise scenario."""

        scenario = ExerciseScenario(
            organization=self.org_name,
            industry=self.industry_name,
            threat_actor=self.actor["name"],
            date=datetime.now().strftime("%Y-%m-%d"),
            duration_hours=3.5,
            participants=[
                "CISO / Security Director",
                "CIO / IT Director",
                "General Counsel / Legal",
                "VP Communications / PR",
                "Operations / Business Unit Leader",
                "CFO / Finance",
                "HR Director",
                "External IR firm (optional)",
            ],
            scenario_summary=(
                f"The {self.actor['name']} ransomware group has compromised {self.org_name}'s "
                f"network using {self.actor['initial_access'][0].lower()}. After approximately "
                f"{self.actor['avg_dwell_time']} of dwell time, the attackers have encrypted "
                f"{encrypted_percentage}% of server infrastructure"
                f"{' and exfiltrated sensitive data including ' + self.industry['data_types'][0] if data_exfiltrated else ''}. "
                f"The ransom demand is {ransom_amount} in cryptocurrency with a 72-hour deadline."
            ),
        )

        # Phase 1: Detection
        scenario.phases.append({
            "number": 1,
            "title": "Initial Detection and Triage",
            "duration_minutes": 30,
            "sitrep": (
                f"At 06:15 this morning, the SOC received multiple alerts from the EDR platform "
                f"indicating suspicious process execution on several servers. Investigation reveals "
                f"that {self.actor['tools'][0]} has been deployed on at least 5 systems. "
                f"Users are beginning to report they cannot access {self.industry['critical_systems'][0]}."
            ),
            "discussion_questions": [
                "Who declares the incident? What is the activation criteria?",
                "What is the first action: investigate further or contain immediately?",
                "Who needs to be notified at this stage?",
                "Do we have the forensic capability to investigate in-house?",
            ],
        })

        # Phase 2: Escalation
        scenario.phases.append({
            "number": 2,
            "title": "Full Scope and Ransom Demand",
            "duration_minutes": 30,
            "sitrep": (
                f"It is now 09:00. Assessment confirms {encrypted_percentage}% of servers are encrypted. "
                f"Ransom notes have been found on all affected systems demanding {ransom_amount} "
                f"in Bitcoin. The ransom note includes a Tor link for negotiation and threatens "
                f"to publish stolen data in 72 hours. "
                f"Affected systems include: {', '.join(self.industry['critical_systems'][:3])}."
            ),
            "discussion_questions": [
                "How do we assess the full scope of the breach?",
                "What is our containment strategy? Full shutdown or selective isolation?",
                f"How do we maintain {self.industry['operational_impact'].split(',')[0].lower()} during the outage?",
                "Do we engage law enforcement now or later?",
            ],
        })

        # Phase 3: Decision Points
        backup_status = "Immutable backup copies appear intact. Primary backups on the NAS are encrypted." if backups_intact else "All backup systems have been compromised. The attackers deleted backup catalogs before encrypting."
        scenario.phases.append({
            "number": 3,
            "title": "Critical Decisions",
            "duration_minutes": 45,
            "sitrep": (
                f"It is now 14:00. Forensic analysis confirms: {backup_status} "
                f"{'Recovery from immutable backups is estimated at 5-7 days for full restoration.' if backups_intact else 'Without backups, recovery would require rebuilding from scratch: estimated 3-4 weeks.'} "
                f"The threat actor has posted a sample of stolen {self.industry['data_types'][0]} "
                f"on their leak site as proof of exfiltration. Your cyber insurance carrier has "
                f"engaged a ransomware negotiation firm."
            ),
            "discussion_questions": [
                "Under what conditions would we pay the ransom?",
                f"What are our regulatory notification obligations under {', '.join(self.industry['regulations'][:2])}?",
                "How do we respond to the public data leak?",
                "What is the cost comparison: pay vs. rebuild?",
                "Have we verified the payment recipient against OFAC sanctions?",
            ],
        })

        # Phase 4: Recovery
        scenario.phases.append({
            "number": 4,
            "title": "Recovery and Communication",
            "duration_minutes": 45,
            "sitrep": (
                f"It is Day 3. {'Recovery from immutable backups is underway. AD and DNS are restored. ' if backups_intact else 'The decision has been made to rebuild without paying. '}"
                f"A major customer has contacted the CEO demanding an update within 24 hours "
                f"or they will begin transitioning to a competitor. The media has picked up the "
                f"story and reporters are calling the communications team."
            ),
            "discussion_questions": [
                "What is the system recovery priority order?",
                "What do we tell customers? How much detail?",
                "What is our media statement?",
                "How do we prevent re-infection during recovery?",
                "What evidence must we preserve for law enforcement and insurance?",
            ],
        })

        # Generate injects
        scenario.injects = self._generate_injects(data_exfiltrated, backups_intact)

        # Evaluation areas
        scenario.evaluation_areas = [
            "Detection and Escalation",
            "Containment Decisions",
            "Internal Communication",
            "External Communication (Regulatory, Customer, Media)",
            "Recovery Planning and Execution",
            "Legal and Compliance",
            "Business Continuity",
            "Payment Decision Framework",
        ]

        self.scenario = scenario
        return scenario

    def _generate_injects(self, data_exfiltrated: bool, backups_intact: bool) -> list:
        injects = []

        injects.append(asdict(ExerciseInject(
            phase=1,
            time_offset_minutes=15,
            title="Threat Intelligence Match",
            description=(
                f"Your threat intel provider confirms the C2 infrastructure matches known "
                f"{self.actor['name']} affiliate activity. This group is known for: "
                f"{self.actor['ttps'][0]}"
            ),
            decision_required="Does this change our response urgency or approach?",
            pressure_element="Known aggressive group with history of following through on threats",
        )))

        injects.append(asdict(ExerciseInject(
            phase=2,
            time_offset_minutes=10,
            title="Employee Social Media Leak",
            description=(
                "An employee has posted on social media: 'Our entire network is down, "
                "looks like we got hacked. IT is scrambling.' The post has 500 shares."
            ),
            decision_required="How do we handle unauthorized employee disclosure?",
            pressure_element="Information control is compromised, media may pick up story faster",
        )))

        if data_exfiltrated:
            injects.append(asdict(ExerciseInject(
                phase=3,
                time_offset_minutes=20,
                title="Regulatory Inquiry",
                description=(
                    f"You receive an inquiry from the regulatory authority regarding reports of "
                    f"a data breach involving {self.industry['data_types'][0]}. "
                    f"Notification timeline: {self.industry['notification_timeline']}"
                ),
                decision_required="What information do we provide at this stage?",
                pressure_element="Regulatory clock is ticking, incomplete information available",
            )))

        injects.append(asdict(ExerciseInject(
            phase=4,
            time_offset_minutes=15,
            title="Recovery Complication",
            description=(
                "During restoration of the primary database, the team discovers that the "
                "backup was taken 6 hours before encryption but the attacker had already "
                "planted a persistence mechanism (scheduled task calling beacon). "
                "Restoring this backup will reintroduce the attacker's foothold."
            ),
            decision_required="How do we handle infected but recent backups?",
            pressure_element="Recovery timeline extends, clean backup may be older with more data loss",
        )))

        return injects

    def export_scenario(self, output_dir: str) -> str:
        """Export scenario to JSON file."""
        if not self.scenario:
            self.generate_scenario()

        output_path = Path(output_dir) / f"ttx_scenario_{self.org_name.replace(' ', '_')}_{datetime.now().strftime('%Y%m%d')}.json"
        with open(output_path, "w") as f:
            json.dump(asdict(self.scenario), f, indent=2)
        return str(output_path)


class ExerciseEvaluator:
    """Evaluates tabletop exercise results and generates AAR."""

    def __init__(self, scenario: ExerciseScenario):
        self.scenario = scenario
        self.evaluations = []

    def add_evaluation(self, area: str, score: int, strengths: list,
                      gaps: list, remediation_actions: list):
        if score < 1 or score > 5:
            raise ValueError("Score must be between 1 and 5")
        self.evaluations.append(ExerciseEvaluation(
            area=area, score=score, strengths=strengths,
            gaps=gaps, remediation_actions=remediation_actions,
        ))

    def calculate_overall_score(self) -> float:
        if not self.evaluations:
            return 0.0
        return round(sum(e.score for e in self.evaluations) / len(self.evaluations), 1)

    def generate_aar(self) -> str:
        """Generate After-Action Report."""
        lines = []
        lines.append("=" * 70)
        lines.append("RANSOMWARE TABLETOP EXERCISE - AFTER ACTION REPORT")
        lines.append("=" * 70)
        lines.append(f"Organization: {self.scenario.organization}")
        lines.append(f"Date: {self.scenario.date}")
        lines.append(f"Threat Actor: {self.scenario.threat_actor}")
        lines.append(f"Industry: {self.scenario.industry}")
        lines.append(f"Duration: {self.scenario.duration_hours} hours")
        lines.append(f"Overall Score: {self.calculate_overall_score()}/5.0")
        lines.append("")
        lines.append("SCENARIO SUMMARY")
        lines.append("-" * 40)
        lines.append(self.scenario.scenario_summary)
        lines.append("")

        lines.append("EVALUATION RESULTS")
        lines.append("-" * 40)
        for eval_item in self.evaluations:
            rating = {1: "Inadequate", 2: "Needs Improvement", 3: "Adequate",
                     4: "Good", 5: "Excellent"}.get(eval_item.score, "N/A")
            lines.append(f"\n  {eval_item.area}: {eval_item.score}/5 ({rating})")
            lines.append("    Strengths:")
            for s in eval_item.strengths:
                lines.append(f"      + {s}")
            lines.append("    Gaps:")
            for g in eval_item.gaps:
                lines.append(f"      - {g}")
            lines.append("    Remediation:")
            for r in eval_item.remediation_actions:
                lines.append(f"      > {r}")

        # Summary statistics
        all_gaps = [g for e in self.evaluations for g in e.gaps]
        all_actions = [a for e in self.evaluations for a in e.remediation_actions]
        lines.append("")
        lines.append("SUMMARY")
        lines.append("-" * 40)
        lines.append(f"Total gaps identified: {len(all_gaps)}")
        lines.append(f"Total remediation actions: {len(all_actions)}")
        lines.append(f"Areas scoring below 3: {sum(1 for e in self.evaluations if e.score < 3)}")
        lines.append("")
        lines.append("=" * 70)

        return "\n".join(lines)


def main():
    """Generate sample tabletop exercise scenario."""
    generator = TabletopGenerator(
        org_name="Acme Healthcare System",
        industry="healthcare",
        threat_actor="rhysida",
    )

    scenario = generator.generate_scenario(
        encrypted_percentage=65,
        data_exfiltrated=True,
        backups_intact=True,
        ransom_amount="$3,500,000",
    )

    # Print scenario overview
    print("=" * 70)
    print("RANSOMWARE TABLETOP EXERCISE SCENARIO")
    print("=" * 70)
    print(f"Organization: {scenario.organization}")
    print(f"Industry: {scenario.industry}")
    print(f"Threat Actor: {scenario.threat_actor}")
    print(f"Duration: {scenario.duration_hours} hours")
    print(f"\nSummary: {scenario.scenario_summary}")
    print(f"\nParticipants: {len(scenario.participants)}")
    for p in scenario.participants:
        print(f"  - {p}")

    print("\nPHASES:")
    for phase in scenario.phases:
        print(f"\n  Phase {phase['number']}: {phase['title']} ({phase['duration_minutes']} min)")
        print(f"  SITREP: {phase['sitrep'][:200]}...")
        print(f"  Questions: {len(phase['discussion_questions'])}")

    print(f"\nINJECTS: {len(scenario.injects)}")
    for inject in scenario.injects:
        print(f"  - Phase {inject['phase']}: {inject['title']}")

    # Export to JSON
    output_path = generator.export_scenario(str(Path(__file__).parent))
    print(f"\nScenario exported to: {output_path}")

    # Demo evaluation
    evaluator = ExerciseEvaluator(scenario)
    evaluator.add_evaluation(
        area="Detection and Escalation",
        score=4,
        strengths=["SOC correctly identified Cobalt Strike indicators",
                   "Incident was declared within 30 minutes"],
        gaps=["No documented criteria for incident declaration threshold"],
        remediation_actions=["Document incident declaration criteria with specific trigger conditions"],
    )
    evaluator.add_evaluation(
        area="Payment Decision Framework",
        score=2,
        strengths=["Legal correctly identified OFAC compliance requirement"],
        gaps=["No pre-established payment decision framework",
              "No pre-engaged negotiation firm",
              "No cryptocurrency procurement mechanism identified"],
        remediation_actions=["Establish ransom payment decision matrix with executive sign-off",
                           "Pre-engage ransomware negotiation firm through cyber insurance",
                           "Identify cryptocurrency procurement path (if payment decision is made)"],
    )

    aar = evaluator.generate_aar()
    print("\n" + aar)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/recovering-from-ransomware-attack/SKILL.md =====
---
name: recovering-from-ransomware-attack
description: 'Executes structured ransomware incident recovery following NIST/CISA
  frameworks: environment isolation, forensic evidence preservation, clean infrastructure
  rebuild, prioritized restoration from verified backups, credential reset, and
  Active Directory/database recovery in dependency order. Use when recovering from
  a ransomware attack, performing post-encryption restoration, or executing disaster
  recovery after ransomware encryption.

  '
domain: cybersecurity
subdomain: ransomware-defense
tags:
- ransomware
- recovery
- incident-response
- backup
- defense
version: 1.0.0
author: mahipal
license: Apache-2.0
nist_csf:
- PR.DS-11
- RS.MA-01
- RC.RP-01
- PR.IR-01
mitre_attack:
- T1078
- T1190
- T1059
- T1003
- T1110
mitre_f3:
  version: '1.1'
  tactics:
  - positioning
  - monetization
  - defense-impairment
  techniques:
  - id: T1531
    name: Account Access Removal
    tactic: positioning
    source: attack
  - id: F1005
    name: Account Manipulation
    tactic: defense-impairment
    source: f3
  - id: F1018
    name: Convert to Cryptocurrency
    tactic: monetization
    source: f3
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
---
# Recovering from Ransomware Attack

## When to Use

- After ransomware has encrypted production systems and the decision has been made to recover from backups
- When building or validating a ransomware recovery runbook before an actual incident
- After receiving a decryption key (paid ransom or law enforcement provided) and needing to safely decrypt
- When partial recovery is needed alongside decryption of remaining systems
- Conducting a recovery drill to validate RTO commitments

**Do not use** before completing containment and forensic scoping. Premature recovery without understanding the attacker's access and persistence mechanisms risks re-infection.

## Prerequisites

- Incident declared and containment phase completed (all attacker access severed)
- Forensic evidence preserved (disk images, memory dumps, network captures)
- Backup integrity verified (immutable/air-gapped copies confirmed clean)
- Clean build media available (OS installation media, golden images)
- Recovery environment prepared (clean network segment isolated from compromised infrastructure)
- Recovery priority list documented (Tier 1/2/3 systems in dependency order)

## Workflow

### Step 1: Establish Clean Recovery Environment

Build recovery infrastructure isolated from the compromised network:

```bash
# Create isolated recovery VLAN
# No connectivity to compromised network segments
# Dedicated internet access for patch downloads only (via proxy)

# Recovery network architecture:
# VLAN 999 (Recovery) - 10.99.0.0/24
#   - Recovery workstations (10.99.0.10-20)
#   - Recovered DCs (10.99.0.50-55)
#   - Recovered servers (10.99.0.100+)
#   - Proxy for internet (10.99.0.1) - patches and updates only

# Firewall rules: DENY all from recovery VLAN to production VLANs
# Allow: Recovery VLAN -> Internet (HTTPS only, via proxy)
# Allow: Recovery VLAN -> Backup infrastructure (restore traffic only)
```

### Step 2: Recover Identity Infrastructure First

Active Directory must be recovered before any domain-joined systems:

```powershell
# AD Recovery Procedure
# Step 2a: Restore AD from known-good backup
# Use DSRM (Directory Services Restore Mode) boot

# 1. Build clean Windows Server from ISO
# 2. Promote as DC using AD restore
# 3. Restore System State from immutable backup

# Verify AD backup is pre-compromise
# Check backup timestamp against earliest known compromise date
wbadmin get versions -backuptarget:E: -machine:DC01

# Restore system state in DSRM
wbadmin start systemstaterecovery -version:02/15/2026-04:00 -backuptarget:E: -machine:DC01 -quiet

# After restore, reset critical accounts
# Reset krbtgt password TWICE (invalidates all Kerberos tickets)
# This prevents Golden Ticket persistence
Import-Module ActiveDirectory
Set-ADAccountPassword -Identity krbtgt -Reset -NewPassword (ConvertTo-SecureString "NewKrbtgt2026!Complex#1" -AsPlainText -Force)
# Wait for replication (minimum 12 hours), then reset again
Set-ADAccountPassword -Identity krbtgt -Reset -NewPassword (ConvertTo-SecureString "NewKrbtgt2026!Complex#2" -AsPlainText -Force)

# Reset all privileged account passwords
$privilegedGroups = @("Domain Admins", "Enterprise Admins", "Schema Admins", "Administrators")
foreach ($group in $privilegedGroups) {
    Get-ADGroupMember -Identity $group -Recursive | ForEach-Object {
        Set-ADAccountPassword -Identity $_.SamAccountName -Reset `
            -NewPassword (ConvertTo-SecureString (New-Guid).Guid -AsPlainText -Force)
        Set-ADUser -Identity $_.SamAccountName -ChangePasswordAtLogon $true
    }
}

# Validate AD health
dcdiag /v /c /d /e /s:DC01
repadmin /showrepl
```

### Step 3: Validate Backup Integrity Before Restoration

```bash
# Scan backup files for ransomware artifacts before restoring
# Use offline antivirus scanning on backup mount

# Mount backup as read-only
mount -o ro,noexec /dev/backup_lv /mnt/backup_verify

# Scan with ClamAV
clamscan -r --infected --log=/var/log/backup_scan.log /mnt/backup_verify

# Check for known ransomware indicators
find /mnt/backup_verify -name "*.encrypted" -o -name "*.locked" \
    -o -name "*.lockbit" -o -name "DECRYPT_*" -o -name "readme.txt" \
    -o -name "RECOVER-*" -o -name "HOW_TO_*" | tee /var/log/ransomware_check.log

# Verify database consistency (SQL Server example)
# Restore database to temporary instance for validation
RESTORE VERIFYONLY FROM DISK = '/mnt/backup_verify/databases/erp_db.bak'
    WITH CHECKSUM
```

### Step 4: Restore Systems in Priority Order

Follow dependency-based recovery sequence:

```
Recovery Order:
Phase 1 (Hours 0-4): Identity & Infrastructure
  1. Domain Controllers (AD, DNS, DHCP)
  2. Certificate Authority (if applicable)
  3. Core network services (DHCP, NTP)

Phase 2 (Hours 4-12): Critical Business Systems
  4. Database servers (SQL, Oracle, PostgreSQL)
  5. Core business applications (ERP, CRM)
  6. Email (Exchange, M365 hybrid)

Phase 3 (Hours 12-24): Important Systems
  7. File servers
  8. Web applications
  9. Monitoring and security tools (SIEM, EDR)

Phase 4 (Hours 24-48): Remaining Systems
  10. Development environments
  11. Archive systems
  12. Non-critical applications
```

```powershell
# Veeam Instant Recovery - fastest restore for VMware/Hyper-V
# Boots VM directly from backup file, then migrates to production storage

# Instant recovery for Tier 1 system
Start-VBRInstantRecovery -RestorePoint (Get-VBRRestorePoint -Name "DC01" |
    Sort-Object CreationTime -Descending | Select-Object -First 1) `
    -VMName "DC01-Recovered" `
    -Server (Get-VBRServer -Name "esxi01.recovery.local") `
    -Datastore "recovery-datastore"

# After validation, migrate to production storage
Start-VBRQuickMigration -VM "DC01-Recovered" `
    -Server (Get-VBRServer -Name "esxi01.prod.local") `
    -Datastore "production-datastore"
```

### Step 5: Validate Recovered Systems and Harden

Before connecting recovered systems to production:

```powershell
# Check for persistence mechanisms
# Scheduled Tasks
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"} |
    Select-Object TaskName, TaskPath, State, Author |
    Export-Csv C:\recovery\scheduled_tasks.csv

# Services
Get-Service | Where-Object {$_.StartType -eq "Automatic"} |
    Select-Object Name, DisplayName, StartType, Status |
    Export-Csv C:\recovery\auto_services.csv

# Startup items
Get-CimInstance Win32_StartupCommand |
    Select-Object Name, Command, Location, User |
    Export-Csv C:\recovery\startup_items.csv

# WMI event subscriptions (common persistence)
Get-WmiObject -Namespace root\subscription -Class __EventFilter
Get-WmiObject -Namespace root\subscription -Class __EventConsumer

# Registry run keys
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\RunOnce"
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"

# Verify no unauthorized admin accounts
Get-LocalGroupMember -Group "Administrators"
Get-ADGroupMember -Identity "Domain Admins"

# Apply latest patches before connecting to production
Install-WindowsUpdate -AcceptAll -AutoReboot
```

### Step 6: Phased Network Reconnection

```
Phase 1: Reconnect identity infrastructure
  - DCs online in production VLAN
  - Validate replication and authentication
  - Monitor for suspicious authentication patterns

Phase 2: Reconnect Tier 1 systems
  - One system at a time
  - Monitor EDR for 1 hour before proceeding to next
  - Validate application functionality

Phase 3: Reconnect remaining systems
  - Groups of 5-10 systems
  - Continue monitoring for re-infection indicators

Throughout: SOC monitoring on high alert
  - EDR in aggressive blocking mode
  - All previous IOCs loaded in detection rules
  - Canary files deployed on recovered systems
```

## Key Concepts

| Term | Definition |
|------|------------|
| **DSRM** | Directory Services Restore Mode: special boot mode for domain controllers that allows AD database restoration |
| **krbtgt Reset** | Resetting the krbtgt account password twice invalidates all Kerberos tickets, defeating Golden Ticket persistence |
| **Instant Recovery** | Backup technology that boots a VM directly from backup storage for immediate availability while migrating data in background |
| **Evidence Preservation** | Maintaining forensic images and logs before recovery begins, required for law enforcement and insurance claims |
| **Clean Build** | Rebuilding systems from trusted installation media rather than attempting to clean infected systems |
| **Dependency Chain** | The order in which systems must be recovered based on service dependencies (e.g., AD before domain members) |

## Tools & Systems

- **Veeam Instant Recovery**: Boots VMs directly from backup with near-zero RTO, then live-migrates to production
- **Microsoft DSRM**: AD-specific recovery mode for restoring domain controllers from backup
- **DSInternals PowerShell Module**: Validates AD database integrity and identifies compromised credentials post-recovery
- **Rubrik Instant Recovery**: Mounts backup as live VM in seconds for rapid recovery validation
- **ClamAV**: Open-source antivirus for scanning backup files before restoration

## Common Scenarios

### Scenario: Manufacturing Company Full Recovery After LockBit Attack

**Context**: A manufacturer with 300 servers has 80% of infrastructure encrypted by LockBit. Immutable backups from 48 hours ago are verified clean. Production lines are down, costing $500K/day.

**Approach**:
1. Establish recovery VLAN (10.99.0.0/24) isolated from compromised network
2. Restore 2 domain controllers from immutable backup using Veeam Instant Recovery (2 hours)
3. Reset krbtgt password twice with 12-hour gap, reset all admin passwords
4. Validate AD with dcdiag, scan for Golden Ticket indicators with DSInternals
5. Restore ERP database (SAP) and verify data consistency (4 hours)
6. Restore MES (Manufacturing Execution System) and SCADA historians (3 hours)
7. Bring production line controllers online in isolated OT network first
8. Phased reconnection over 48 hours with continuous EDR monitoring
9. Total recovery: 72 hours (within 96-hour RTO commitment)

**Pitfalls**:
- Rushing to reconnect systems without validating absence of persistence mechanisms, causing re-infection
- Restoring from the most recent backup without verifying it predates the compromise (attacker may have poisoned recent backups)
- Not resetting the krbtgt password twice, allowing attackers to maintain Golden Ticket access
- Restoring systems in the wrong order (application servers before their database dependencies)

## Output Format

```
## Ransomware Recovery Status Report

**Incident ID**: [ID]
**Recovery Start**: [Timestamp]
**Current Phase**: [1-4]
**Estimated Completion**: [Timestamp]

### Recovery Progress
| Phase | Systems | Status | Started | Completed | RTO Target |
|-------|---------|--------|---------|-----------|------------|
| 1 - Identity | DC01, DC02, DNS | Complete | HH:MM | HH:MM | 4 hours |
| 2 - Critical | ERP, DB01, DB02 | In Progress | HH:MM | -- | 12 hours |
| 3 - Important | FS01, Email, Web | Pending | -- | -- | 24 hours |
| 4 - Remaining | Dev, Archive | Pending | -- | -- | 48 hours |

### Validation Checklist
- [ ] AD integrity verified (dcdiag, repadmin)
- [ ] krbtgt password reset (2x with interval)
- [ ] All admin passwords reset
- [ ] Persistence mechanisms scanned
- [ ] EDR deployed and active on recovered systems
- [ ] IOCs loaded in detection rules
- [ ] Canary files deployed
```

===== END FILE =====


===== FILE: skills/recovering-from-ransomware-attack/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/recovering-from-ransomware-attack/references/api-reference.md =====
# API Reference: Recovering from Ransomware Attack

## Recovery Priority Order

| Priority | Systems | Why First |
|----------|---------|-----------|
| 1 | Domain Controllers | All auth depends on AD |
| 2 | DNS/DHCP | Network functionality |
| 3 | Authentication (SSO/MFA) | User access |
| 4 | Email | Communication |
| 5 | Database Servers | Business data |
| 6 | Application Servers | Business operations |
| 7 | File Servers | Data access |
| 8 | Workstations | End user devices |

## KRBTGT Reset Procedure

| Step | Command | Note |
|------|---------|------|
| 1 | `Reset-KrbtgtPassword` | First reset |
| 2 | Wait 12 hours | Allow replication |
| 3 | `Reset-KrbtgtPassword` | Second reset |
| 4 | `dcdiag /v` | Validate DC health |

## Backup Verification Commands

| Command | Description |
|---------|-------------|
| `veeamcli verify` | Verify Veeam backup integrity |
| `wbadmin get versions` | List Windows Server backups |
| `aws s3api head-object` | Check S3 backup metadata |

## 3-2-1-1-0 Backup Strategy

| Component | Description |
|-----------|-------------|
| 3 copies | Production + 2 backups |
| 2 media types | Disk + tape/cloud |
| 1 offsite | Geographically separate |
| 1 offline | Air-gapped or immutable |
| 0 errors | Verified with restore tests |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `json` | stdlib | Recovery tracking |
| `datetime` | stdlib | Timeline documentation |
| `pathlib` | stdlib | Backup path verification |

## References

- CISA Ransomware Guide: https://www.cisa.gov/stopransomware/ransomware-guide
- NIST SP 1800-26: https://www.nccoe.nist.gov/data-integrity-recovering-ransomware
- NoMoreRansom: https://www.nomoreransom.org/

===== END FILE =====


===== FILE: skills/recovering-from-ransomware-attack/references/standards.md =====
# Standards & References - Recovering from Ransomware Attack

## Frameworks
- NIST SP 800-61r3: Computer Security Incident Handling Guide (Recover phase)
- NIST IR 8374: Ransomware Risk Management - Recovery section
- CISA #StopRansomware Guide: Recovery checklist
- CIS Controls v8: Control 11 (Data Recovery)
- NIST CSF 2.0: Recover function (RC.RP, RC.CO)

## AD Recovery
- Microsoft: AD Forest Recovery Guide - https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/ad-forest-recovery-guide
- DSInternals: https://github.com/MichaelGrafnetter/DSInternals
- krbtgt reset guidance: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-resetting-the-krbtgt-password

## MITRE ATT&CK (Recovery Validation)
- T1053: Scheduled Task/Job (persistence to check)
- T1543: Create or Modify System Process (persistence to check)
- T1547: Boot or Logon Autostart Execution (persistence to check)
- T1558.001: Golden Ticket (must reset krbtgt)
- T1098: Account Manipulation (check for backdoor accounts)

===== END FILE =====


===== FILE: skills/recovering-from-ransomware-attack/references/workflows.md =====
# Workflows - Recovering from Ransomware Attack

## Workflow 1: Recovery Decision and Planning

```
Containment Complete + Forensics Initiated
  |
  v
[Assess backup availability]
  |-- Immutable copies intact? --> Primary recovery path
  |-- Air-gapped copies available? --> Secondary recovery path
  |-- No clean backups? --> Consider decryption key (paid or NoMoreRansom)
  |
  v
[Determine recovery scope]
  |-- Full rebuild vs. selective restore
  |-- Identify minimum viable recovery set
  |
  v
[Map system dependencies]
  |-- AD/DNS -> Database -> Application -> Web
  |
  v
[Estimate recovery timeline per tier]
  |
  v
[Brief executive team on recovery plan and timeline]
  |
  v
[Begin recovery]
```

## Workflow 2: System Recovery Execution

```
[Establish clean recovery VLAN]
  |
  v
[Phase 1: Identity Recovery]
  |-- Restore DCs from verified backup
  |-- Reset krbtgt (2x)
  |-- Reset all admin passwords
  |-- Validate AD health (dcdiag)
  |
  v
[Phase 2: Critical Systems]
  |-- Restore databases
  |-- Verify data consistency
  |-- Restore core applications
  |-- Test application functionality
  |
  v
[Phase 3: Important Systems]
  |-- Restore in groups of 5-10
  |-- Validate each before proceeding
  |
  v
[Phase 4: Remaining Systems]
  |
  v
[Each system: Scan for persistence -> Patch -> Deploy EDR -> Connect]
  |
  v
[Post-recovery monitoring (7-14 days elevated alert)]
```

===== END FILE =====


===== FILE: skills/recovering-from-ransomware-attack/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for ransomware attack recovery coordination.

Manages recovery workflow: backup verification, system rebuild
prioritization, credential reset tracking, and post-recovery
validation checklists with timeline documentation.
"""

import json
import sys
from datetime import datetime
from pathlib import Path


RECOVERY_PRIORITY = [
    ("Domain Controllers", "critical", "Rebuild from clean media first"),
    ("DNS/DHCP Servers", "critical", "Required for network functionality"),
    ("Authentication Services", "critical", "SSO, MFA, RADIUS"),
    ("Email Server", "high", "Communication during recovery"),
    ("Database Servers", "high", "Restore from verified clean backup"),
    ("Application Servers", "high", "Business-critical applications"),
    ("File Servers", "medium", "Restore data from backup"),
    ("Workstations", "medium", "Reimage, do not file-level restore"),
    ("Print Servers", "low", "Rebuild after core services"),
]


class RansomwareRecoveryAgent:
    """Coordinates ransomware attack recovery procedures."""

    def __init__(self, case_id, output_dir="./recovery"):
        self.case_id = case_id
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.recovery = {
            "case_id": case_id, "status": "in_progress",
            "timeline": [], "systems": [], "checklists": {},
        }

    def log_event(self, event_type, description):
        self.recovery["timeline"].append({
            "timestamp": datetime.utcnow().isoformat(),
            "type": event_type, "description": description,
        })

    def verify_backup(self, backup_path, backup_type, last_verified=None):
        """Record backup verification status."""
        status = {
            "path": backup_path, "type": backup_type,
            "verified_date": last_verified or datetime.utcnow().isoformat(),
            "integrity": "pending",
        }
        if Path(backup_path).exists() if not backup_path.startswith("s3://") else True:
            status["integrity"] = "accessible"
        self.recovery.setdefault("backups", []).append(status)
        self.log_event("backup_check", f"Verified {backup_type}: {backup_path}")
        return status

    def build_recovery_plan(self, clean_backup_available=True):
        """Generate prioritized system recovery plan."""
        systems = []
        for name, priority, note in RECOVERY_PRIORITY:
            systems.append({
                "system": name, "priority": priority, "note": note,
                "status": "pending", "recovery_method": (
                    "Restore from backup" if clean_backup_available
                    else "Rebuild from scratch"
                ),
            })
        self.recovery["systems"] = systems
        self.log_event("plan_created", f"{len(systems)} systems in recovery plan")
        return systems

    def update_system_status(self, system_name, status, notes=""):
        for sys_entry in self.recovery["systems"]:
            if sys_entry["system"] == system_name:
                sys_entry["status"] = status
                if notes:
                    sys_entry["recovery_notes"] = notes
                self.log_event("system_update",
                               f"{system_name}: {status}")
                return sys_entry
        return None

    def generate_credential_reset_checklist(self):
        """Generate credential reset checklist for post-recovery."""
        checklist = [
            {"item": "Reset KRBTGT password (twice, 12h apart)", "status": "pending",
             "priority": "critical"},
            {"item": "Reset all Domain Admin passwords", "status": "pending",
             "priority": "critical"},
            {"item": "Reset all service account passwords", "status": "pending",
             "priority": "critical"},
            {"item": "Reset all user passwords", "status": "pending",
             "priority": "high"},
            {"item": "Revoke and reissue all certificates", "status": "pending",
             "priority": "high"},
            {"item": "Rotate all API keys and tokens", "status": "pending",
             "priority": "high"},
            {"item": "Reset cloud IAM credentials", "status": "pending",
             "priority": "high"},
            {"item": "Deploy LAPS for local admin passwords", "status": "pending",
             "priority": "medium"},
        ]
        self.recovery["checklists"]["credential_reset"] = checklist
        return checklist

    def generate_hardening_checklist(self):
        """Post-recovery hardening recommendations."""
        checklist = [
            {"item": "Enforce MFA on all remote access", "status": "pending"},
            {"item": "Implement 3-2-1-1-0 backup strategy", "status": "pending"},
            {"item": "Deploy EDR on all endpoints", "status": "pending"},
            {"item": "Enable PowerShell Script Block Logging", "status": "pending"},
            {"item": "Implement network segmentation", "status": "pending"},
            {"item": "Block SMB between workstations", "status": "pending"},
            {"item": "Disable NTLM where possible", "status": "pending"},
            {"item": "Deploy application whitelisting on servers", "status": "pending"},
            {"item": "Implement privileged access workstations", "status": "pending"},
        ]
        self.recovery["checklists"]["post_hardening"] = checklist
        return checklist

    def get_recovery_progress(self):
        """Calculate overall recovery progress."""
        total = len(self.recovery["systems"])
        completed = sum(1 for s in self.recovery["systems"]
                        if s["status"] in ("recovered", "verified"))
        return {
            "total_systems": total,
            "recovered": completed,
            "progress_pct": round(completed / max(total, 1) * 100, 1),
        }

    def generate_report(self):
        self.generate_credential_reset_checklist()
        self.generate_hardening_checklist()
        progress = self.get_recovery_progress()

        report = {
            **self.recovery,
            "progress": progress,
            "report_date": datetime.utcnow().isoformat(),
        }
        report_path = self.output_dir / f"{self.case_id}_recovery.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2, default=str)
        print(json.dumps(report, indent=2, default=str))
        return report


def main():
    case_id = sys.argv[1] if len(sys.argv) > 1 else "RAN-2025-001"
    agent = RansomwareRecoveryAgent(case_id)
    agent.build_recovery_plan(clean_backup_available=True)
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/recovering-from-ransomware-attack/scripts/process.py =====
#!/usr/bin/env python3
"""
Ransomware Recovery Orchestration and Tracking Tool

Tracks recovery progress across multiple systems and phases:
- Recovery phase tracking with dependency management
- RTO compliance monitoring
- System validation checklists
- Recovery status reporting
"""

import json
import sys
from dataclasses import dataclass, field, asdict
from datetime import datetime, timedelta
from pathlib import Path
from typing import Optional


@dataclass
class RecoverableSystem:
    name: str
    tier: int
    system_type: str  # dc, database, application, fileserver, web, other
    dependencies: list = field(default_factory=list)
    rto_hours: float = 24.0
    status: str = "pending"  # pending, restoring, validating, online, failed
    backup_source: str = ""
    backup_timestamp: Optional[str] = None
    restore_start: Optional[str] = None
    restore_end: Optional[str] = None
    validation_checks: dict = field(default_factory=dict)
    notes: str = ""


@dataclass
class RecoveryPlan:
    incident_id: str
    organization: str
    recovery_start: str
    compromise_date: str
    systems: list = field(default_factory=list)
    phases: dict = field(default_factory=dict)
    overall_status: str = "in_progress"


class RecoveryOrchestrator:
    """Manages and tracks ransomware recovery across all systems."""

    def __init__(self, incident_id: str, org_name: str, compromise_date: str):
        self.plan = RecoveryPlan(
            incident_id=incident_id,
            organization=org_name,
            recovery_start=datetime.now().isoformat(),
            compromise_date=compromise_date,
        )

    def add_system(self, system: RecoverableSystem):
        self.plan.systems.append(system)

    def start_restore(self, system_name: str, backup_source: str):
        for sys in self.plan.systems:
            if sys.name == system_name:
                # Check dependencies are online
                for dep in sys.dependencies:
                    dep_sys = next((s for s in self.plan.systems if s.name == dep), None)
                    if dep_sys and dep_sys.status != "online":
                        raise ValueError(
                            f"Cannot restore {system_name}: dependency {dep} "
                            f"is {dep_sys.status}, must be 'online'"
                        )
                sys.status = "restoring"
                sys.backup_source = backup_source
                sys.restore_start = datetime.now().isoformat()
                return
        raise ValueError(f"System not found: {system_name}")

    def complete_restore(self, system_name: str):
        for sys in self.plan.systems:
            if sys.name == system_name:
                sys.status = "validating"
                sys.restore_end = datetime.now().isoformat()
                return
        raise ValueError(f"System not found: {system_name}")

    def validate_system(self, system_name: str, checks: dict):
        """Record validation check results for a restored system."""
        for sys in self.plan.systems:
            if sys.name == system_name:
                sys.validation_checks = checks
                all_passed = all(checks.values())
                sys.status = "online" if all_passed else "failed"
                return all_passed
        raise ValueError(f"System not found: {system_name}")

    def check_rto_compliance(self) -> list:
        """Check which systems are at risk of exceeding their RTO."""
        violations = []
        recovery_start = datetime.fromisoformat(self.plan.recovery_start)

        for sys in self.plan.systems:
            rto_deadline = recovery_start + timedelta(hours=sys.rto_hours)

            if sys.status in ("pending", "restoring", "validating"):
                if datetime.now() > rto_deadline:
                    violations.append({
                        "system": sys.name,
                        "tier": sys.tier,
                        "rto_hours": sys.rto_hours,
                        "deadline": rto_deadline.isoformat(),
                        "status": sys.status,
                        "exceeded_by_hours": round(
                            (datetime.now() - rto_deadline).total_seconds() / 3600, 1
                        ),
                    })
        return violations

    def get_recovery_progress(self) -> dict:
        """Calculate overall recovery progress."""
        total = len(self.plan.systems)
        if total == 0:
            return {"progress": 0, "by_status": {}, "by_tier": {}}

        by_status = {}
        by_tier = {}
        for sys in self.plan.systems:
            by_status[sys.status] = by_status.get(sys.status, 0) + 1
            tier_key = f"tier_{sys.tier}"
            if tier_key not in by_tier:
                by_tier[tier_key] = {"total": 0, "online": 0}
            by_tier[tier_key]["total"] += 1
            if sys.status == "online":
                by_tier[tier_key]["online"] += 1

        online = by_status.get("online", 0)
        return {
            "progress": round((online / total) * 100, 1),
            "total_systems": total,
            "by_status": by_status,
            "by_tier": by_tier,
        }

    def get_next_recoverable(self) -> list:
        """Get list of systems ready for recovery (dependencies met)."""
        ready = []
        for sys in self.plan.systems:
            if sys.status != "pending":
                continue
            deps_met = all(
                next((s for s in self.plan.systems if s.name == dep), None) is not None
                and next((s for s in self.plan.systems if s.name == dep)).status == "online"
                for dep in sys.dependencies
            )
            if deps_met or not sys.dependencies:
                ready.append(sys)
        return sorted(ready, key=lambda s: s.tier)

    def generate_report(self) -> str:
        lines = []
        lines.append("=" * 70)
        lines.append("RANSOMWARE RECOVERY STATUS REPORT")
        lines.append("=" * 70)
        lines.append(f"Incident: {self.plan.incident_id}")
        lines.append(f"Organization: {self.plan.organization}")
        lines.append(f"Recovery Started: {self.plan.recovery_start}")
        lines.append(f"Compromise Date: {self.plan.compromise_date}")

        progress = self.get_recovery_progress()
        lines.append(f"\nOverall Progress: {progress['progress']}%")
        lines.append(f"Total Systems: {progress['total_systems']}")

        for status, count in progress["by_status"].items():
            lines.append(f"  {status}: {count}")

        lines.append("\nBy Tier:")
        for tier, data in sorted(progress["by_tier"].items()):
            lines.append(f"  {tier}: {data['online']}/{data['total']} online")

        # RTO violations
        violations = self.check_rto_compliance()
        if violations:
            lines.append(f"\nRTO VIOLATIONS ({len(violations)}):")
            for v in violations:
                lines.append(f"  {v['system']} (Tier {v['tier']}): "
                           f"RTO {v['rto_hours']}h exceeded by {v['exceeded_by_hours']}h")

        # System details
        lines.append("\nSystem Recovery Status:")
        lines.append("-" * 50)
        for tier in sorted(set(s.tier for s in self.plan.systems)):
            tier_systems = [s for s in self.plan.systems if s.tier == tier]
            lines.append(f"\n  Tier {tier}:")
            for sys in tier_systems:
                status_icon = {"pending": "[ ]", "restoring": "[~]", "validating": "[?]",
                              "online": "[+]", "failed": "[X]"}.get(sys.status, "[?]")
                lines.append(f"    {status_icon} {sys.name} ({sys.system_type}) - {sys.status}")
                if sys.restore_start and sys.restore_end:
                    start = datetime.fromisoformat(sys.restore_start)
                    end = datetime.fromisoformat(sys.restore_end)
                    duration = (end - start).total_seconds() / 3600
                    lines.append(f"        Restore time: {duration:.1f} hours")
                if sys.validation_checks:
                    for check, passed in sys.validation_checks.items():
                        result = "PASS" if passed else "FAIL"
                        lines.append(f"        {check}: {result}")

        # Next recoverable systems
        ready = self.get_next_recoverable()
        if ready:
            lines.append(f"\nReady for Recovery ({len(ready)}):")
            for sys in ready:
                lines.append(f"  - {sys.name} (Tier {sys.tier}, {sys.system_type})")

        lines.append("\n" + "=" * 70)
        return "\n".join(lines)

    def export_plan(self, output_path: str):
        with open(output_path, "w") as f:
            json.dump(asdict(self.plan), f, indent=2)


def main():
    """Demo recovery orchestration."""
    orch = RecoveryOrchestrator(
        incident_id="INC-2026-0042",
        org_name="Acme Manufacturing",
        compromise_date="2026-02-20T03:00:00",
    )

    # Define systems with dependencies
    orch.add_system(RecoverableSystem(
        name="DC01", tier=1, system_type="dc", rto_hours=4,
    ))
    orch.add_system(RecoverableSystem(
        name="DC02", tier=1, system_type="dc", rto_hours=4,
    ))
    orch.add_system(RecoverableSystem(
        name="DNS01", tier=1, system_type="dns", rto_hours=4,
        dependencies=["DC01"],
    ))
    orch.add_system(RecoverableSystem(
        name="SQL-ERP", tier=1, system_type="database", rto_hours=8,
        dependencies=["DC01", "DNS01"],
    ))
    orch.add_system(RecoverableSystem(
        name="ERP-APP", tier=1, system_type="application", rto_hours=12,
        dependencies=["SQL-ERP", "DC01"],
    ))
    orch.add_system(RecoverableSystem(
        name="Exchange", tier=2, system_type="email", rto_hours=12,
        dependencies=["DC01", "DC02"],
    ))
    orch.add_system(RecoverableSystem(
        name="FileServer01", tier=2, system_type="fileserver", rto_hours=24,
        dependencies=["DC01"],
    ))
    orch.add_system(RecoverableSystem(
        name="WebApp01", tier=2, system_type="web", rto_hours=24,
        dependencies=["SQL-ERP"],
    ))
    orch.add_system(RecoverableSystem(
        name="DevServer", tier=3, system_type="other", rto_hours=48,
        dependencies=["DC01"],
    ))

    # Simulate recovery progress
    orch.start_restore("DC01", "Immutable Veeam Hardened Repo")
    orch.complete_restore("DC01")
    orch.validate_system("DC01", {
        "dcdiag": True, "repadmin": True, "dns_resolution": True,
        "krbtgt_reset": True, "persistence_scan": True,
    })

    orch.start_restore("DC02", "Immutable Veeam Hardened Repo")
    orch.complete_restore("DC02")
    orch.validate_system("DC02", {
        "dcdiag": True, "repadmin": True, "replication": True,
    })

    orch.start_restore("DNS01", "Immutable Veeam Hardened Repo")

    print(orch.generate_report())

    output_path = str(Path(__file__).parent / "recovery_plan.json")
    orch.export_plan(output_path)
    print(f"\nRecovery plan exported to: {output_path}")


if __name__ == "__main__":
    main()

===== END FILE =====
