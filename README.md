# dNFTs

dNFTs is an open source free standard that provides distributed ownership of non fungible assets on EOS blockchain. It offers utility to various industries that require ownership in the form of distributed rights and shares(in percentage).
Its use cases involve multimedia industries (videos, music, e-books, photography etc.) wherein many shareholders can exist. It allows creator as well as investor to sell their ownership on EOS platform by listing or bidding for it. Also, it would motivate the public participation in such valuable assets that are existent in real-world.


**Telegram**: https://t.me/dnfts_eos   
**Medium** : https://medium.com/quillhash/partial-ownership-on-eos-773aea600e3  
**Website**: https://www.quillhash.com/products/distributed-non-fungible-tokens          
**Jungle account**: [dnftversion1](https://jungle.bloks.io/account/dnftversion1)              


### Architecture
<img src="https://github.com/Quillhash/dNFTs/blob/master/architecture.png" >

### RAM usage
The RAM usage is used for every creation and listing of the digital asset. It also depends on how much data is stored in the mdata fields. If considered empty, each digital asset may take upto 0.4 - 0.5 kB RAM.
Except the create action, for all other actions RAM payer may be configured to the contract.

---------------------------  

### Rationale
0. `setconfig`action is called by contract account specifying the version that helps external contracts parse actions and tables 
1. Contract account adds the validators account for categories like `videos`, `music`, `ebooks`, `photography` etc. `addvalidator` action is called.
2. Anyone can create the NFT under the category by calling `create` action. The creator needs to send mail to both validators with original JSON file, listing their creation & the legal binding agreement (if needed for some high valued NFT in real world). 
3. 2 validators validate the creation after taking fees offered. Validators validate using `validate` action.
4. After validated creation of NFT by both, NFT creator can issue ownership of NFTs to different accounts. Issuing NFT can be done by calling action `issue`.
5. All current owners of NFT can list for sale of their ownership at fixed rate (using `listsale` action) or choose the bidding mechanism (using `startbid` action).
6. Investors can buy the ownership using eosio.token **```transfer```** action by sending EOS to the contract with the memo as:
**```sale_id,to```**
7. Sale listing or bidding may be closed by seller before expiration. Bidding is realised as unsuccessful if that happens. For the successful bidding, highest bidder receives the ownership shares. Actions for them are `closesale` and `closebid`.
8. For all the accounts with zero ownership may be deleted by anyone if the ram payer is configured as contract. Action `close` can be called to do so.
9. `burn` action needs to be called by creator in case of non-validation to release RAM or when anyone wants to burn their ownership shares.
10. In case any validator is found suspicious, `delvalidator` action is called by contract account. Validator account no longer can validate any further NFTs.


##### Note: To ensure the authenticity of NFT, mail mechanism is kept. Keeping blockchain fundamentals as permissionless, any EOS account can invest and do all operations normally.

---------------------------  

### Contract Actions:

#### setconfig

```
ACTION setconfig(string version)
```

To maintain version of the contract for external marketplaces, wallets etc. 

#### create

```
ACTION create (name issuer,
               name category,
               name token_name,
               bool burnable, 
               string base_uri,
               uint8_t span,
               checksum256 origin_hash,
               uint64_t validator_id)
 ```

To create a new NFT under some category like ```Harrypotter``` for ```ebook``` category. The validator_id is chosen manually from validators table as per the trust score.


#### issue

```
ACTION issue (name to, 
              name category,
              name token_name,
              asset quantity,
              string memo)
```     

It is callable by issuer; to issue to their true ownership holders as per their percentage. Quantity needs to be in PER only.


#### addvalidator 

```
ACTION addvalidator (name validator,
                     string email,
                     name category)
```

It is callable by _self; to add trusted validator to the blockchain. Validator validates the NFT creation of the category authorised to.


#### delvalidator

```
ACTION delvalidator (name owner)
```

It is callable by _self; to remove validator. Account cannot validate any category thereafter.


#### validate

```
ACTION validate (name validator,
                 name category,
                 name token_name)
```

It is callable by validator (chosen) after validating the NFT creation and for existence of that digital asset in real world.


#### startbid

```
ACTION startbid (name seller,
                 name category,
                 name token_name,
                 asset percent_share,
                 asset base_price,
                 time_point_sec expiration)
```

It is callable by validated NFT holder to sell their rights in the form of bidding.

#### closebid

```
ACTION closebid (name seller,
                 uint64_t sale_id);

```

It is callable by seller who listed for bidding to cancel the bidding before expiration time. No rights are sold in this case. If however, the action is called after expiration time (anyone can call), the rights listed for bidding are transferred to the highest bidder. The RAM is released in both the cases.


#### burn

```
ACTION burn (name owner, 
             uint64_t dnft_id, 
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
It is called when a user sends EOS from eosio.token to this smart contract account with a memo of "sale_id,to_account". The sale ids are unique and no sale `listings` and `biddings` sale_ids can be same.


#### listsale

```
ACTION listsale (name seller,
                 name category,
                 name token_name,
                 asset percent_share,
                 asset per_percent_amt,
                 time_point_sec expiration)
```
It is callable only by owner; creates a sale listing in the token contract valid for 1 week, transfers ownership to token contract


#### closesale

```
ACTION closesale (name seller, 
                  uint64_t sale_id)
```
It is callable by seller if listing hasn't expired, or by anyone if the listing is expired.


### Data Structures

#### tokenconfigs
```
TABLE tokenconfigs {
        name standard;
        string version;
};
```
Scope is _self.

#### categories
```
TABLE categories {
        name category;
        vector<uint64_t> validator_ids;
        uint64_t primary_key() const { return category.value; }
};
```
Scope is _self.

#### uids
```
TABLE uids{
    uint64_t dnft_id;
    uint64_t validator_id;
};

```
Singleton table that stores latest dnft_id and validator_id.

#### accounts
```
TABLE accounts {
        uint64_t dnft_id;
        asset amount;
        uint64_t primary_key() const { return dnft_id; }
};
```
Scope is owner of the NFT.

#### dnft
```
TABLE dnft {
    name owner;
    uint64_t primary_key() const { return owner.value; }
};
```
Scope is dnft_id (category name -> token name)

#### dnftstats
```
TABLE dnftstats {
            bool burnable;
            name issuer;
            name token_name;
            uint64_t dnft_id;
            asset current_supply;
            asset issued_supply;
            string base_uri;
            checksum256 origin_hash;
            uint8_t span;
            bool validated;
            map<uint64_t, bool> validated_by;
            uint64_t primary_key() const { return token_name.value; }
            uint64_t is_verified() const { return validated; }
        };
```        
Scope is category,and token_name is unique, has information of all the NFTs in the category. Validated and unvalidated NFT creations can be separated through secondary index.

#### lists
```
TABLE lists {
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

#### bids
```
TABLE bids {
            uint64_t sale_id;
            name token_name;
            name category;
            name seller;
            asset percent_share;
            asset base_price;
            asset current_bid;
            name current_bidder;
            time_point_sec expiration;
            uint64_t primary_key() const { return sale_id; }
            uint64_t get_seller() const { return seller.value; }
         }; 
```
Scope is _self. All the information of bidding listings are here.

#### validators
```
TABLE validators {
            uint64_t validator_id;
            name validator;
            string email;
            vector<name> categories;
            bool active;
            uint64_t primary_key() const { return validator.value; }
        };

```
Scope is _self. The list of all the trusted validators accounts with their unique emails are here. 

---------------------------  


## Standards for digital assets and NFTs  
If your use case is different, the following standards may be helpful:  
* https://github.com/CryptoLions/SimpleAssets   
* https://github.com/MythicalGames/dgoods   
* https://github.com/unicoeos/eosio.nft   

## Credits
* [Quillhash](https://www.quillhash.com/)
* [cc32d9|EOS Amsterdam](https://github.com/cc32d9/cc32d9_ideas_for_EOS/blob/master/Object_Token.md)
* [Zeptagram](https://zeptagram.com/) 
