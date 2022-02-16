

We can interact with the Bitcoin Core client via a JSON-RPC interface. But instead of using it directly `bitcoin-cli` is provided as a wrapper. 

Let's take a look at some interesting examples.

First enter `bitcoin-cli help` to see a list of commands available.

- Get blockchain info

```
$ bitcoin-cli getblockchaininfo
{
  "chain": "main",
  "blocks": 723536,
  "headers": 723536,
  "bestblockhash": "0000000000000000000259e6856e11c9fe9a4601050fd30f1cd1d6edd77c6f0f",
  "difficulty": 26690525287405.5,
  "time": 1644984755,
  "mediantime": 1644981175,
  "verificationprogress": 0.9999990006721302,
  "initialblockdownload": false,
  "chainwork": "000000000000000000000000000000000000000028cc56562bb9113cd0b5724e",
  "size_on_disk": 444246310600,
  "pruned": false,
}
```

This shows we are using the main chain, there are `723536` blocks in the chain. Our verification progress which may be much lower if you just started `bitcoind` this will be somewhere between 0-1. The size on disk in bytes which is the estimated size of the block and undo files on disk. 

We can also use a too like `jq` to grab only the values we're interested in:

```
$ bitcoin-cli getblockchaininfo | jq .blocks
723543
```

You can install `jq` with `sudo apt install jq`

If you would like to read more about the other outputs and how they are defined, checkout this [site](https://developer.bitcoin.org/reference/rpc/getblockchaininfo.html) 
- Get blockhash

We can also get more specific help by entering before the command we are interested in, for instance

```
$ bitcoin-cli help getblockhash
getblockhash height

Returns hash of block in best-block-chain at height provided.

Arguments:
1. height    (numeric, required) The height index

Result:
"hex"    (string) The block hash

Examples:
> bitcoin-cli getblockhash 1000
> curl --user myusername --data-binary '{"jsonrpc": "1.0", "id": "curltest", "method": "getblockhash", "params": [1000]}' -H 'content-type: text/plain;' http://127.0.0.1:8332/
```

Here we get a description and an example of how to make the same request using `curl`, make sure to change `myusername` to the rpc username set in the `bitcoin.conf` file, after running the command you will also be prompted for the rpc password as well.

We can get the blockhash of any block:

```
$ bitcoin-cli getblockhash 723544
00000000000000000003c9b74783083c0cf9c281bd872900abd908dbff5f52e2
```

You should see the exact same blockhash printed to your screen when running the above command.

- getrawtransaction, decoderawtransaction

We can pass this blockhash to `getrawtransaction` which will return the serialized transaction data. We can get this in human readable form by passing the output to `decoderawtransaction`:

```

```
