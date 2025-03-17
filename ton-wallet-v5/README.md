# Ton Wallet v5

## user interaction flow

an example msg for sending jettons, sent from user to the wallet
```
[prefix::signed_external (0x7369676E)]
[wallet_id (32 bits)]
[valid_until (32 bits)]
[seqno (32 bits)]
=> (outgoing action cell, which will be stored in c5 register) [
  [action_send_msg prefix (0x0ec3c86d)]
  [send_mode (8 bits)]
  => (full msg cell) [
    [info bits + ihr_disabled + bounce + bounced flags]
    [src address (empty or wallet address)]
    [dst address (user's jetton wallet address)]
    [amount (coins to send with message)]
    => (msg body cell) [
      [op code for transfer (0xf8a7ea5)]
      [query_id (64 bits)]
      [jetton_amount (coins amount to transfer)]
      [destination address (recipient's jetton wallet)]
      [response_destination address (usually sender's address)]
      [forward_ton_amount (coins to attach to notification)]
      [forward_payload (optional data, can be empty)]
    ]
  ]
  [next_action (empty for single action)]
]
[has_other_actions flag (1 bit) = 0]
[signature (512 bits)]
```

- user created a msg with `prefix::signed_external` prefix, similar to the example above
- `recv_external()`: ensures it's has `prefix::signed_external` prefix
- `process_signed_request()`:
  - check signature and wallet metadatas (seqno, wallet_id, public_key)
  - `accept_message()` accept gas payment
  - update state: stored_seqno += 1
  - `commit()` (mark tx as success and persist state changes)
  - `process_actions()`
    - `verify_c5_actions()`: verify msg has `action_send_msg` prefixes
    - put action reference to c5 register (c5 is the register for storing actions)
    - loop through extensions, and apply each action
    - contract execution ends, TVM processes all actions in C5 automatically