Day 1
Describe what an event is, and why it might be useful to a client.
events are used to communicate to the outside world that something happened

Deploy a contract with an event in it, and emit the event somewhere else in the contract indicating that it happened.
pub contract C5D1 {

  pub event EventDay (id: UInt64)

  pub resource NewDay { 
    pub let id: UInt64
    init() {
      self.id = self.uuid
      emit Day (id: self.id)
    }
  }
}

Using the contract in step 2), add some pre conditions and post conditions to your contract to get used to writing them out.

pub contract C5D1 {

  pub event EventDay (id: UInt64)

  pub resource Day { 
    pub let id: UInt64

    pub fun newDay(date:Int): Int{
    pre{
      date > 9  && date =< 31: "The date must have two digits"
    }
    return date
    }

    pub fun monhtDate(month: String){
    post{
      month.length >= 2: "No month with this lenght"
    }
    log(month)
    }
    init() {
      self.id = self.uuid
      emit EventDay (id: self.id)
    }
  }
}

For each of the functions below (numberOne, numberTwo, numberThree), follow the instructions.

pub contract Test {

  // TODO
  // Tell me whether or not this function will log the name. YES
  // name: 'Jacob'
  pub fun numberOne(name: String) {
    pre {
      name.length == 5: "This name is not cool enough."
    }
    log(name)
  }

  // TODO
  // Tell me whether or not this function will return a value. YES
  // name: 'Jacob'
  pub fun numberTwo(name: String): String {
    pre {
      name.length >= 0: "You must input a valid name."
    }
    post {
      result == "Jacob Tucker"
    }
    return name.concat(" Tucker")
  }

  pub resource TestResource {
    pub var number: Int

    // TODO
    // Tell me whether or not this function will log the updated number. NO
    // Also, tell me the value of `self.number` after it's run. =1
    pub fun numberThree(): Int {
      post {
        before(self.number) == result + 1
      }
      self.number = self.number + 1
      return self.number
    }

    init() {
      self.number = 0
    }

  }

}


Day 2
Explain why standards can be beneficial to the Flow ecosystem.
singular way with interacting with contracts, ensures a contract is what it claims to be, marketplace dapps can interact with them with certain expectations

What is YOUR favourite food?
Carbonara, only with spaguetti

Please fix this code (Hint: There are two things wrong):

The contract interface:

pub contract interface ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    pre {
      newNumber >= 0: "We don't like negative numbers for some reason. We're mean."
    }
    post {
      self.number == newNumber: "Didn't update the number to be the new number."
    }
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff {
    pub var favouriteActivity: String
  }
}
The implementing contract:
//missing to import the contract interface ITest "import ITest from 0x01"
//must implement contract interface ITest in pub contract Test "pub contract Test: ITest"
pub contract Test {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    self.number = 5
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff: IStuff {
    pub var favouriteActivity: String

    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }

  init() {
    self.number = 0
  }
}


Day 3
What does "force casting" with as! do? Why is it useful in our Collection?
we can use it to check if token is @NFT type, otherwise 'panic'

What does auth do? When do we use it?
allows us to call an authorized reference, we can use it if we want to downcast a reference auth, it must be used to call the reference

This last quest will be your most difficult yet. Take this contract:

import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return &self.ownedNFTs[id] as &NonFungibleToken.NFT
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
and add a function called borrowAuthNFT just like we did in the section called "The Problem" above.
pub fun borrowAuthNFT(id: UInt64): &NFT {
  let ref = &self.ownedNFTs[id] as auth &NonfungibleToken.NFT
  return ref as! &NFT
}

Then, find a way to make it publically accessible to other people so they can read our NFT's metadata. 
pub resource interface MyCollection { 
  pub fun borrowAuthNFT(id: UInt64): &NFT
}

Then, run a script to display the NFTs metadata for a certain id.

You will have to write all the transactions to set up the accounts, mint the NFTs, and then the scripts to read the NFT's metadata. We have done most of this in the chapters up to this point, so you can look for help there :)

import CryptoPoops from 0x03
import NonFungibleToken from 0x02

transaction {

  prepare(acct: AuthAccount) {

  acct.save(<- CryptoPoops.createEmptyCollection(), to: /storage/Collection1)


  acct.link<&CryptoPoops.Collection{NonFungibleToken.Provider, NonFungibleToken.Receiver, 
                                      NonFungibleToken.CollectionPublic, CryptoPoops.CollectionAuth}>
                                        (/public/Collection1, target: /storage/Collection1)
  }

  execute {
    log("we saved a Collection to our storage")
  }
}


import CryptoPoops from 0x03
import NonFungibleToken from 0x02

transaction(name: String, favouriteFood: String, luckyNumber: Int, recipient: Address) {

    prepare(admin: AuthAccount) {
        let minterRef = admin.borrow<&CryptoPoops.Minter>(from: /storage/Minter)
                            ?? panic("This is not Admin")
        let nft <- minterRef.createNFT(name: name, favouriteFood: favouriteFood, luckyNumber: luckyNumber)

        let CollectionRef = getAccount(recipient).getCapability(/public/Collection1)
                                .borrow<&CryptoPoops.Collection{NonFungibleToken.Receiver}>()
                                    ?? panic("Collection does not exist")

        log(nft.id)

        CollectionRef.deposit(token: <- nft)
    }

    execute {
        log("We deposited and NFT to our Collection")
    }
}


Script
import CryptoPoops from 0x03

pub fun main(address: Address, id: UInt64): String {
  let ref = getAccount(address).getCapability<&CryptoPoops.Collection{CryptoPoops.CollectionAuth}>(/public/Collection1)
            .borrow() ?? panic("Collection does not exist")

  let nftRef: &CryptoPoops.NFT = ref.borrowAuthNFT(id: id)
  return nftRef.name

}
