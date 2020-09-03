## /v2/info 返回的字段的含义
```
{
        "peer_version":385875968,
        "burn_consensus":"d8148c6e2cb1e7d74fdc3927174c8bb65d875b11",
        "burn_block_height":5303,
        "stable_burn_consensus":"afa4e73e0fc45874bf601d10241a935280b72e46",
        "stable_burn_block_height":5302,"server_version":"blockstack-core 0.0.1 =>
            23.0.0.0 (master:bdd042242+, release build, linux [x86_64])",
        "network_id":2147483648,"parent_network_id":3669344250,
        "stacks_tip_height":3769,
        "stacks_tip":"462d7f6c86fd53e33d66cef6bae324581154a072fb5d2ebbf04c4685ac1e219c",
        "stacks_tip_burn_block":"7396331c65f0a170d073a19a343bc3bab21417c6ae6cf253a65229cea53d27e1",
        "unanchored_tip":"17f34c5e0fae30be507c1294ab5a71bda5eca9300ecf5e3bf51484143cf70f28",
        "exit_at_block_height":28160
}
```

## transaction 的 fee_rate
```
{"limit":5,"offset":0,"total":4420,"results":           
    {
        "tx_id":"0xf770cf66b5db7293a0669f466c8bb64de0699955019bc625790fa90497805c52",
        "tx_type":"coinbase",
        "fee_rate":"0",
        "sender_address":"ST2TJRHDHMYBQ417HFB0BDX430TQA5PXRX6495G1V",
        "sponsored":false,
        "post_condition_mode":"deny",
        "tx_status":"success",
        "block_hash":"0xe129efd0105129e91686a619d66da9fd927a7f92536bc0ddf0362a3b05dd4628",
        "block_height":3771,
        "burn_block_time":1599054004,
        "burn_block_time_iso":"2020-09-02T13:40:04.000Z",
        "canonical":true,
        "tx_index":0,
        "tx_result":{"hex":"0x03","repr":"true"},
        "coinbase_payload":{"data":"0x0000000000000000000000000000000000000000000000000000000000000000"}
     },
     {"tx_id":"0xb962767ce955d9c85f3d77b3dc1fdf82ca347da613b0c6a760426bd4dfe938fd","tx_type":"coinbase","fee_rate":"0","sender_address":"ST2TJRHDHMYBQ417HFB0BDX430TQA5PXRX6495G1V","sponsored":false,"post_condition_mode":"deny","tx_status":"success","block_hash":"0xad4648557799586ceca04d9383959030e658f1b890eb2b14386b8a841c273148","block_height":3770,"burn_block_time":1599053764,"burn_block_time_iso":"2020-09-02T13:36:04.000Z","canonical":true,"tx_index":0,"tx_result":{"hex":"0x03","repr":"true"},"coinbase_payload":{"data":"0x0000000000000000000000000000000000000000000000000000000000000000"}},{"tx_id":"0x7fcf391d81a2fad2942057d15c7c0e054b8e6c0dab6f41915cc7203ec26393ed","tx_type":"contract_call","fee_rate":"1000","sender_address":"ST1DM1B8KFSFEJR3WZF3A8J4QDEGCGQKDJ0A65FKZ","sponsored":false,"post_condition_mode":"allow","tx_status":"success","block_hash":"0x462d7f6c86fd53e33d66cef6bae324581154a072fb5d2ebbf04c4685ac1e219c","block_height":3769,"burn_block_time":1599053524,"burn_block_time_iso":"2020-09-02T13:32:04.000Z","canonical":true,"tx_index":1,"tx_result":{"hex":"0x0100000000000000000000000000000001","repr":"u1"},"post_conditions":[],"contract_call":{"contract_id":"ST1DM1B8KFSFEJR3WZF3A8J4QDEGCGQKDJ0A65FKZ.hello_world","function_name":"set-value","function_signature":"(define-public (set-value (key (buff 32)) (value (buff 32))))","function_args":[{"hex":"0x0200000003666f6f","repr":"\"foo\"","name":"key","type":"(buff 32)"},{"hex":"0x0200000003626172","repr":"\"bar\"","name":"value","type":"(buff 32)"}]},"events":[]},{"tx_id":"0x43199d1c25017870c4ed797711c2c06885846375e95caa1c73d7536fda0582dd","tx_type":"coinbase","fee_rate":"0","sender_address":"ST2TJRHDHMYBQ417HFB0BDX430TQA5PXRX6495G1V","sponsored":false,"post_condition_mode":"deny","tx_status":"success","block_hash":"0x462d7f6c86fd53e33d66cef6bae324581154a072fb5d2ebbf04c4685ac1e219c","block_height":3769,"burn_block_time":1599053524,"burn_block_time_iso":"2020-09-02T13:32:04.000Z","canonical":true,"tx_index":0,"tx_result":{"hex":"0x03","repr":"true"},"coinbase_payload":{"data":"0x0000000000000000000000000000000000000000000000000000000000000000"}},{"tx_id":"0x27d7d175f52b0bdb3676710b2dd469f7bf9ba4bae7ed0e0753c8ee77304f5d87","tx_type":"coinbase","fee_rate":"0","sender_address":"ST2TJRHDHMYBQ417HFB0BDX430TQA5PXRX6495G1V","sponsored":false,"post_condition_mode":"deny","tx_status":"success","block_hash":"0x25074313754a1d4db485a8d4526216577012d8321d6bd33a835447a0db3e824d","block_height":3768,"burn_block_time":1599053284,"burn_block_time_iso":"2020-09-02T13:28:04.000Z","canonical":true,"tx_index":0,"tx_result":{"hex":"0x03","repr":"true"},"coinbase_payload":{"data":"0x0000000000000000000000000000000000000000000000000000000000000000"}}]}
```