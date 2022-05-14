Day 1
Deploy a contract to account 0x03 called "JacobTucker". Inside that contract, declare a constant variable named is, and make it have type String. Initialize it to "the best" when your contract gets deployed.

pub contract JacobTucker {

  pub let is: String
  init() {
      self.is = "the best"
  }
}

Check that your variable is actually equals "the best" by executing a script to read that variable. Include a screenshot of the output.

import JacobTucker from 0x03

pub fun main(): String {
  return JacobTucker.is
}

![image](https://user-images.githubusercontent.com/25767253/168428170-7c6c6a1b-e3dd-4049-906a-a91de7bd4d7b.png)


Day 2
Explain why we wouldn't call changeGreeting in a script.
Because changeGreeting is a function that interact with the blockchain, it needs to be in a transaction. What we need to call is a script should be the variable greetings.

What does the AuthAccount mean in the prepare phase of the transaction?
It is the Account on Flow that accepted the transaction so we can acces to its data.

What is the difference between the prepare phase and the execute phase in the transaction?
Prepare is used to acces to the information of the account while the execute phase cannot do that but it can execute other functions to interact with the blockchain.

Add two new things inside your contract:

A variable named myNumber that has type Int (set it to 0 when the contract is deployed)
    pub var myNumber: Int

A function named updateMyNumber that takes in a new number named newNumber as a parameter that has type Int and updates myNumber to be newNumber
    pub fun updateMyNumber(newNumber: Int) {
        self.myNumber = newNumber
    }

Add a script that reads myNumber from the contract
    import HelloWorld from 0x01

    pub fun main(): Int {
        return HelloWorld.myNumber
    }

Add a transaction that takes in a parameter named myNewNumber and passes it into the updateMyNumber function. Verify that your number changed by running the script again.
    import HelloWorld from 0x01

    transaction(myNewNumber: Int) {

     prepare(signer: AuthAccount) {}

      execute {
        HelloWorld.updateMyNumber(newNumber: myNewNumber)
     }
    }

![image](https://user-images.githubusercontent.com/25767253/168429092-6521a6ec-6ee6-455a-831b-60a1ac45cc69.png)

Day 3
In a script, initialize an array (that has length == 3) of your favourite people, represented as Strings, and log it.
  var myArray: [String] = ["Ainhoa", "Jacob", "Sara"]
  log (myArray)

In a script, initialize a dictionary that maps the Strings Facebook, Instagram, Twitter, YouTube, Reddit, and LinkedIn to a UInt64 that represents the order in which you use them from most to least. For example, YouTube --> 1, Reddit --> 2, etc. If you've never used one before, map it to 0!

Explain what the force unwrap operator ! does, with an example different from the one I showed you (you can just change the type).

Using this picture below, explain...

What the error message means
Why we're getting this error
How to fix it


Day 4
Deploy a new contract that has a Struct of your choosing inside of it (must be different than Profile).

Create a dictionary or array that contains the Struct you defined.

Create a function to add to that array/dictionary.

Add a transaction to call that function in step 3.

Add a script to read the Struct you defined.
