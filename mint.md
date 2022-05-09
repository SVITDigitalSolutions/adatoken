Create your keys from passphrase (using account 2)

```
mkdir $HOME/cnode/keys
chdir $HOME/cnode/keys
cat phrase.prv | $HOME/cnode/cardano-address key from-recovery-phrase Shelley > root.xsk

#Create private keys [note the 1H]
$HOME/cnode/cardano-address key child 1852H/1815H/1H/0/0 < root.xsk > payment.xsk
$HOME/cnode/cardano-cli key convert-cardano-address-key --shelley-payment-key --signing-key-file payment.xsk --out-file payment.skey

$HOME/cnode/cardano-address key child 1852H/1815H/1H/2/0 < root.xsk > stake.xsk
$HOME/cnode/cardano-cli key convert-cardano-address-key --shelley-stake-key --signing-key-file stake.xsk --out-file stake.skey

#Public(verification) Keys

$HOME/cnode/cardano-cli key verification-key --signing-key-file payment.skey --verification-key-file payment.evkey
$HOME/cnode/cardano-cli key verification-key --signing-key-file stake.skey --verification-key-file stake.evkey

$HOME/cnode/cardano-cli key non-extended-key --extended-verification-key-file payment.evkey --verification-key-file payment.vkey
$HOME/cnode/cardano-cli key non-extended-key --extended-verification-key-file stake.evkey --verification-key-file stake.vkey

testnet="--testnet-magic 1097911063"
$HOME/cnode/cardano-cli address build --payment-verification-key-file payment.vkey --stake-verification-key stake.vkey $testnet > testpayment.addr
```
Policy Keys - Using Account 3
```
$HOME/cnode/cardano-address key child 1852H/1815H/2H/0/0 < root.xsk > policy.xsk
$HOME/cnode/cardano-cli key convert-cardano-address-key --shelley-payment-key --signing-key-file policy.xsk --out-file policy.skey
$HOME/cnode/cardano-cli key verification-key --signing-key-file policy.skey --verification-key-file policy.evkey
$HOME/cnode/cardano-cli key non-extended-key --extended-verification-key-file policy.evkey --verification-key-file policy.vkey
```

```
address=$(cat testpayment.addr)

$HOME/cnode/cardano-cli query protocol-parameters $testnet --out-file protocol.json

mkdir policy

touch ./policy/policy.script && echo "" > ./policy/policy.script
echo "{" >> ./policy/policy.script 
echo "  \"keyHash\": \"$($HOME/cnode/cardano-cli address key-hash --payment-verification-key-file ./keys/policy.vkey)\"," >> ./policy/policy.script 
echo "  \"type\": \"sig\"" >> ./policy/policy.script
echo "}" >> ./policy/policy.script

$HOME/cnode/cardano-cli transaction policyid --script-file ./policy/policy.script > ./policy/policyID

$HOME/cnode/cardano-cli query utxo --address $address $testnet

txhash="<insert your txhash here>"
txix="<insert your TxIx here>"
funds="<funds here>"
policyid=$(cat ./policy/policyID)
fee="300000"


$HOME/cnode/cardano-cli transaction build-raw \
 --fee $fee \
 --tx-in $txhash#$txix \
 --tx-out $address+$output+"$tokenamount $policyid.$tokenname1" \
 --mint="$tokenamount $policyid.$tokenname1" \
 --minting-script-file policy/policy.script \
 --out-file matx.raw
```
Now calculate the real fee
```
fee=$($HOME/cnode/cardano-cli transaction calculate-min-fee --tx-body-file matx.raw --tx-in-count 1 --tx-out-count 1 --witness-count 2 $testnet --protocol-params-file protocol.json | cut -d " " -f1)

output=$(expr $funds - $fee)

Rebuild

$HOME/cnode/cardano-cli transaction build-raw \
 --fee $fee \
 --tx-in $txhash#$txix \
 --tx-out $address+$output+"$tokenamount $policyid.$tokenname1" \
 --mint="$tokenamount $policyid.$tokenname1" \
 --minting-script-file policy/policy.script \
 --out-file matx.raw

Sign

$HOME/cnode/cardano-cli transaction sign  \
--signing-key-file $HOME/cnode/keys/payment.skey  \
--signing-key-file $HOME/cnode/keys/policy.skey  \
$testnet --tx-body-file matx.raw  \
--out-file matx.signed

and submit

$HOME/cnode/cardano-cli transaction submit --tx-file matx.signed $testnet
```
check
```
$HOME/cnode/cardano-cli query utxo --address $address $testnet
```
