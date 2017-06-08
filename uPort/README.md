## uPort

`userKey` is your private identity and `Proxy` can be consider your public identity. Your interact with other contracts via `Proxy` by fowarding the call first to `Controller` then `Proxy` and finally to other contracts. If your lose your `userKey`, you can replace one by `Recovery` collecting enough votes from your trusted parties. As long as you have your `userKey` intact, you can do whatever you want to replace/modify `Proxy`, `Controller` and `Recovery`.

1. demo1: shows how uPort identity is created and managed.
    * EOA stands for externally owned account
    * Black arrows with no description stand for create contract call. Black lines with description stands for contract call.
    * Factory contract creates Proxy, Controller and Recovery contracts respectively.
    * Blue arrows stand for link by storage variable. Arrow origin is the contract which stores the variable with the name shown besides arrow. Arrow destination is the value of the variable which is a contract or an EOA.

2. demo2: similar to demo1 but replace Recovery contract with a single recovery key(which is an EOA).