Day 1
Explain what lives inside of an account.
Contract and data

What is the difference between the /storage/, /public/, and /private/ paths?
in the storage account only de accoun owner can have access, public is available for everyone and private is only available to the owner and the people that the owner gives acces.

What does .save() do? What does .load() do? What does .borrow() do?
save - save the data in some storage path, it inserts data.
load - remove the data
borrow - view the data

Explain why we couldn't save something to our account storage inside of a script.
Scripts are only to view data.

Explain why I couldn't save something to your account.
Because you do not have authorization.

Define a contract that returns a resource that has at least 1 field in it. Then, write 2 transactions:
pub contract C4D1 {

    pub resource Calendar { 
        pub var month: String 
        
        init() { 
            self.month = "June" 
        }
    }

    pub fun createCalendar(): @Calendar { 
    return <- create Calendar() 
    }
}   

A transaction that first saves the resource to account storage, then loads it out of account storage, logs a field inside the resource, and destroys it.

import C4D1 from 0x01 

transaction() { 
  prepare(signer: AuthAccount) { 
  
  let oneMonth <- C4D1.createCalendar() 
  signer.save(<- oneMonth, to: /storage/MyCalendar)
  let outMonth <- signer.load<@C4D1.Calendar>(from: /storage/MyCalendar)!
  log(outMonth.month)
  
  destroy outMonth
  }

  execute {

  } 
}

A transaction that first saves the resource to account storage, then borrows a reference to it, and logs a field inside the resource.

import C4D1 from 0x01 

transaction() { 
  prepare(signer: AuthAccount) { 
  
  let oneMonth <- C4D1.createCalendar() 
  signer.save(<- oneMonth, to: /storage/MyCalendar)
  let outMonth = signer.borrow<&C4D1.Calendar>(from: /storage/MyCalendar) ?? panic ("No results, check it")
  log(outMonth.month)
  }

  execute {

  } 
}


Day 2
Please answer in the language of your choice.

What does .link() do?
Put data to the public so we can read 

In your own words (no code), explain how we can use resource interfaces to only expose certain things to the /public/ path.
To limit utility. Is important to use it so we can limit the funcions people outside can use. As for now seems good to avoid writing functions in the interface but yes in the resource.

Deploy a contract that contains a resource that implements a resource interface. Then, do the following:
pub contract C4D2 {

pub resource interface ICalendar { 
  pub var name: String 
}

pub resource Calendar: ICalendar { 
  pub var name: String

  pub fun changeName(newName: String) {
    self.name = newName
  }

  init() {
    self.name = "June"
  }
}
  pub fun createCalendar(): @Calendar {
    return <- create Calendar() 
  }
}

In a transaction, save the resource to storage and link it to the public with the restrictive interface.

import C4D2 from 0x02

transaction() { 
  prepare(signer: AuthAccount) { 
  
  signer.save(<- C4D2.createCalendar(), to: /storage/MyCalendar2)
  signer.link<&C4D2.Calendar{C4D2.ICalendar}>(/public/MyCalendar2, target: /storage/MyCalendar2)

  }

  execute {

  } 
}

Run a script that tries to access a non-exposed field in the resource interface, and see the error pop up.

import C4D2 from 0x02

transaction(address: Address) { 
  prepare(signer: AuthAccount) {
  }

  execute { 
    let publicCapability: Capability<&C4D2.Calendar> = getAccount(address).getCapability<&C4D2.Calendar>(/public/MyCalendar2)

    let calendar: &C4D2.Calendar = publicCapability.borrow() 
        ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

    calendar.changeName(newName: "May")
  } 
}

Run the script and access something you CAN read from. Return it from the script.

import C4D2 from 0x02

transaction(address: Address) { 
  prepare(signer: AuthAccount) {
  }

  execute { 
    let publicCapability: Capability<&C4D2.Calendar{C4D2.ICalendar}> = getAccount(address).getCapability<&C4D2.Calendar{C4D2.ICalendar}>(/public/MyCalendar2)

    let calendar: &C4D2.Calendar{C4D2.ICalendar} = publicCapability.borrow() 
        ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

    calendar.name
  } 
}


Day 3
Why did we add a Collection to this contract? List the two main reasons.

What do you have to do if you have resources "nested" inside of another resource? ("Nested resources")

Brainstorm some extra things we may want to add to this contract. Think about what might be problematic with this contract and how we could fix it.

Idea #1: Do we really want everyone to be able to mint an NFT? 🤔.

Idea #2: If we want to read information about our NFTs inside our Collection, right now we have to take it out of the Collection to do so. Is this good?


Day 4
Because we had a LOT to talk about during this Chapter, I want you to do the following:

Take our NFT contract so far and add comments to every single resource or function explaining what it's doing in your own words. Something like this:

pub contract CryptoPoops {
  pub var totalSupply: UInt64

  // This is an NFT resource that contains a name,
  // favouriteFood, and luckyNumber
  pub resource NFT {
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

  // This is a resource interface that allows us to... you get the point.
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }

  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }

    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NFT {
      return &self.ownedNFTs[id] as &NFT
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @Collection {
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
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
