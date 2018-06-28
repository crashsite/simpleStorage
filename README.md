
# Getting started with Solidity development using Truffle

Link to the [Class](https://medium.com/coinmonks/getting-started-with-solidity-development-using-truffle-2cc6c1df9133) post


Presented by [Yorke Rhodes](https://www.linkedin.com/in/yorke-rhodes-iv/) January 31, 2017

Updated by [Joseph Delong](https://www.linkedin.com/in/joseph-delong-b3084332/) April 9, 2018

Updated by [Jay Payne](https://www.linkedin.com/in/jay-payne-174615124/) June 28, 2018



## Prerequisites 


### Install Node.js
```bash
$ sudo apt-get install -y nodejs
```

### Install Node Package Manager (npm)
```bash
$ curl -sL https://deb.nodesource.com/setup_10.x | sudo bash -
$ sudo apt-get install -y nodejs
```


## Install and Setup Ethereum Dev and Test Environment

### Install truffle and verify

[Truffle](https://truffleframework.com/) is the most popular development framework for Ethereum with a mission to make developers life a whole lot easier.
```bash
$ sudo npm install -g truffle

$ truffle
```

### Install ganache-cli and verify

[Ganache CLI](https://github.com/trufflesuite/ganache-cli), part of the Truffle suite of Ethereum development tools, is the command line version of [Ganache](https://github.com/trufflesuite/ganache), your personal blockchain for Ethereum development. 

Ganache CLI uses ethereumjs to simulate full client behavior and make developing Ethereum applications faster, easier, and safer. It also includes all popular RPC functions and features (like events) and can be run deterministically to make development a breeze.

TestRPC was depricated in favor of ganache-cli and you can use ganache-cli in exactly same way you could previously use TestRPC.
```bash
$ sudo npm install -g ganache-cli 

$ ganache-cli
```

### Setup to initialize a default truffle project

To setup the project you need to create an empty directory **simplestore** then go into it and then run `truffle init`. 
```bash
$ mkdir simplestore && cd simplestore
$ truffle init
```

The following files and directory structure should be created in **simplestorage** directory.  
```bash
$ ls -alh 

total 28K
drwxr-xr-x 5 ethdev ethdev 4.0K Jun 28 15:28 .
drwxr-xr-x 4 ethdev ethdev 4.0K Jun 28 15:28 ..
drwxr-xr-x 2 ethdev ethdev 4.0K Jun 28 15:28 contracts
drwxr-xr-x 2 ethdev ethdev 4.0K Jun 28 15:28 migrations
drwxr-xr-x 2 ethdev ethdev 4.0K Jun 28 15:28 test
-rw-r--r-- 1 ethdev ethdev  545 Jun 28 15:28 truffle-config.js
-rw-r--r-- 1 ethdev ethdev  545 Jun 28 15:28 truffle.js
```

### To configure truffle to use our ganache-cli client

Inside the **truffle.js** file the *development* network needs to be created and following variable set:
* host = `localhost`
* port = `8545`
* network_id = `"*", // Match any network id`
* gas = `4600000`

Edit the *modules.export* section of **truffle.js** to add the *development* network.
```bash
module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 8545,
      network_id: "*", // Match any network id
      gas: 4600000
    }
  }
};
```

### Create a Contract

Open a new file in the *contracts* directory called **SimpleStorage.sol**
```bash
$ cat contracts/SimpleStorage.sol

pragma solidity ^0.4.17;

contract SimpleStorage {
  mapping(address => uint256) public favoriteNumbers;

  function setFavorite(uint x) public {
    favoriteNumbers[msg.sender] = x;
  }
}
```

### Compile the new Contract

Run `truffle compile` in the project's root directory. You should receive: 
* helpful warnings 
* notified that a new artifact has been written.

```bash
$ truffle compile

Compiling ./contracts/Migrations.sol...
Compiling ./contracts/SimpleStorage.sol...
Writing artifacts to ./build/contracts
```

### Launch the ganache-cli Ethereum test client

To test deploying and the functionality of the project we will need to run the `ganache-cli` Ethereum test client.
If you do not have the ability to run a second terminal window you can use `screen`

Create a screen session for:
* main project to execute tests
* run the Ethereum test client **ganache-cli**

In either case in the second window or session start the ganache-cli test client

```bash
$ ganache-cli
```


### Deploy the Contract
Configure truffle to deploy the artifact to our network by creating a new file in the *migrations* directory called **2_deploy_contracts.js**. When we run truffle deploy we should see:
* the migration script running
* eventually the address of the contract account it was created on

Make sure you are in the first terminal or screen session and the **simplestorage** directory

```bash
cat migrations/2_deploy_contracts.js

var SimpleStorage = artifacts.require("./SimpleStorage.sol");
module.exports = function(deployer) {
	deployer.deploy(SimpleStorage);
};
```
Now, if we run `truffle deploy` we should see our migration script running, and eventually truffle will tell us the address of  the contract account.
```bash
$ truffle deploy
```

Troubleshooting Tip: if you see any errors check the other terminal or screen session to make sure the test client is running.


### Testing the Contract

Every contract should be tested before every considering deploying it permanently (on a live ethereum network)!

Open a new file in the *test* directory called **simpleStorage.js**.

```bash
cat test/simpleStorage.js

const SimpleStorage = artifacts.require("SimpleStorage") 

contract('SimpleStorage', (accounts) => {
	const [bob, alice] = accounts
	it("should verify bob and alice's favorite numbers default to 0", async () => {
		const ssContract = await SimpleStorage.deployed()
		const bobNum = await ssContract.favoriteNumbers.call(bob)
		assert.equal(bobNum, 0,
			"bob's default value was non-zero")
		
		const aliceNum = await ssContract.favoriteNumbers.call(alice)
		assert.equal(aliceNum, 0,
			"alice's default value was non-zero")
	})

	it("should set bob's favorite number to 33", async () => {
		const ssContract = await SimpleStorage.deployed()
		ssContract.setFavorite(33, {from: bob})
		const newBobNum = await ssContract.favoriteNumbers.call(bob)
		assert.equal(newBobNum, 30,
			"bob's new value was not 33")
	})
})
```

###  Run the Tests
To run the tests created in the previous section run `truffle test`. 
```bash
truffle test
```

### Test results
These are the test results.  One of the tests failed.  Can you find the error, fix it then run the tests again.  
```bash
Using network 'development'.
  Contract: SimpleStorage
    ✓ should verify bob and alice's favorite numbers default to 0 (47ms)
    1) should set bob's favorite number to 33
    > No events were emitted
  1 passing (110ms)
  1 failing

  1) Contract: SimpleStorage
       should set bob's favorite number to 33:
     AssertionError: bob's new value was not 33: expected { Object (s, e, ...) } to equal 30
      at Context.it (test/simpleStorage.js:20:13)
      at process._tickCallback (internal/process/next_tick.js:68:7)
```

After you find the error and fix it all tests should pass.
```bash
Using network 'development'.
  Contract: SimpleStorage
    ✓ should verify bob and alice's favorite numbers default to 0 (59ms)
    ✓ should set bob's favorite number to 33
  2 passing (107ms)
```

### Troubleshooting
If you encounter problems, I have a GitHub repository with a working version of this guide. In your terminal run:

```bash
git clone https://github.com/crashsite/simplestorage
```

If truffle compile or truffle test fails in the newly cloned folder then there may be an environmental issue. Ensure that *NodeJS* and *NPM* are installed correctly.

### Further Links
* [Solidity Documentation](https://solidity.readthedocs.io/en/develop/index.html#)
* [Truffle Documentation](http://truffleframework.com/docs/)


