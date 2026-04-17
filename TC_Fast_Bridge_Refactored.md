# Nexus V1 — Fast Bridge Test Checklist (Refactored)

**Source:** Nexus_V1_QAChecklist.xlsx → Fast Bridge sheet  
**Total Test Cases:** 50  
**Last Updated:** 2026-04-17

---

## Legend

| Priority | Meaning |
|----------|---------|
| P0 | Blocker — must pass before release |
| P1 | Critical — core user flow |
| P2 | Important — edge case / error handling |

| Type | Meaning |
|------|---------|
| UI | Visual / rendering verification |
| Functional | Feature behavior verification |
| E2E | End-to-end on-chain transaction |
| Error | Error state and failure handling |

---

## Preconditions (All Sections)

- Wallet is installed (MetaMask / Rabby / OKX / Coinbase)
- Wallet is connected to the dApp on **Mainnet**
- Sufficient token balance and native gas on source chain(s)
- Note all starting balances before executing any E2E test

---

## Section 1 — Rendering & Layout

> Verify the Fast Bridge widget renders correctly and all core UI elements are present on load.

| # | Test Case | Priority | Type | Steps | Expected Result |
|---|-----------|----------|------|-------|-----------------|
| 1 | Widget renders without errors on load | P0 | UI | 1. Navigate to the Fast Bridge page<br>2. Observe the browser console | Widget loads fully; no JS errors or blank states in the console |
| 2 | Source chain and destination chain selectors are visible | P0 | UI | 1. Load the Fast Bridge widget<br>2. Inspect the form layout | Both source chain and destination chain dropdown selectors are rendered and labeled |
| 3 | Token selector shows available bridgeable tokens | P0 | UI | 1. Select a source chain<br>2. Open the token selector dropdown | Only tokens bridgeable from the selected source chain are listed; no empty or broken state |
| 4 | Fee details section is displayed before confirmation | P1 | UI | 1. Select chains and token<br>2. Enter a valid amount<br>3. Observe the form before clicking Bridge | Fee breakdown section is visible below the amount input, showing estimated bridge fee and gas |

---

## Section 2 — Bridge Flow

> Verify the core interaction flow — chain selection, amount input, balance display, and ERC-20 approval — works end to end.

| # | Test Case | Priority | Type | Steps | Expected Result |
|---|-----------|----------|------|-------|-----------------|
| 5 | User can select source chain and token | P0 | Functional | 1. Click the source chain dropdown<br>2. Select a chain (e.g., Ethereum)<br>3. Open token selector<br>4. Select a token (e.g., USDC) | Source chain and token are set; form reflects the selection; balance loads |
| 6 | User can select destination chain | P0 | Functional | 1. Click the destination chain dropdown<br>2. Select a chain different from source (e.g., Arbitrum) | Destination chain updates; routing quote is triggered; source and destination cannot be identical |
| 7 | Amount input accepts valid numeric values | P0 | Functional | 1. Click the amount input field<br>2. Type `1.5`<br>3. Observe the form | Input accepts the value; quoted receive amount and fees update in real time |
| 8 | Source balance breakdown is shown accurately | P1 | Functional | 1. Connect wallet<br>2. Select source chain and token<br>3. Observe the balance display | Displayed balance matches the wallet balance for that token on that chain (±0.001 tolerance) |
| 9 | Allowance flow triggers ERC-20 approval when needed | P0 | Functional | 1. Select an ERC-20 token (e.g., USDC) with zero current allowance<br>2. Enter an amount<br>3. Click Bridge | Wallet popup requests ERC-20 approval for the bridge contract before the bridge transaction |
| 10 | Approval transaction completes before bridge transaction | P0 | Functional | 1. Confirm the ERC-20 approval in the wallet<br>2. Wait for approval to confirm on-chain<br>3. Observe the bridge button state | Bridge transaction is only triggered after the approval transaction is confirmed; no simultaneous submissions |

---

## Section 3 — Progress & Status

> Verify the step-by-step progress UI correctly tracks and displays each phase of the bridge execution.

| # | Test Case | Priority | Type | Steps | Expected Result |
|---|-----------|----------|------|-------|-----------------|
| 11 | Progress steps are shown during bridge execution | P0 | Functional | 1. Submit a bridge transaction<br>2. Observe the UI immediately after submission | A progress tracker with discrete steps (e.g., Approving → Submitting → Routing → Complete) is displayed |
| 12 | Each step transitions correctly (pending → active → done) | P1 | Functional | 1. Submit a bridge transaction<br>2. Watch each step in the progress tracker | Steps move from `Pending` to `Active` to `Done` in sequence; no step is skipped or stuck indefinitely |
| 13 | Intent-based routing completes successfully | P0 | E2E | 1. Submit a bridge with a valid route<br>2. Wait for full settlement | The intent is picked up by a solver; bridge settles on the destination chain; status shows `Completed` |
| 14 | Success state shows explorer link for the bridge transaction | P1 | Functional | 1. Wait for bridge to reach `Completed` state<br>2. Observe the success screen | A block explorer link is shown for the source chain transaction; clicking it opens the correct tx on the explorer |

---

## Section 4 — Fees & Edge Cases

> Verify fee display accuracy and graceful handling of error conditions.

| # | Test Case | Priority | Type | Steps | Expected Result |
|---|-----------|----------|------|-------|-----------------|
| 15 | Fee breakdown is shown accurately before confirmation | P1 | Functional | 1. Select chains, token, and enter a valid amount<br>2. Observe the fee section before submitting | Itemized fees are displayed (bridge fee + gas estimate); total cost is clearly shown |
| 16 | Bridge fails gracefully when RPC is unavailable | P1 | Error | 1. Simulate RPC failure (disable network or use a bad RPC URL)<br>2. Attempt to initiate a bridge | An error message is displayed (e.g., "Network unavailable" or "RPC error"); no partial transaction is submitted |
| 17 | Insufficient balance is communicated to the user | P0 | Error | 1. Enter an amount greater than the available source balance<br>2. Observe the UI | Bridge button is disabled; an "Insufficient balance" error is shown inline; no wallet popup is triggered |
| 18 | Switching source/destination chain resets the form cleanly | P1 | Functional | 1. Fill in all fields (chain, token, amount)<br>2. Change the source chain to a different chain<br>3. Observe the form state | Token selector resets to a valid token for the new chain; amount clears or revalidates; quote refreshes; no stale data shown |

---

## Section 5 — 1-to-1 Chain Routing (Single Source → Single Destination)

> Verify that bridging from one chain to one destination works correctly across all key scenarios.

**Preconditions:** Wallet funded on the source chain with the token being tested.

| # | Test Case | Priority | Type | Steps | Expected Result |
|---|-----------|----------|------|-------|-----------------|
| 19 | Bridge USDC: Ethereum → Arbitrum — destination balance credited | P0 | E2E | 1. Select Ethereum as source, Arbitrum as destination<br>2. Select USDC, enter amount (e.g., 1 USDC)<br>3. Submit and wait for settlement | Destination USDC balance increases by the quoted receive amount; source balance decreases accordingly |
| 20 | Bridge ETH: Optimism → Base — correct destination token amount | P0 | E2E | 1. Select Optimism as source, Base as destination<br>2. Select ETH, enter amount<br>3. Submit and confirm | ETH (or wrapped ETH) lands on Base in the quoted amount; no token mismatch |
| 21 | Bridge WBTC: Polygon → Avalanche — accurate fee estimate before confirmation | P1 | E2E | 1. Select Polygon as source, Avalanche as destination<br>2. Select WBTC, enter amount<br>3. Review fee estimate pre-submission | Fee estimate shown matches (or is within 5% of) the actual fee deducted post-transaction |
| 22 | Source chain token balance decreases by bridged amount after confirmation | P0 | E2E | 1. Record source balance pre-bridge<br>2. Submit bridge for any token/chain pair<br>3. Wait for source chain confirmation | Source balance = pre-bridge balance − bridged amount − gas (±0.001 tolerance) |
| 23 | Destination chain token balance increases correctly after settlement | P0 | E2E | 1. Record destination balance pre-bridge<br>2. Complete a bridge<br>3. Wait for full settlement | Destination balance = pre-bridge balance + quoted receive amount (±slippage tolerance) |
| 24 | Bridge with exact maximum available balance succeeds without overflow | P1 | E2E | 1. Click the MAX button to fill the input<br>2. Submit the bridge<br>3. Observe for errors | Transaction submits successfully; no overflow, dust, or "exceeds balance" error |
| 25 | Bridge with minimum allowed amount is processed correctly | P1 | E2E | 1. Enter the minimum bridgeable amount for the selected token<br>2. Submit the bridge | Transaction is accepted; amount arrives on destination; no "below minimum" rejection |
| 26 | Round-trip bridge (Chain A → Chain B → Chain A) maintains expected net balance | P2 | E2E | 1. Bridge token from Chain A to Chain B<br>2. Wait for full settlement<br>3. Bridge the same token back from Chain B to Chain A<br>4. Compare final balance to starting balance | Final balance on Chain A = original balance − total fees from both legs |
| 27 | Bridge between chains with different finality times displays accurate ETA | P1 | Functional | 1. Select a chain pair with known different finality (e.g., Ethereum → Optimism)<br>2. Observe the ETA shown in the progress/quote UI | ETA is displayed and reflects the slower chain's finality; not zero or placeholder |
| 28 | Bridge using recently approved token does not re-prompt ERC-20 approval | P1 | Functional | 1. Complete a bridge for an ERC-20 token (approval already granted)<br>2. Initiate a second bridge for the same token (same or lesser amount) | Wallet skips the approval step; goes directly to the bridge transaction |

---

## Section 6 — 2-to-1 Chain Routing (Two Sources → Single Destination)

> Verify that aggregated bridging from two source chains to one destination works correctly.

**Preconditions:** Wallet funded on both source chains for the selected token.

| # | Test Case | Priority | Type | Steps | Expected Result |
|---|-----------|----------|------|-------|-----------------|
| 29 | Aggregated bridge from Ethereum + Arbitrum → Optimism splits contributions correctly | P0 | E2E | 1. Select Ethereum and Arbitrum as two sources; Optimism as destination<br>2. Enter total amount<br>3. Submit | UI shows the per-chain split; both source chains submit transactions; amounts sum correctly |
| 30 | Total amount on destination equals sum of both source chain contributions | P0 | E2E | 1. Record both source balances<br>2. Complete the 2-to-1 bridge<br>3. Check destination balance post-settlement | Destination received = sum of both chain contributions − bridge fees |
| 31 | Fee breakdown itemizes gas costs per source chain separately | P1 | Functional | 1. Set up a 2-source bridge<br>2. Observe the fee confirmation screen | Fee section shows a line item for gas on each source chain independently |
| 32 | Progress UI shows parallel "Collecting on Source" steps for both chains | P1 | Functional | 1. Submit a 2-to-1 bridge<br>2. Observe the progress tracker during execution | Two "Collecting" steps are shown simultaneously (or sequentially labelled by chain) with individual statuses |
| 33 | One source chain with insufficient balance falls back to full amount from other chain | P1 | Error | 1. Set source 1 to have less than the required split amount<br>2. Set source 2 to have the full amount<br>3. Submit | Widget re-routes all funds from source 2; bridge completes; no error requiring user action |
| 34 | ERC-20 approval triggered independently and correctly for each source chain | P0 | Functional | 1. Use an ERC-20 token with zero allowance on both source chains<br>2. Submit the bridge | Two separate approval prompts appear, one per chain; both must confirm before bridge proceeds |
| 35 | RPC failure on one source chain does not submit the other chain | P1 | Error | 1. Simulate RPC failure on one source chain<br>2. Initiate the 2-to-1 bridge | Error is shown for the failing chain; the functioning chain's transaction is NOT submitted; user is informed |
| 36 | Switching one source chain re-computes the split and updates fee estimate | P1 | Functional | 1. Configure a 2-source bridge<br>2. Change one source chain to a different chain | Split percentages recalculate; fee estimate updates for the new chain combination; form is not stale |
| 37 | Success screen shows individual explorer links for both source chain transactions | P1 | Functional | 1. Complete a 2-to-1 bridge<br>2. Observe the success/completion screen | Two separate block explorer links are shown — one per source chain transaction |
| 38 | Cancelling mid-flow after approving one source chain does not submit the other | P0 | Error | 1. Approve ERC-20 on source chain 1<br>2. Cancel/close the flow before approving source chain 2<br>3. Observe on-chain state | Source chain 2 transaction is not submitted; only the approval on chain 1 was submitted (which is safe) |

---

## Section 7 — 3+-to-1 Chain Routing (Three or More Sources → Single Destination)

> Verify that multi-source aggregated bridging (3+ chains) works correctly, handles partial failures, and scales gracefully.

**Preconditions:** Wallet funded on all source chains being used. Record all source and destination balances before testing.

| # | Test Case | Priority | Type | Steps | Expected Result |
|---|-----------|----------|------|-------|-----------------|
| 39 | Aggregated bridge from 3 chains (Ethereum + Arbitrum + Optimism) → Base completes | P0 | E2E | 1. Select 3 source chains; Base as destination<br>2. Enter total amount<br>3. Submit and wait for settlement | All 3 source transactions confirm; amount lands on Base as the aggregate total |
| 40 | Aggregated bridge from 5 source chains → single destination completes without error | P1 | E2E | 1. Select 5 source chains<br>2. Enter the total bridge amount<br>3. Submit | All 5 source chain transactions succeed; destination balance is credited correctly |
| 41 | Correct total token amount arrives on destination after multi-source aggregation | P0 | E2E | 1. Record per-chain send amounts<br>2. Complete multi-source bridge<br>3. Check destination balance | Received = sum of all source contributions − total bridge fees (±slippage tolerance) |
| 42 | All source chain fees are individually itemized in the fee breakdown screen | P1 | Functional | 1. Configure 3+ source chains<br>2. Open the fee breakdown / confirmation screen | A line item per source chain is shown for gas; no chains are lumped into a single unlabelled fee |
| 43 | Progress UI shows collection steps for all source chains (sequential or parallel) | P1 | Functional | 1. Submit a 3+-to-1 bridge<br>2. Watch the progress tracker | A "Collecting" step appears for each source chain; all are labelled; overall progress is visible |
| 44 | Single source chain failure does not block collection from remaining chains | P1 | Error | 1. Configure 3+ sources<br>2. Simulate failure on 1 source chain (e.g., bad RPC)<br>3. Submit | Remaining source chains proceed and contribute to the destination; failed chain is surfaced as an error; bridge is not entirely blocked |
| 45 | Partial fill: some source chains confirm and partial amount lands on destination | P1 | E2E | 1. Configure 3+ sources where 1–2 chains fail<br>2. Submit the bridge<br>3. Observe destination balance | Partial amount from the confirming source chains lands on destination; UI clearly shows partial fill state |
| 46 | ERC-20 approvals are handled for each source chain independently before submission | P0 | Functional | 1. Use an ERC-20 token with no existing allowance on all source chains<br>2. Submit the bridge | An approval transaction is prompted per source chain; all must confirm before the bridge proceeds |
| 47 | Dust-level contribution from one chain is either excluded or handled gracefully | P2 | Error | 1. Configure one source chain with a dust-level balance (e.g., 0.000001 USDC)<br>2. Submit the bridge | Dust contribution is either excluded from routing automatically, or a clear warning is shown; no transaction failure due to dust |
| 48 | Success state shows per-chain amounts, aggregate total, and all source explorer links | P1 | Functional | 1. Complete a 3+-to-1 bridge<br>2. Observe the success/completion screen | Success screen lists: each source chain's contribution, total received on destination, and individual explorer links per source chain |
| 49 | Re-routing triggers automatically if one source chain becomes congested mid-flow | P2 | Functional | 1. Configure 3+ sources<br>2. Simulate high congestion on one source chain mid-execution (if testable)<br>3. Observe routing behavior | Bridge automatically deprioritizes or re-routes away from the congested chain; no manual intervention required |
| 50 | Gas spike on one source chain causes it to be de-prioritized in automatic routing | P2 | Functional | 1. Configure 3+ sources with one chain having significantly higher gas<br>2. Observe routing order before submission | High-gas chain is ranked lower or excluded from routing; fee estimate reflects the optimized route |

---

## Test Execution Summary

| # | Test Case | Priority | Type | Status | Notes | Issue |
|---|-----------|----------|------|--------|-------|-------|
| 1 | Widget renders without errors on load | P0 | UI | ☐ Not Started | | |
| 2 | Source and destination chain selectors visible | P0 | UI | ☐ Not Started | | |
| 3 | Token selector shows bridgeable tokens | P0 | UI | ☐ Not Started | | |
| 4 | Fee details shown before confirmation | P1 | UI | ☐ Not Started | | |
| 5 | User can select source chain and token | P0 | Functional | ☐ Not Started | | |
| 6 | User can select destination chain | P0 | Functional | ☐ Not Started | | |
| 7 | Amount input accepts valid numeric values | P0 | Functional | ☐ Not Started | | |
| 8 | Source balance breakdown shown accurately | P1 | Functional | ☐ Not Started | | |
| 9 | ERC-20 approval triggers when needed | P0 | Functional | ☐ Not Started | | |
| 10 | Approval completes before bridge transaction | P0 | Functional | ☐ Not Started | | |
| 11 | Progress steps shown during execution | P0 | Functional | ☐ Not Started | | |
| 12 | Steps transition correctly (pending→active→done) | P1 | Functional | ☐ Not Started | | |
| 13 | Intent-based routing completes successfully | P0 | E2E | ☐ Not Started | | |
| 14 | Success state shows explorer link | P1 | Functional | ☐ Not Started | | |
| 15 | Fee breakdown accurate before confirmation | P1 | Functional | ☐ Not Started | | |
| 16 | Bridge fails gracefully when RPC unavailable | P1 | Error | ☐ Not Started | | |
| 17 | Insufficient balance communicated to user | P0 | Error | ☐ Not Started | | |
| 18 | Switching chain resets form cleanly | P1 | Functional | ☐ Not Started | | |
| 19 | Bridge USDC: Ethereum → Arbitrum | P0 | E2E | ☐ Not Started | | |
| 20 | Bridge ETH: Optimism → Base | P0 | E2E | ☐ Not Started | | |
| 21 | Bridge WBTC: Polygon → Avalanche — fee estimate | P1 | E2E | ☐ Not Started | | |
| 22 | Source balance decreases by bridged amount | P0 | E2E | ☐ Not Started | | |
| 23 | Destination balance increases after settlement | P0 | E2E | ☐ Not Started | | |
| 24 | MAX balance bridge succeeds without overflow | P1 | E2E | ☐ Not Started | | |
| 25 | Minimum amount bridge processed correctly | P1 | E2E | ☐ Not Started | | |
| 26 | Round-trip bridge maintains expected net balance | P2 | E2E | ☐ Not Started | | |
| 27 | Different finality chains display accurate ETA | P1 | Functional | ☐ Not Started | | |
| 28 | Recent approval not re-prompted on second bridge | P1 | Functional | ☐ Not Started | | |
| 29 | 2-source bridge: Ethereum + Arbitrum → Optimism | P0 | E2E | ☐ Not Started | | |
| 30 | Destination total = sum of both source contributions | P0 | E2E | ☐ Not Started | | |
| 31 | Fee breakdown itemizes gas per source chain | P1 | Functional | ☐ Not Started | | |
| 32 | Progress UI shows parallel collection steps | P1 | Functional | ☐ Not Started | | |
| 33 | Insufficient source 1 falls back to source 2 | P1 | Error | ☐ Not Started | | |
| 34 | ERC-20 approval triggered per source chain | P0 | Functional | ☐ Not Started | | |
| 35 | RPC failure on one chain doesn't submit other | P1 | Error | ☐ Not Started | | |
| 36 | Switching one source re-computes split + fees | P1 | Functional | ☐ Not Started | | |
| 37 | Success screen shows explorer links per source | P1 | Functional | ☐ Not Started | | |
| 38 | Cancel mid-flow doesn't submit unapproved chain | P0 | Error | ☐ Not Started | | |
| 39 | 3-chain aggregate bridge → Base completes | P0 | E2E | ☐ Not Started | | |
| 40 | 5-chain aggregate bridge completes without error | P1 | E2E | ☐ Not Started | | |
| 41 | Correct total arrives after multi-source bridge | P0 | E2E | ☐ Not Started | | |
| 42 | All source chain fees individually itemized | P1 | Functional | ☐ Not Started | | |
| 43 | Progress UI shows steps for all source chains | P1 | Functional | ☐ Not Started | | |
| 44 | Single chain failure doesn't block others | P1 | Error | ☐ Not Started | | |
| 45 | Partial fill: partial amount lands on destination | P1 | E2E | ☐ Not Started | | |
| 46 | ERC-20 approvals handled per chain independently | P0 | Functional | ☐ Not Started | | |
| 47 | Dust contribution excluded or handled gracefully | P2 | Error | ☐ Not Started | | |
| 48 | Success shows per-chain amounts + all explorer links | P1 | Functional | ☐ Not Started | | |
| 49 | Re-routing triggers on mid-flow congestion | P2 | Functional | ☐ Not Started | | |
| 50 | Gas spike de-prioritizes chain in auto-routing | P2 | Functional | ☐ Not Started | | |

---

## Defect Report Template

```
Defect ID       : DEF-FB-XXX
TC Reference    : TC #XX
Step #          : X
Summary         : <one-line description>
Priority        : P0 / P1 / P2
Severity        : Critical / High / Medium / Low
Steps to Reproduce:
  1.
  2.
  3.
Expected Result :
Actual Result   :
Tx Hash (if E2E):
Screenshot      :
Environment     : Browser: ___ | Wallet: ___ | Network: Mainnet | Version: ___
```
