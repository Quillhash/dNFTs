# dNFTs

The proposed standard is for distributed ownership of non fungible assets on EOS. For the digital assets like videos, music, e-books, photography etc., partial ownership is generally required. It would allow user to sell some part of their ownership on EOS platform by listing for it. Also, it would motivate the public participation in such valuable assets too.

**Telegram**: https://t.me/dnfts_eos   
**Medium** : https://medium.com/quillhash/partial-ownership-on-eos-773aea600e3

### Architecture
<img src="https://github.com/Quillhash/dnfts/blob/master/architecture.png" >

### RAM usage
The RAM usage is used for every creation and listing of the digital asset. It also depends on how much data is stored in the mdata fields. If considered empty, each digital asset may take upto 0.4 - 0.5 kB RAM. The contract deployment requires ~717 kB RAM.

### Platforms using it   
**Zeptagram** is the one doing great job using this standard and are first to implement it. They have created a marketplace for music rights via their platform. It shall be an awesome oppurtunity for anyone to list and earn from their music.
https://zeptagram.com/

---------------------------  

### Contract Actions:

#### setconfig
```
ACTION setconfig(string version)
```

To maintain version and token category ids. 

#### create

```
ACTION create (name issuer,
               name category,
               name token_name,
               bool burnable, 
               string base_uri)
 ```

To create a new token under some category like ```Harrypotter``` for ```ebook``` category. 

#### issue

```
ACTION issue (name to, 
              name category,
              name token_name,
              asset quantity,
              string memo)
```     

It is callable by issuer; to issue to their true ownership holders as per their percentage. Quantity needs to be in PER only.


#### burn

```
ACTION burn (name owner, 
             uint64_t category_name_id, 
             asset quantity)
```
It is callable only by token owner; to burn quantity PERs of  their ownership on particular token. 


#### buyshare

```
ACTION buyshare(name from,
                name to, 
                asset quantity, 
                string memo)
```
It is called when a user sends EOS from eosio.token to this smart contract account with a memo of "sale_id,to_account"


#### listsale

```
ACTION listsale (name seller,
                 uint64_t category_id, 
                 asset percent_share, 
                 asset per_percent_amt)
```
It is callable only by owner; creates a sale listing in the token contract valid
  for 1 week, transfers ownership to token contract


#### closesale

```
ACTION closesale (name seller, 
                  uint64_t sale_id)
```
It is callable by seller if listing hasn't expired, or anyone if the listing is expired.


#### giveroyalty

```
ACTION giveroyalty (uint64_t category_id,
                    asset royalty)
```
This action is used to give away the royalty collected to the ownership holders of particular VT-A. The x% ownership means x% of the royalty. Can be called by the `_self` every quarter. 

---------------------------  

### Data Structures

#### tokenconfigs
```
TABLE tokenconfigs {
            name standard;
            string version;
        };
```
Scope is _self.

#### categoryids
```
TABLE categoryids{
    uint64_t cat_id;
    uint64_t primary_key() const { return cat_id; }
};
```

#### accounts
```
TABLE accounts {
            uint64_t category_name_id;
            name category;
            name token_name;
            asset amount;
            uint64_t primary_key() const { return category_name_id; }
        };
```
Scope is owner of the asset.

#### dnft
```
TABLE dnft {
                name owner;
                uint64_t primary_key() const { return owner.value; }
      };
```
Scope is category_id (category name -> token name)

#### dnftstats
```
TABLE dnftstats {
            bool    burnable;
            name    issuer;
            name    token_name;
            uint64_t category_name_id;
            asset current_supply;
            asset issued_supply;
            string base_uri;
            uint64_t primary_key() const { return token_name.value; }
        };
```        
Scope is category,and token_name is unique, has information of all the tokens in the category..

#### asks
```
TABLE asks {
            uint64_t sale_id;
            name token_name;
            name category;
            name seller;
            asset per_percent_amt;
            asset percent_share;
            time_point_sec expiration;
            uint64_t primary_key() const { return sale_id; }
            uint64_t get_seller() const { return seller.value; }
        };
```
Scope is _self. All the information of sale listings are here.

---------------------------  

## How to buy dNFTs ownership?

To buy the ownership, one has to transfer the EOS from the eosio.token **```transfer```** action with the memo as:
**```sale_id,to```**

## Standards for digital assets and NFTs  
If your use case is different, the following standards may be helpful:  
* https://github.com/CryptoLions/SimpleAssets   
* https://github.com/MythicalGames/dgoods   
* https://github.com/unicoeos/eosio.nft   
