Day 1
1. In words, list 3 reasons why structs are different from resources.
    They cannot be copied, lost or creather whenever you want.

2. Describe a situation where a resource might be better to use than a struct.
    To make it very difficult to destroy or lose the variable, for example, NFT.

3. What is the keyword to make a new resource?
    Create

4. Can a resource be created in a script or transaction (assuming there isn't a public function to create one)?
    No.

5. What is the type of the resource below?

pub resource Jacob {

}

    It is public.

1. Let's play the "I Spy" game from when we were kids. I Spy 4 things wrong with this code. Please fix them.
pub contract Test {

    // Hint: There's nothing wrong here ;)
    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }

    pub fun createJacob(): Jacob { // there is 1 here
        let myJacob = Jacob() // there are 2 here
        return myJacob // there is 1 here
    }
}

Missing "@" when we create the function (@Jacob)
Missing "<-" when we move the value and the word "create"
We cannot return a value, we move it to the return value with "<-".

Correct code
    pub fun createJacob(): @Jacob { // there is 0 here
        let myJacob <- create Jacob() // there are 0 here
        return <- myJacob // there is 0 here


Day 2
Write your own smart contract that contains two state variables: an array of resources, and a dictionary of resources. Add functions to remove and add to each of them. They must be different from the examples above.

pub contract C3D2 {

  pub var day: @{UInt64: Days}

  pub var month: @[Month]

  pub resource Month {
      pub let nameMonth: String
      init() {
          self.nameMonth = "June"
      }
  }

  pub resource Days {
      pub let numberDay: UInt64
      init() {
          self.numberDay = 1
      }
  }

  pub fun addDay(day: @Days) {
      let key = day.numberDay
      
      let oldDay <- self.day[key] <- day
      destroy oldDay
  }

  pub fun removeDay(key: UInt64): @Days {
      let day <- self.day.remove(key: key) ?? panic("Could not find the day!")
      return <- day}

  pub fun addMonth(month: @Month) {
      self.month.append(<- month)
  }

  pub fun removeMonth(index: Int): @Month {
      return <- self.month.remove(at: index)
  }

  init() {
      self.month <- [] 
      self.day <- {}
  }
}


Day 3
Define your own contract that stores a dictionary of resources. Add a function to get a reference to one of the resources in the dictionary.

pub contract C3D3 {

  pub var day: @{UInt64: Days}

  pub resource Days {
      pub let numberDay: UInt64
      init(_numberDay: UInt64) {
          self.numberDay = _numberDay
      }
  }

  pub fun getReference (key: UInt64): &Days {
    return &self.day[key] as &Days
  }

  init() {
      self.day <- {
       28: <- create Days(_numberDay: 365), 
       27: <- create Days(_numberDay: 364)
      }
  }
}

Create a script that reads information from that resource using the reference from the function you defined in part 1.

import C3D3 from 0x03

pub fun main(): UInt64 { 
let ref = C3D3.getReference(key: 28) 
return ref.numberDay // returns "English" 
}

Explain, in your own words, why references can be useful in Cadence.

We can get the info without moving the resources and resources force as to type cast a reference or we receive an error, which it can be helpful.

Day 4
Explain, in your own words, the 2 things resource interfaces can be used for (we went over both in today's content)

It specifies a set of requirements for something to implement and it allows you to only expose certain things to certain people. Resource must have everything that the interface have at least.

Define your own contract. Make your own resource interface and a resource that implements the interface. Create 2 functions. In the 1st function, show an example of not restricting the type of the resource and accessing its content. In the 2nd function, show an example of restricting the type of the resource and NOT being able to access its content.

pub contract C3D4 {

  pub resource interface ICalendar {
    pub var month: String
    pub var day: Int
    pub fun updateDay(newDay: Int): Int
  }

  pub resource Calendar: ICalendar {
    pub var month: String
    pub var day: Int

    pub fun updateDay(newDay: Int): Int {
      self.day = newDay
      return self.day // returns the new day
    }

    init() {
      self.month = "June"
      self.day = 4
    }
  }

  //not restricting 
  pub fun yesInterface() {
    let calendar: @Calendar{ICalendar} <- create Calendar()
    let newDay = calendar.updateDay(newDay: 5)
    log(newDay) 

    destroy calendar
  }

  //restricting 
  pub fun noInterface() {
    let calendar: @Calendar <- create Calendar()
    calendar.updateDay(newDay: 5)
    log(calendar.day)

    destroy calendar
  }
}


How would we fix this code?

pub contract Stuff {

    pub struct interface ITest {
      pub var greeting: String
      pub var favouriteFruit: String
    }

    // ERROR:
    // `structure Stuff.Test does not conform 
    // to structure interface Stuff.ITest`
    pub struct Test: ITest {
      pub var greeting: String

      pub fun changeGreeting(newGreeting: String): String {
        self.greeting = newGreeting
        return self.greeting // returns the new greeting
      }

      init() {
        self.greeting = "Hello!"
      }
    }

    pub fun fixThis() {
      let test: Test{ITest} = Test()
      let newGreeting = test.changeGreeting(newGreeting: "Bonjour!") // ERROR HERE: `member of restricted type is not accessible: changeGreeting`
      log(newGreeting)
    }
}

CODE without errors
pub contract Stuff {

    pub struct interface ITest {
      pub var greeting: String
      pub var favouriteFruit: String
      pub fun changeGreeting(newGreeting: String): String 
    }

    pub struct Test: ITest {
      pub var greeting: String
      pub var favouriteFruit: String

      pub fun changeGreeting(newGreeting: String): String {
        self.greeting = newGreeting
        return self.greeting // returns the new greeting
      }

      init() {
        self.greeting = "Hello!"
        self.favouriteFruit = "Strawberry"
      }
    }

    pub fun fixThis() {
      let test: Test{ITest} = Test()
      let newGreeting = test.changeGreeting(newGreeting: "Bonjour!")
      log(newGreeting)
    }
}


Day 5
For today's quest, you will be looking at a contract and a script. You will be looking at 4 variables (a, b, c, d) and 3 functions (publicFunc, contractFunc, privateFunc) defined in SomeContract. In each AREA (1, 2, 3, and 4), I want you to do the following: for each variable (a, b, c, and d), tell me in which areas they can be read (read scope) and which areas they can be modified (write scope). For each function (publicFunc, contractFunc, and privateFunc), simply tell me where they can be called.

access(all) contract SomeContract {
    pub var testStruct: SomeStruct

    pub struct SomeStruct {

        //
        // 4 Variables
        //

        pub(set) var a: String

        pub var b: String

        access(contract) var c: String

        access(self) var d: String

        //
        // 3 Functions
        //

        pub fun publicFunc() {}

        access(contract) fun contractFunc() {}

        access(self) fun privateFunc() {}


        pub fun structFunc() {
            /**************/
            /*** AREA 1 ***/
            /**************/
            
            // a =  read/write
            // b =  read
            // c =  read
            // d =  read
            // publicFunc = access
            // contractFunc = access
            // privateFunc = access
            
        }

        init() {
            self.a = "a"
            self.b = "b"
            self.c = "c"
            self.d = "d"
        }
    }

    pub resource SomeResource {
        pub var e: Int

        pub fun resourceFunc() {
            /**************/
            /*** AREA 2 ***/
            /**************/
            
            // a =  read/write
            // b =  read
            // c =  read
            // d =  no read
            // publicFunc = access
            // contractFunc = access
            // privateFunc = no access
            
        }

        init() {
            self.e = 17
        }
    }

    pub fun createSomeResource(): @SomeResource {
        return <- create SomeResource()
    }

    pub fun questsAreFun() {
        /**************/
        /*** AREA 3 ****/
        /**************/
        
        // a =  read/write
        // b = read
        // c = read
        // d = no read
        // publicFunc = access 
        // contractFunc = access
        // privateFunc = no access
        
    }

    init() {
        self.testStruct = SomeStruct()
    }
}
This is a script that imports the contract above:

import SomeContract from 0x01

pub fun main() {
  /**************/
  /*** AREA 4 ***/
  /**************/
  
  // a = read/write 
  // b = read 
  // c = no read/write 
  // d = no read/write 
  // publicFunc = access 
  // contractFunc = no access 
  // privateFunc = no access
  
}

