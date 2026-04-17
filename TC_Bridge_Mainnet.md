# Nexus SDK V2 — Bridge Functionality Test Cases (Mainnet)

**App URL:** https://nexus-sdk-v2.vercel.app/bridge  
**Network:** Mainnet  
**Module:** Bridge  
**Last Updated:** 2026-04-17

---

## Prerequisites

- A supported wallet installed (MetaMask, Rabby, OKX, or Coinbase Wallet)
- Wallet funded on the source chain (e.g., ETH on Ethereum, MATIC on Polygon)
- Sufficient native gas token on the source chain to cover transaction fees
- Record wallet address and note starting balances before testing

---

## TC-BR-01 — Page Load & Default State Validation

**Objective:** Verify the Bridge page loads correctly and displays the expected default UI before wallet connection.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Navigate to https://nexus-sdk-v2.vercel.app/bridge | Page loads without errors |
| 2 | Observe the header | "Nexus V2" logo visible; **MAINNET** indicator shown in green |
| 3 | Observe the navigation tabs | Four tabs present: `Exact Out Swap`, `Swap & Execute`, `Bridge`, `Bridge & Execute` |
| 4 | Confirm active tab | `Bridge` tab is highlighted/selected |
| 5 | Observe the main content area | "Connect your wallet" placeholder is displayed with taglines: "Swap across chains", "Bridge in one click", "Deposit into DeFi" |
| 6 | Observe the header controls | `Switch to testnet`, `View balances`, Dark/Light mode toggle, and `Connect Wallet` button are all visible |

**Pass Criteria:** All UI elements render correctly; bridge form is gated behind wallet connection.

---

## TC-BR-02 — Wallet Connection (MetaMask)

**Objective:** Verify wallet connection flow works correctly and the bridge form unlocks after connection.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Click the `Connect Wallet` button in the top-right header | Wallet selection modal opens |
| 2 | Observe the modal options | Options shown: `Continue with Family`, `Rabby Wallet`, `OKX Wallet`, `MetaMask`, `Coinbase Wallet`, `I don't have a wallet` |
| 3 | Click `MetaMask` | MetaMask extension popup opens requesting connection approval |
| 4 | In MetaMask popup, select the desired account and click `Connect` | MetaMask popup closes |
| 5 | Observe the dApp header | `Connect Wallet` button is replaced by the connected wallet address (truncated, e.g., `0xABCD...1234`) |
| 6 | Observe the main content area | Bridge form is now visible (source chain, destination chain, token selector, amount input) |
| 7 | Verify the network indicator | `MAINNET` indicator remains green; no network mismatch warning shown |

**Pass Criteria:** Wallet connects successfully; bridge form is accessible; wallet address is displayed.

---

## TC-BR-03 — Source & Destination Chain Selection

**Objective:** Verify chain selectors work and cover supported mainnet chains.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Observe the bridge form after wallet connection | Source chain and destination chain dropdowns are visible |
| 2 | Click the **Source Chain** dropdown | A list of supported mainnet chains appears (e.g., Ethereum, Polygon, Arbitrum, Optimism, Avail) |
| 3 | Select a source chain (e.g., **Ethereum**) | Source chain updates to Ethereum; token list refreshes |
| 4 | Click the **Destination Chain** dropdown | A list of supported destination chains appears |
| 5 | Select a destination chain different from source (e.g., **Avail**) | Destination chain updates; route is calculated |
| 6 | Attempt to set source and destination to the same chain | App should prevent this or show a validation warning |
| 7 | Click the **Swap chains** (reverse) icon between the two selectors | Source and destination chains are swapped |

**Pass Criteria:** Chain selectors display all supported chains; same-chain selection is blocked; chain swap works.

---

## TC-BR-04 — Source Balance Check (Pre-Bridge)

**Objective:** Record and validate the source chain token balance before executing a bridge transaction.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Ensure wallet is connected and source chain is selected (e.g., Ethereum) | Bridge form is visible |
| 2 | Select the token to bridge (e.g., **USDC**) from the token dropdown | Token selected; current balance displayed next to the input field |
| 3 | Note the displayed source balance (e.g., `Balance: 50.00 USDC`) | Balance is a non-negative number |
| 4 | Cross-verify balance in MetaMask / wallet for the same token on the same chain | dApp balance matches wallet balance (±0.001 tolerance for rounding) |
| 5 | Click `View balances` button in the header (if applicable) | Balance overview panel shows all token balances across chains |

**Pre-Bridge Source Balance (record here):**  
`Token: ________ | Amount: ________ | Chain: ________`

**Pass Criteria:** Source balance displayed is accurate and matches the wallet.

---

## TC-BR-05 — Destination Balance Check (Pre-Bridge)

**Objective:** Record and validate the destination chain token balance before executing a bridge transaction.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Ensure destination chain is selected (e.g., **Avail**) | Bridge form shows destination chain |
| 2 | Observe any "Receive" or destination balance indicator in the form | Destination balance (if shown) is a non-negative number |
| 3 | Cross-verify balance on the destination chain using a block explorer or the destination wallet | Balance is confirmed independently |
| 4 | Note the destination balance for the bridged token | Recorded for post-bridge comparison |

**Pre-Bridge Destination Balance (record here):**  
`Token: ________ | Amount: ________ | Chain: ________`

**Pass Criteria:** Destination balance is accessible and verifiable before the transaction.

---

## TC-BR-06 — Token Selection & Amount Input

**Objective:** Verify token selection and amount entry behave correctly.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Click the token dropdown in the **From** section | Token list opens showing available tokens on the source chain |
| 2 | Search for a token (e.g., type "USDC") | Filtered results show USDC |
| 3 | Select **USDC** | Token selected; balance refreshes |
| 4 | Enter an amount less than the available balance (e.g., `1`) | Amount accepted; estimated receive amount and fees are shown |
| 5 | Enter `0` or a negative value | Validation error shown; Bridge button remains disabled |
| 6 | Enter an amount greater than the available balance | Insufficient balance error shown; Bridge button is disabled |
| 7 | Click `MAX` button (if present) | Input fills with the maximum bridgeable amount (balance minus gas reserve) |
| 8 | Clear the input field | Bridge button becomes disabled; quote disappears |

**Pass Criteria:** Token selection and amount validation work correctly; fee and receive estimates are shown for valid amounts.

---

## TC-BR-07 — Bridge Quote & Fee Validation

**Objective:** Verify that a bridge quote (estimated receive amount and fees) is shown before executing.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | With source chain, destination chain, token, and valid amount set | A route/quote section appears below the inputs |
| 2 | Observe the quote details | Estimated amount to receive on destination, bridge fee, gas estimate, and estimated time are all displayed |
| 3 | Note the quoted receive amount | Receive amount is less than sent amount (fees deducted) |
| 4 | Wait 30 seconds without interacting | Quote auto-refreshes or shows a "refresh quote" prompt |
| 5 | Click refresh quote (if available) | Updated quote is fetched; amounts may have changed slightly |

**Pass Criteria:** A clear quote is shown before bridging; fees are transparent; quote refreshes correctly.

---

## TC-BR-08 — Execute Bridge Transaction

**Objective:** Execute a real bridge transaction on mainnet and validate the wallet interaction flow.

> ⚠️ **Warning:** This test moves real funds on mainnet. Use a small amount (e.g., 1 USDC).

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Confirm source chain, destination chain, token (e.g., USDC), and amount (e.g., `1`) are set | Quote is visible |
| 2 | Click the **Bridge** button | If token approval is required, MetaMask opens an approval transaction |
| 3 | **[If approval needed]** Review the approval in MetaMask and click `Confirm` | Approval transaction submitted; spinner/progress shown on dApp |
| 4 | Wait for approval confirmation | dApp shows approval confirmed; Bridge button becomes active |
| 5 | Click **Bridge** (or the button advances automatically) | MetaMask opens the bridge transaction for signing |
| 6 | Review the transaction details in MetaMask (gas fee, recipient contract) | Details look correct; gas fee is within expected range |
| 7 | Click `Confirm` in MetaMask | Transaction submitted to the source chain |
| 8 | Observe the dApp status | Transaction hash is displayed; status shows `Pending` or `Processing` |
| 9 | Click the transaction hash link | Block explorer opens showing the transaction as pending/confirmed |
| 10 | Wait for source chain confirmation (1–3 minutes typical) | Source chain tx status shows `Success` |
| 11 | Wait for bridge relay and destination chain finalization (may take 5–30 minutes depending on chains) | dApp updates status to `Completed` or `Success` |

**Transaction Details (record here):**  
`Tx Hash: ________`  
`Source Chain Tx confirmed at block: ________`  
`Destination Chain Tx Hash (if shown): ________`

**Pass Criteria:** Transaction submitted successfully; source chain confirms; bridge relay completes without errors.

---

## TC-BR-09 — Source Balance Validation (Post-Bridge)

**Objective:** Confirm that the source chain balance decreased by the expected amount after the bridge transaction.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | After bridge transaction is confirmed on source chain, observe the source balance on the dApp | Balance has decreased |
| 2 | Calculate expected balance: Pre-bridge balance − Bridged amount − Gas fees | Expected balance is computed |
| 3 | Compare dApp displayed balance with expected balance | Difference ≤ 0.001 (rounding tolerance) |
| 4 | Cross-verify in MetaMask / wallet on source chain | Wallet balance matches dApp balance |
| 5 | Cross-verify on block explorer for source chain | On-chain balance confirms the deduction |

**Post-Bridge Source Balance (record here):**  
`Token: ________ | Amount: ________ | Chain: ________`  
`Difference from pre-bridge: ________` (should equal bridged amount + gas)

**Pass Criteria:** Source balance decreases by exactly the bridged amount plus gas costs.

---

## TC-BR-10 — Destination Balance Validation (Post-Bridge)

**Objective:** Confirm that the destination chain balance increased by the expected received amount.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | After dApp shows bridge status as `Completed`, observe destination balance on dApp | Balance has increased |
| 2 | Compare post-bridge destination balance with pre-bridge destination balance | Difference equals the quoted receive amount (±0.001 tolerance) |
| 3 | Cross-verify in the wallet connected to the destination chain | Wallet balance on destination chain matches dApp display |
| 4 | Cross-verify on the destination chain block explorer | On-chain balance confirms the receipt |
| 5 | Confirm the received amount matches the quoted amount from TC-BR-07 | Received amount ≥ quoted amount (or within acceptable slippage range) |

**Post-Bridge Destination Balance (record here):**  
`Token: ________ | Amount: ________ | Chain: ________`  
`Increase from pre-bridge: ________` (should equal quoted receive amount)

**Pass Criteria:** Destination balance increases by the quoted receive amount within the acceptable slippage tolerance.

---

## TC-BR-11 — Transaction History / Status Tracking

**Objective:** Verify that the dApp correctly tracks and displays the bridge transaction history.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | After completing a bridge, look for a transaction history or activity panel | Transaction appears in the list |
| 2 | Observe the transaction entry details | Shows: source chain, destination chain, token, amount, status, and timestamp |
| 3 | Click on the transaction entry | Opens detail view or block explorer link |
| 4 | Reload the page and reconnect wallet | Previous transaction still visible in history |

**Pass Criteria:** Transaction history accurately reflects completed bridge operations.

---

## TC-BR-12 — Wallet Disconnection

**Objective:** Verify the bridge form correctly reverts to the "Connect Wallet" state after disconnecting.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Click on the wallet address displayed in the header | Wallet menu or disconnect option appears |
| 2 | Click `Disconnect` | Wallet disconnected |
| 3 | Observe the dApp | `Connect Wallet` button reappears; bridge form is hidden; "Connect your wallet" placeholder shown |
| 4 | Refresh the page | Wallet remains disconnected; page loads in default state |

**Pass Criteria:** Disconnection works correctly; page reverts to unauthenticated state.

---

## TC-BR-13 — Dark Mode Toggle

**Objective:** Verify the dark/light mode toggle functions correctly and persists across navigation.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Click the moon/sun icon in the header | Theme switches between dark and light mode |
| 2 | Navigate to another tab (e.g., `Bridge & Execute`) and back | Theme preference is retained |
| 3 | Refresh the page | Theme preference persists (stored in local storage) |

**Pass Criteria:** Theme toggle works and persists.

---

## TC-BR-14 — Network Indicator (Mainnet Enforcement)

**Objective:** Verify the app correctly enforces and displays the Mainnet network.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Observe the `MAINNET` indicator (green dot) in the header | Green dot with "MAINNET" label is visible |
| 2 | Connect wallet on the wrong network (e.g., a testnet like Goerli) in MetaMask | dApp shows a "Wrong network" banner or prompts to switch to Mainnet |
| 3 | Switch MetaMask to Mainnet (Ethereum) | Warning disappears; bridge form is accessible |
| 4 | Click `Switch to testnet` button in the header | App switches to testnet mode and updates the network indicator accordingly |

**Pass Criteria:** Mainnet is enforced; wrong-network warnings are shown; testnet switch works.

---

## Test Execution Summary

| TC ID | Test Case | Status | Notes |
|-------|-----------|--------|-------|
| TC-BR-01 | Page Load & Default State | ☐ Pass / ☐ Fail | |
| TC-BR-02 | Wallet Connection | ☐ Pass / ☐ Fail | |
| TC-BR-03 | Chain Selection | ☐ Pass / ☐ Fail | |
| TC-BR-04 | Source Balance Check (Pre) | ☐ Pass / ☐ Fail | |
| TC-BR-05 | Destination Balance Check (Pre) | ☐ Pass / ☐ Fail | |
| TC-BR-06 | Token Selection & Amount Input | ☐ Pass / ☐ Fail | |
| TC-BR-07 | Bridge Quote & Fee Validation | ☐ Pass / ☐ Fail | |
| TC-BR-08 | Execute Bridge Transaction | ☐ Pass / ☐ Fail | |
| TC-BR-09 | Source Balance (Post-Bridge) | ☐ Pass / ☐ Fail | |
| TC-BR-10 | Destination Balance (Post-Bridge) | ☐ Pass / ☐ Fail | |
| TC-BR-11 | Transaction History / Status | ☐ Pass / ☐ Fail | |
| TC-BR-12 | Wallet Disconnection | ☐ Pass / ☐ Fail | |
| TC-BR-13 | Dark Mode Toggle | ☐ Pass / ☐ Fail | |
| TC-BR-14 | Network Indicator (Mainnet) | ☐ Pass / ☐ Fail | |

---

## Defect Reporting Template

```
Defect ID    : DEF-BR-XXX
TC Reference : TC-BR-XX
Step #       : X
Summary      : <one-line description>
Severity     : Critical / High / Medium / Low
Steps to Reproduce:
  1.
  2.
  3.
Expected Result :
Actual Result   :
Screenshot/Tx Hash :
Environment     : Browser: ___ | Wallet: ___ | Network: Mainnet
```
