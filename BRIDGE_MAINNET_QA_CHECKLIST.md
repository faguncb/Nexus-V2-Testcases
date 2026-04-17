# Nexus Bridge Mainnet QA Checklist

URL under test: [Nexus Bridge dApp](https://nexus-sdk-v2.vercel.app/bridge)

| Step | Test Area | Action (Mainnet Only) | Expected Result | Pass/Fail | Notes |
|---|---|---|---|---|---|
| 1 | Environment Setup | Open [Nexus Bridge dApp](https://nexus-sdk-v2.vercel.app/bridge). | Page loads without error; `Bridge` tab is active. | ☐ | |
| 2 | Mainnet Confirmation | Verify top network indicator shows `MAINNET` (not testnet). | Mainnet badge/state is visible and stable after refresh. | ☐ | |
| 3 | Wallet Prereq | Click `Connect Wallet` and connect a funded mainnet wallet. | Wallet connects successfully; address appears in UI. | ☐ | |
| 4 | Token & Chain Controls | Check source chain, destination chain, token selectors, amount input, and bridge CTA are visible/enabled as expected after wallet connect. | All bridge form elements render correctly with no overlap/disabled state (unless valid reason). | ☐ | |
| 5 | Source Chain Selection | Select source chain as `Ethereum Mainnet` (or intended mainnet source). | Selector updates correctly; no console/UI errors. | ☐ | |
| 6 | Destination Chain Selection | Select a supported destination chain on mainnet route. | Destination updates; unsupported pair combinations are blocked with clear message. | ☐ | |
| 7 | Token Selection | Pick a supported token for source chain. | Token list opens, token is selectable, symbol/icon updates in form. | ☐ | |
| 8 | Balance Visibility | Validate wallet balance for selected token appears. | Accurate available balance is shown; `Max` (if present) works. | ☐ | |
| 9 | Amount Validation (Empty) | Keep amount empty and inspect CTA. | Bridge CTA remains disabled or prompts for required amount. | ☐ | |
| 10 | Amount Validation (Zero/Invalid) | Enter `0`, negative, or non-numeric input. | Input validation prevents invalid values; clear error text appears. | ☐ | |
| 11 | Amount Validation (Exceeds Balance) | Enter amount greater than wallet balance. | CTA blocked; proper "insufficient balance" style validation shown. | ☐ | |
| 12 | Quote/Route Fetch | Enter valid amount within balance. | Fee/estimate/received amount/route details load successfully. | ☐ | |
| 13 | Quote Refresh Behavior | Change token, chain, or amount after quote loaded. | Quote refreshes correctly; stale quote is replaced. | ☐ | |
| 14 | Approval Flow (if ERC-20) | Click `Approve` when required and confirm in wallet. | Approval txn submits and confirms; UI progresses to bridge step. | ☐ | |
| 15 | Approval Rejection Handling | Reject approval in wallet. | UI shows non-breaking error; user can retry approval. | ☐ | |
| 16 | Bridge Submit | Click bridge CTA and confirm transaction in wallet. | Bridge txn submits; tx hash or tracking state appears. | ☐ | |
| 17 | Wallet Rejection Handling | Reject bridge tx in wallet. | Graceful error shown; no stuck loading; CTA recoverable. | ☐ | |
| 18 | Pending State UX | During pending tx, observe status/progress UI. | Clear pending status shown; duplicate submit is prevented. | ☐ | |
| 19 | Success State | Wait for completion/confirmed state. | Success message/state appears with explorer/tracking link(s). | ☐ | |
| 20 | Destination Receipt | Verify bridged funds/token on destination chain wallet after completion. | Received amount matches expected (minus documented fees). | ☐ | |
| 21 | Fee Accuracy Check | Compare displayed fee vs actual tx + bridge deductions. | Variance is explainable and within expected tolerance. | ☐ | |
| 22 | Navigation Stability | Switch between app tabs (`Exact Out Swap`, `Swap & Execute`, `Bridge`, `Bridge & Execute`) and return to `Bridge`. | No form corruption/crash; current bridge state is preserved or reset intentionally. | ☐ | |
| 23 | Link Integrity | Click all bridge-related links (explorer/help/docs) shown in flow. | No broken links, no 404, and correct destinations open. | ☐ | |
| 24 | Refresh/Recovery | Refresh during idle and (separately) after completed bridge. | App recovers safely; completed transactions remain traceable. | ☐ | |
| 25 | Console/Network Health | Check browser console and failed network calls during complete flow. | No critical JS errors; failed calls (if any) are retried/handled gracefully. | ☐ | |
| 26 | Regression Smoke | Repeat a second bridge with different supported pair/token on mainnet. | Second run succeeds; no state leakage from previous run. | ☐ | |
