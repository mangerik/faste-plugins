---
description: Generate a QRIS top-up payment link and verify credit settlement via Faste Media MCP
argument-hint: "[amount in IDR, e.g. 50000]"
---

Help the user top up their Faste Media credit balance via QRIS using the `faste-media` MCP server. Request context: `$ARGUMENTS`

Follow this exact safe workflow:
1. Call `get_account_balance()` first to inspect current `balance_available` (Credits & IDR at `1 Credit = Rp 100`) and check `key_scopes`.
2. Verify that `billing:write` is included in `key_scopes`:
   - If `billing:write` is **not** present (e.g., a restricted `agent` preset key), do **not** call `create_topup`. Explain clearly that the connected API key does not have `billing:write` permission and advise reconnecting with a `billing:write`-enabled key or topping up directly via the account owner.
3. Determine the desired top-up nominal in Indonesian Rupiah (`amount_idr`):
   - Valid range: minimum **Rp 25.000** (or server minimum), maximum **Rp 496.200** per QRIS transaction, and must be a multiple of **Rp 100** (`1 Credit = Rp 100`).
   - If `$ARGUMENTS` does not include an explicit amount—or if the user has not explicitly confirmed the exact nominal in this turn—state the current balance and ask the user to confirm the exact Rupiah nominal before calling `create_topup`.
4. Once explicitly confirmed, call `create_topup(amount_idr=<nominal>, idempotency_key="topup-<unique-slug>")`.
5. Present the response clearly:
   - Clickable **`payment_url`** (QRIS payment link)
   - `credits_granted`, `base_amount_idr`, `gateway_fee_idr`, `total_payable_idr`
   - `order_id` and `expires_at`
6. When the user says they have completed payment, call `get_topup_status(order_id="<order_id>")` (which auto-reconciles with the payment gateway) and `get_account_balance()` to confirm `status == "paid"` and the updated credit balance.
