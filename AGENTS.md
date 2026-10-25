# Project architecture

- Stock table inline edits use a shared conversion helper and blur-saving cell; serialize writes and apply grouped quantity deltas once to preserve stock consistency.
- Stock photos use private owner-scoped storage paths saved on every product in a grouped row, with signed display URLs; this keeps photos persistent across stock edits without exposing other users' uploads.
- Profit reports use the shared invoice calculator for totals, trends and details; purchase rates are per square foot, and new sale items snapshot the rate and cost quantity to prevent later stock-price edits changing their profit. Legacy costs and returns without original cost linkage remain explicitly estimated.
- Saved invoice previews resolve included previous balances through the shared invoice-due helper before falling back to the customer balance, because saved invoice due can already include previous debt.