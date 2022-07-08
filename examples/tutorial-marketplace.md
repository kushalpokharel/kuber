# Implementing marketplace mint/buy/sell transactions on Cardano using Kuber API

Welcome to the kuber, an API/Haskell library for easily constructing transactions for Cardano.
Here you will find a tutorial that describes the features for constructing transactions using kuber API and submitting it using NAMI wallet for now.

Kuber is an open-source library and easy-to-use API created by Dquadrant which makes it easy for Cardano developers to compose complex EUTxO transactions. https://github.com/dQuadrant/kuber

We are using cardano testnet and kuber playground hosted in https://dquadrant.github.io/kuber/

Part 1 - Minting a Cardano Native Token with metadata using kuber playground and NAMI wallet
1. Setup NAMI wallet and fund it if not already
2. Here, We are going to use simple script  to mint a single token named mytoken
The script json is
```json
{
  "keyHash": "2331c473405afe1b34dfc8164f11e910f0c29a6157418652cc8b06aa",
  "type": "sig"
}
```
3. Here the value for keyHash is given a random address keyhash for example.
  If you don't have your key-hash readily available you can obtain it via kuber playground.
    Click on more menu at navbar
    Click on Get Key Hash
    Enter your address if you don't know it you can copy it from NAMI -> Click on Receive -> Copy address below QR
    Click on Get Key Hash then key hash will apear below it copy it and paste it in above script json keyHash key's value

4. Prepare metadata for your token if you want
5. Final transaction builder json for kuber will be :
```json
{
    "mint": [
        {
            "script": {
                "keyHash": "2331c473405afe1b34dfc8164f11e910f0c29a6157418652cc8b06aa",
                "type": "sig"
            },
            "amount": {
                "mytoken": 1
            }
        }
    ],
    "metadata": {
        "721": {
            "5cded5f924b5f029d95918ef5026a737ff6be56daac2667d3aa43c67": {
                "6d79746f6b656e": {
                    "name": "my nft",
                    "image": "https://ipfs.pics/QmW3FgNGeD46kHEryFUw1ftEUqRw254WkKxYeKaouz7DJA",
                    "description": "This is a test token minted from kuber api",
                    "mediaType": "image/jpeg"
                }
            }
        }
    }
}
```

Part 2 - Placing token on sell using kuber playground

Here the address is a simple marketplace contract address
The simple contract is located on 
https://github.com/dQuadrant/cardano-marketplace/blob/main/marketplace-plutus/Plutus/Contracts/V1/SimpleMarketplace.hs
and it's contract address is
```
addr_test1wqsewsqrurhxer8wx598p7naf9mmcwhr2jchkq6srtf78hctj0p2r
```
Then the value to be locked in the contract is placed as 2 Ada + Token i want to put on sell.
With the datum generated from Plutus Data Used in the contract. You can generate the script data format for cardano using Plutus which is currently out of scope for this tutorial. Anyways the datum used in this contract defines the seller address to be paid and price of the asset. The Data format used in the given contract above.
```haskell
data SimpleSale=SimpleSale{
    sellerAddress:: Address, 
    priceOfAsset:: Integer  -- price of the asset in lovelace
  } deriving(Show,Generic)

```
So the final transaction builder json for placing on sell using kuber will be :
```json
{
    "outputs": [
        {
            "address": "addr_test1wqsewsqrurhxer8wx598p7naf9mmcwhr2jchkq6srtf78hctj0p2r",
            "value": "2A + 5cded5f924b5f029d95918ef5026a737ff6be56daac2667d3aa43c67.mytoken",
            "datum":{"fields":[{"fields":[{"fields":[{"bytes":"570d3e0ae4c913fb3d791a62dcb04b3fbdf03c9cc9fdae966b788db5"}],"constructor":0},{"fields":[],"constructor":1}],"constructor":0},{"int":2000000}],"constructor":0}
        }
    ]
}
```

Part 3 - Buying token placed on sell using kuber playground
1. Sample script json for the contract above
```json
{
    "type": "PlutusScriptV2",
    "description": "",
    "cborHex": "590ef7590ef4010000323232323322332232323232323322323233223232323232323322323232323232323232323232323232323232332232322223232325335001102d13502b4901035054350032323235003225335350022233500223503100125030215335005153355335333573466e1cd401088cccd54c04c48004c8cd405c88ccd406000c004008d4054004cd4058888c00cc008004800488cdc00009a801111a80091111a8021119a801124000490011a80111111111111100624000900101b01a881b099ab9c491164d756c7469706c652073637269707420696e707574730003515335333573466e20c8c0d0004ccd54c0344800488cd54c048480048d400488cd540f8008cd54c054480048d400488cd54104008ccd40048cc1112000001223304500200123304400148000004cd54c048480048d400488cd540f8008ccd40048cd54c058480048d400488cd54108008d5406000400488ccd55404c0700080048cd54c058480048d400488cd54108008d5405c004004ccd55403805c00800540e0c8d4004888888888888ccd54c0684800488d40088888d401088cd400894cd4ccd5cd19b8f01700104c04b133504d00600810082008504500a35004220020020350361036133573892010f53656c6c6572206e6f742070616964000351035153353235001222222222222533533355301e12001335021225335002210031001503b25335333573466e3c03c00411010c4d40f4004540f0010841104108d40108800840d84cd5ce24811853656c6c6572205369676e6174757265204d697373696e6700035132632031335738920118536372697074204164647265737320696e2073656c6c6572000313333573466e1cd55cea80224000466442466002006004646464646464646464646464646666ae68cdc39aab9d500c480008cccccccccccc88888888888848cccccccccccc00403403002c02802402001c01801401000c008cd4084088d5d0a80619a8108111aba1500b33502102335742a014666aa04ceb94094d5d0a804999aa8133ae502535742a01066a0420566ae85401cccd540980b1d69aba150063232323333573466e1cd55cea801240004664424660020060046464646666ae68cdc39aab9d5002480008cc8848cc00400c008cd40e1d69aba150023039357426ae8940088c98c8114cd5ce01d02282189aab9e5001137540026ae854008c8c8c8cccd5cd19b8735573aa004900011991091980080180119a81c3ad35742a00460726ae84d5d1280111931902299ab9c03a045043135573ca00226ea8004d5d09aba2500223263204133573806c08207e26aae7940044dd50009aba1500533502175c6ae854010ccd540980a08004d5d0a801999aa8133ae200135742a00460546ae84d5d1280111931901e99ab9c03203d03b135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d55cf280089baa00135742a00860346ae84d5d1280211931901799ab9c02402f02d333502823232323333333574800846666ae68cdc3a8012400446666aae7d40108d40c4488004940c00cc8cccd5cd19b875003480008cccd55cfa80291a8190910011281881a128180190189281712817128171281701889aab9d5002135573ca00226ea800401524110496e76616c69642072656465656d657200333502723232323333333574800846666ae68cdc39aab9d5004480008cccd55cfa8021281791999aab9f500425030233335573e6ae89401494cd54cd4c8c8c8c8ccccccd5d200211999ab9a3370e6aae7540112000233335573ea0084a07046666aae7d4010940e48cccd55cf9aba2500525335303535742a00e42a66a646464646666666ae900108cccd5cd19b875002480008cccd55cfa8021282111999aab9f35744a00a4a66a6464646464646666666ae900188cccd5cd19b875002480088cccd55cfa8031282611999aab9f50062504d233335573ea00c4a09c46666aae7cd5d128039299a98249aba1500a215335304a35742a01442a66a60966ae85402884d414cccc11800c00800454144541405413c9413c14814414013c8cccd5cd19b875003480008cccd55cfa8039282691999aab9f35744a0104a66a60926ae85402484d4140c110004541389413814414094130138134941289412894128941281344d55cea80209aba25001135744a00226aae7940044dd50009aba15006213504535045001150432504304604523333573466e1d400d2002233335573ea00a46a088a0864a08608c4a0840880864a0804a0804a0804a08008626aae7540084d55cf280089baa00135742a00e426a07a6604e0040022a0762a0744a07407a0780764a06e0724a06c4a06c4a06c4a06c07226ae8940044d55cf280089baa00135742a00e426a066424660020060042a06242a66a60586ae85401c84d40d0c008004540c8540c4940c40d00cc0c8940b80c0940b4940b4940b4940b40c04d5d1280089aab9e50011375400200a92010c496e76616c6964206461746100135573ca00226ea8004444888ccd54c0104800540b8cd54c01c480048d400488cd540cc008d54024004ccd54c0104800488d4008894cd4ccd54c03048004c8cd404088ccd400c88008008004d40048800448cc004894cd400840d040040c48d400488cc028008014018400c4cd40c801000d40bc004cd54c01c480048d400488c8cd540d000cc004014c8004d540d8894cd40044d5402800c884d4008894cd4cc03000802044888cc0080280104c01800c008c8004d540bc88448894cd40044008884cc014008ccd54c01c480040140100044484888c00c0104484888c004010c8004d540b08844894cd4004540b0884cd40b4c010008cd54c01848004010004c8004d540ac88448894cd40044d401800c884ccd4024014c010008ccd54c01c4800401401000448d40048800448d40048800848848cc00400c00888ccd5cd19b8f0020010230221232230023758002640026aa04e446666aae7c004940988cd4094c010d5d080118019aba200201f232323333573466e1cd55cea8012400046644246600200600460166ae854008c014d5d09aba2500223263201f33573802803e03a26aae7940044dd50009191919191999ab9a3370e6aae75401120002333322221233330010050040030023232323333573466e1cd55cea80124000466016602c6ae854008cd403804cd5d09aba2500223263202433573803204804426aae7940044dd50009aba150043335500975ca0106ae85400cc8c8c8cccd5cd19b875001480108c84888c008010d5d09aab9e500323333573466e1d4009200223212223001004375c6ae84d55cf280211999ab9a3370ea00690001091100191931901319ab9c01b026024023022135573aa00226ea8004d5d0a80119a8053ae357426ae8940088c98c8080cd5ce00a81000f09aba25001135744a00226aae7940044dd5000910919800801801099aa800bae75a224464460046eac004c8004d5408c88c8cccd55cf80112811919a81119aa81218031aab9d5002300535573ca00460086ae8800c0704d5d080089119191999ab9a3370ea002900011a80c18029aba135573ca00646666ae68cdc3a801240044a030464c6403866ae700440700680644d55cea80089baa001232323333573466e1d400520062321222230040053007357426aae79400c8cccd5cd19b875002480108c848888c008014c024d5d09aab9e500423333573466e1d400d20022321222230010053007357426aae7940148cccd5cd19b875004480008c848888c00c014dd71aba135573ca00c464c6403866ae7004407006806406005c4d55cea80089baa001232323333573466e1cd55cea80124000466442466002006004600a6ae854008dd69aba135744a004464c6403066ae700340600584d55cf280089baa0012323333573466e1cd55cea800a400046eb8d5d09aab9e500223263201633573801602c02826ea80048c8c8c8c8c8cccd5cd19b8750014803084888888800c8cccd5cd19b875002480288488888880108cccd5cd19b875003480208cc8848888888cc004024020dd71aba15005375a6ae84d5d1280291999ab9a3370ea00890031199109111111198010048041bae35742a00e6eb8d5d09aba2500723333573466e1d40152004233221222222233006009008300c35742a0126eb8d5d09aba2500923333573466e1d40192002232122222223007008300d357426aae79402c8cccd5cd19b875007480008c848888888c014020c038d5d09aab9e500c23263201f33573802803e03a03803603403203002e26aae7540104d55cf280189aab9e5002135573ca00226ea80048c8c8c8c8cccd5cd19b875001480088ccc01cdd69aba15004375a6ae85400cdd69aba135744a00646666ae68cdc3a801240004601260146ae84d55cf280311931900c19ab9c00d018016015135573aa00626ae8940044d55cf280089baa00121223002003222122333001005004003232323333573466e1d400520022300a375c6ae84d55cf280191999ab9a3370ea0049000118061bae357426aae7940108c98c804ccd5ce00400980880809aab9d50011375400224464646666ae68cdc3a800a40084244400246666ae68cdc3a8012400446424446006008600c6ae84d55cf280211999ab9a3370ea00690001091100111931900a19ab9c009014012011010135573aa00226ea80048c8cccd5cd19b87500148008804c8cccd5cd19b87500248000804c8c98c8040cd5ce00280800700689aab9d3754002920103505431002333333357480024a0104a0104a01046a0126eb40089402002c8c8c8c8ccccccd5d200211999ab9a3370ea004900111999aab9f50042500c233335573e6ae89401494cd4c02cd5d0a803109a80798050008a8069280680800791999ab9a3370ea006900011999aab9f50052500d233335573e6ae89401894cd4c030d5d0a803909a80818060008a80712807008808128060070069280512805128051280500689aab9d5002135573ca00226ea80048488c00800c8488c00400c8ccccccd5d20009280212802128021280211a8029bae00200712225335300300221001135006001121223002003112200112326320033357380020069309000990009aa8049119a800a4000446a00444a66a666ae68cdc78010068048040980380089803001990009aa8041119a800a4000446a00444a66a666ae68cdc78010060040038800898030018910010910008891001091091198008020018891091980080180124500223370000400222464600200244660066004004003"
}
```
2. Find out the input of previous sell transaction
3. Get previous datum
4. Use redeemer for buy - sample script data for redeemer
```json
{
    "fields": [],
    "constructor": 0
}
```
5. Build output for transferring the cost to the seller
```json
{
    "address": "addr_test1qq3nr3rngpd0uxe5mlypvnc3ayg0ps56v9t5rpjjej9sd250kew3mfvytehnlda2k2v7c4upv4jqjn9pfr7j9a826n2q0mdh8u",
    "value": "2Ada"
}
```
6. So the final transaction builder json for placing on sell using kuber will be :
```json
{
    "outputs": [
        {
            "address": "addr_test1vr6eutuz24n6eu49ymufe37wg89nls5w88j46zju09zuvfskhqmtk",
            "value": "2Ada"
        }
    ],
    "inputs": [
        {
            "utxo": "0e7bb4d01d7c9bca5ded73501a1bf1415ebf005efe6500d2f4a59b3299e5ab27#0",
            "script": {
                "type": "PlutusScriptV2",
                "description": "",
                "cborHex": "590ef7590ef4010000323232323322332232323232323322323233223232323232323322323232323232323232323232323232323232332232322223232325335001102d13502b4901035054350032323235003225335350022233500223503100125030215335005153355335333573466e1cd401088cccd54c04c48004c8cd405c88ccd406000c004008d4054004cd4058888c00cc008004800488cdc00009a801111a80091111a8021119a801124000490011a80111111111111100624000900101b01a881b099ab9c491164d756c7469706c652073637269707420696e707574730003515335333573466e20c8c0d0004ccd54c0344800488cd54c048480048d400488cd540f8008cd54c054480048d400488cd54104008ccd40048cc1112000001223304500200123304400148000004cd54c048480048d400488cd540f8008ccd40048cd54c058480048d400488cd54108008d5406000400488ccd55404c0700080048cd54c058480048d400488cd54108008d5405c004004ccd55403805c00800540e0c8d4004888888888888ccd54c0684800488d40088888d401088cd400894cd4ccd5cd19b8f01700104c04b133504d00600810082008504500a35004220020020350361036133573892010f53656c6c6572206e6f742070616964000351035153353235001222222222222533533355301e12001335021225335002210031001503b25335333573466e3c03c00411010c4d40f4004540f0010841104108d40108800840d84cd5ce24811853656c6c6572205369676e6174757265204d697373696e6700035132632031335738920118536372697074204164647265737320696e2073656c6c6572000313333573466e1cd55cea80224000466442466002006004646464646464646464646464646666ae68cdc39aab9d500c480008cccccccccccc88888888888848cccccccccccc00403403002c02802402001c01801401000c008cd4084088d5d0a80619a8108111aba1500b33502102335742a014666aa04ceb94094d5d0a804999aa8133ae502535742a01066a0420566ae85401cccd540980b1d69aba150063232323333573466e1cd55cea801240004664424660020060046464646666ae68cdc39aab9d5002480008cc8848cc00400c008cd40e1d69aba150023039357426ae8940088c98c8114cd5ce01d02282189aab9e5001137540026ae854008c8c8c8cccd5cd19b8735573aa004900011991091980080180119a81c3ad35742a00460726ae84d5d1280111931902299ab9c03a045043135573ca00226ea8004d5d09aba2500223263204133573806c08207e26aae7940044dd50009aba1500533502175c6ae854010ccd540980a08004d5d0a801999aa8133ae200135742a00460546ae84d5d1280111931901e99ab9c03203d03b135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d55cf280089baa00135742a00860346ae84d5d1280211931901799ab9c02402f02d333502823232323333333574800846666ae68cdc3a8012400446666aae7d40108d40c4488004940c00cc8cccd5cd19b875003480008cccd55cfa80291a8190910011281881a128180190189281712817128171281701889aab9d5002135573ca00226ea800401524110496e76616c69642072656465656d657200333502723232323333333574800846666ae68cdc39aab9d5004480008cccd55cfa8021281791999aab9f500425030233335573e6ae89401494cd54cd4c8c8c8c8ccccccd5d200211999ab9a3370e6aae7540112000233335573ea0084a07046666aae7d4010940e48cccd55cf9aba2500525335303535742a00e42a66a646464646666666ae900108cccd5cd19b875002480008cccd55cfa8021282111999aab9f35744a00a4a66a6464646464646666666ae900188cccd5cd19b875002480088cccd55cfa8031282611999aab9f50062504d233335573ea00c4a09c46666aae7cd5d128039299a98249aba1500a215335304a35742a01442a66a60966ae85402884d414cccc11800c00800454144541405413c9413c14814414013c8cccd5cd19b875003480008cccd55cfa8039282691999aab9f35744a0104a66a60926ae85402484d4140c110004541389413814414094130138134941289412894128941281344d55cea80209aba25001135744a00226aae7940044dd50009aba15006213504535045001150432504304604523333573466e1d400d2002233335573ea00a46a088a0864a08608c4a0840880864a0804a0804a0804a08008626aae7540084d55cf280089baa00135742a00e426a07a6604e0040022a0762a0744a07407a0780764a06e0724a06c4a06c4a06c4a06c07226ae8940044d55cf280089baa00135742a00e426a066424660020060042a06242a66a60586ae85401c84d40d0c008004540c8540c4940c40d00cc0c8940b80c0940b4940b4940b4940b40c04d5d1280089aab9e50011375400200a92010c496e76616c6964206461746100135573ca00226ea8004444888ccd54c0104800540b8cd54c01c480048d400488cd540cc008d54024004ccd54c0104800488d4008894cd4ccd54c03048004c8cd404088ccd400c88008008004d40048800448cc004894cd400840d040040c48d400488cc028008014018400c4cd40c801000d40bc004cd54c01c480048d400488c8cd540d000cc004014c8004d540d8894cd40044d5402800c884d4008894cd4cc03000802044888cc0080280104c01800c008c8004d540bc88448894cd40044008884cc014008ccd54c01c480040140100044484888c00c0104484888c004010c8004d540b08844894cd4004540b0884cd40b4c010008cd54c01848004010004c8004d540ac88448894cd40044d401800c884ccd4024014c010008ccd54c01c4800401401000448d40048800448d40048800848848cc00400c00888ccd5cd19b8f0020010230221232230023758002640026aa04e446666aae7c004940988cd4094c010d5d080118019aba200201f232323333573466e1cd55cea8012400046644246600200600460166ae854008c014d5d09aba2500223263201f33573802803e03a26aae7940044dd50009191919191999ab9a3370e6aae75401120002333322221233330010050040030023232323333573466e1cd55cea80124000466016602c6ae854008cd403804cd5d09aba2500223263202433573803204804426aae7940044dd50009aba150043335500975ca0106ae85400cc8c8c8cccd5cd19b875001480108c84888c008010d5d09aab9e500323333573466e1d4009200223212223001004375c6ae84d55cf280211999ab9a3370ea00690001091100191931901319ab9c01b026024023022135573aa00226ea8004d5d0a80119a8053ae357426ae8940088c98c8080cd5ce00a81000f09aba25001135744a00226aae7940044dd5000910919800801801099aa800bae75a224464460046eac004c8004d5408c88c8cccd55cf80112811919a81119aa81218031aab9d5002300535573ca00460086ae8800c0704d5d080089119191999ab9a3370ea002900011a80c18029aba135573ca00646666ae68cdc3a801240044a030464c6403866ae700440700680644d55cea80089baa001232323333573466e1d400520062321222230040053007357426aae79400c8cccd5cd19b875002480108c848888c008014c024d5d09aab9e500423333573466e1d400d20022321222230010053007357426aae7940148cccd5cd19b875004480008c848888c00c014dd71aba135573ca00c464c6403866ae7004407006806406005c4d55cea80089baa001232323333573466e1cd55cea80124000466442466002006004600a6ae854008dd69aba135744a004464c6403066ae700340600584d55cf280089baa0012323333573466e1cd55cea800a400046eb8d5d09aab9e500223263201633573801602c02826ea80048c8c8c8c8c8cccd5cd19b8750014803084888888800c8cccd5cd19b875002480288488888880108cccd5cd19b875003480208cc8848888888cc004024020dd71aba15005375a6ae84d5d1280291999ab9a3370ea00890031199109111111198010048041bae35742a00e6eb8d5d09aba2500723333573466e1d40152004233221222222233006009008300c35742a0126eb8d5d09aba2500923333573466e1d40192002232122222223007008300d357426aae79402c8cccd5cd19b875007480008c848888888c014020c038d5d09aab9e500c23263201f33573802803e03a03803603403203002e26aae7540104d55cf280189aab9e5002135573ca00226ea80048c8c8c8c8cccd5cd19b875001480088ccc01cdd69aba15004375a6ae85400cdd69aba135744a00646666ae68cdc3a801240004601260146ae84d55cf280311931900c19ab9c00d018016015135573aa00626ae8940044d55cf280089baa00121223002003222122333001005004003232323333573466e1d400520022300a375c6ae84d55cf280191999ab9a3370ea0049000118061bae357426aae7940108c98c804ccd5ce00400980880809aab9d50011375400224464646666ae68cdc3a800a40084244400246666ae68cdc3a8012400446424446006008600c6ae84d55cf280211999ab9a3370ea00690001091100111931900a19ab9c009014012011010135573aa00226ea80048c8cccd5cd19b87500148008804c8cccd5cd19b87500248000804c8c98c8040cd5ce00280800700689aab9d3754002920103505431002333333357480024a0104a0104a01046a0126eb40089402002c8c8c8c8ccccccd5d200211999ab9a3370ea004900111999aab9f50042500c233335573e6ae89401494cd4c02cd5d0a803109a80798050008a8069280680800791999ab9a3370ea006900011999aab9f50052500d233335573e6ae89401894cd4c030d5d0a803909a80818060008a80712807008808128060070069280512805128051280500689aab9d5002135573ca00226ea80048488c00800c8488c00400c8ccccccd5d20009280212802128021280211a8029bae00200712225335300300221001135006001121223002003112200112326320033357380020069309000990009aa8049119a800a4000446a00444a66a666ae68cdc78010068048040980380089803001990009aa8041119a800a4000446a00444a66a666ae68cdc78010060040038800898030018910010910008891001091091198008020018891091980080180124500223370000400222464600200244660066004004003"
            },
            "redeemer": {
                "fields": [],
                "constructor": 0
            },
            "datum":{"constructor":0,"fields":[{"constructor":0,"fields":[{"constructor":0,"fields":[{"bytes":"f59e2f825567acf2a526f89cc7ce41cb3fc28e39e55d0a5c7945c626"}]},{"constructor":1,"fields":[]}]},{"int":2000000}]}
        }
    ]
}
```

