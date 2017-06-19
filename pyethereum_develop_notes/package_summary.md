### config
contains default configuration of a chain

* class Env(db, config, global\_config): creates an env object including settings like db, config and global\_config
    * global\_config: 
* also hardcoded config of different hardfork versions here

___

### common

* def validate_header(state, header): checks if current/parent block headers pass the following check
    1. hash mismatch
    2. block number not in a sequence
    3. gas limit validity
    4. block difficulty validity
    5. gas usage < gas limit
    6. extra data validity
    7. timestamp validity

* def check_gaslimit(parent, gas\_limit, config): checks if 
    1. gas limit adjustment between parent/child block does not exceed GASLIMIT\_ADJMAX\_FACTOR
    2. gas limit is above MIN\_GAS\_LIMIT

___

### genesis_helpers

* mk\_basic\_state(alloc, header, env): creates and sets a state object accordingly
    1. previous header: assumes to be a genesis block header if there's no header data provided
    2. set account balance, code ,nonce and storage according to alloc

___

### utils
1. 

___

### tools.tester
1. 