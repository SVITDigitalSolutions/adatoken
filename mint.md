Create your keys from passphrase

```
mkdir $HOME/cnode/keys
chdir $HOME/cnode/keys
cat phrase.prv | $HOME/cnode/cardano-address key from-recovery-phrase Shelley > root.xsk

#Create private keys [note the 1H - 2nd account is the maker]
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
