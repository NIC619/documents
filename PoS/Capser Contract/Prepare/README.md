## Prepare Rules
___
1. a prepare needs to *link to/based upon* a previous **justified** prepare, by justified it means that more than 2/3 of validators submit the same prepare(see [PREPARE_REQ in Minimal Slashing Condition](https://medium.com/@VitalikButerin/minimal-slashing-conditions-20f0b500fc6c)).
2. so in each prepare, there are messages of **current epoch**(meaning messages you are preparing in the moment), and messages of **source epoch**(meaning messages of the previous justified prepare). Messages of these two kind both include **hash** and **ancestry hash**(Note that ancestry hash is not just hash in a prepare in previous epoch, it's calculated by *sha3(hash + ancestry hash of previous epoch)*), so in each prepare, there are
    * current epoch
    * hash in current epoch
    * ancestry hash in current epoch
    * source epoch
    * hash in source epoch
    * ancestry hash in source epoch
3. a table of links between ancestry hashes are maintained, example:
    * a prepare in epoch 3 will include a hash and the ancestry hash of epoch 2, and ancestry hash of epoch 3 is calculated by *sha3(hash + ancestry hash of epoch 2)*, then a link between ancestry of epoch 2 and ancestry hash of epoch 3 is created
4. justified hash in each epoch is *sha3(current epoch + hash in current epoch + ancestry hash in current epoch + source epoch + hash in source epoch + ancestry hash in source epoch)* instead of simply the hash of each epoch because you can have two prepare with same hash but based on different source prepare