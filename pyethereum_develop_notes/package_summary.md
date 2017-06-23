### ethereum.config

default configuration of a chain and hardcoded config changes of different hardfork versions
[TODO, list configs]
#### class Env(object)
* `__init__(self, db=None, config=None, global_config=None)` - creates an env object including settings like database, configuration and global\_configuration
    * global\_config: [TODO, what is a global config]


### ethereum.common

* `calc_gaslimit(parent, config=default_config)` - calculate gas limit for the new block
* `calc_difficulty(parent, timestamp, config=default_config)` - calculate difficulty for the new block
* `check_gaslimit(parent, gas_limit, config=default_config)` - checks if 
    1. gas limit adjustment between parent/child block does not exceed GASLIMIT\_ADJMAX\_FACTOR
    2. gas limit is above MIN\_GAS\_LIMIT
* `validate_header(state, header)` - checks if current/parent block headers pass the following check
    1. hash mismatch
    2. block number not in a sequence
    3. gas limit validity
    4. block difficulty validity
    5. gas usage < gas limit
    6. extra data validity
    7. timestamp validity




### ethereum.genesis_helpers

creates genesis block/state from given genesis data

* `mk_basic_state(alloc, header=None, env=None)` - creates a basic state or a genesis state if not header provided
    1. previous header: assumes to be a genesis block header if there's no header data provided
    2. set account balance, code ,nonce and storage according to alloc

* `mk_genesis_block(env, **kwargs)` - create a genesis block



### tools.tester

creates a custom chain for testing

* class Chain(alloc, env): create a test chain with ether allocations and configuration if specified
    * direct_tx(transaction): apply a transaction in RLP format and commit
    * tx(sender, to, value, data, startgas, gasprice): apply a transaction with different specified inputs
        * put together inputs and apply the tx with direct_tx
        * NOTE: **sender** is the private key of sender
    * contract(sourcecode, args, sender, value, language, startgas, gasprice): compile, deploy the contract and return a contract object
        * NOTE: **sender** is the private key of sender
    * mine(number\_of\_blocks, coinbase): mine specified number of blocks
    * snapshot: copy the current state of the block
    * revert(snapshot): revert to the specified state of snapshot
        * NOTE: only works on states in the same block, i.e., can not revert between blocks

* NOTE: the `head_state` attribute is a copy of latest(head) state of the chain, like this - `self.head_state = self.chain.state.ephemeral_clone()`, so if you want to attach a event watcher using `log_listeners`, you should attach it using `self.chain.state` instead of `self.head_state`, for example:
```
c = tester.Chain()
c.chain.state.log_listeners.append(lambda logs: print(logs))
# trigger log...
c.mine() # also need to mine to update self.chain.state
```