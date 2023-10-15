Smart Contract

We are using Ownable.sol from Openzeppelin which helps us manage the Ownership of a contract. Ownable also let transferOwnership from the owner account to a new one, andrenounceOwnership for the owner to relinquish this administrative privilege, a common pattern after an initial stage with centralized administration is over.

Also we are using an extension of ERC721 known as ERC721 Enumerable. ERC721 Enumerable helps you to keep track of all the tokenIds in the contract and also the tokensIds held by an address for a given contract.

To build the smart contract we are using Hardhat. Hardhat is an Ethereum development environment and framework designed for full stack development in Solidity. 

Creating a new file inside and call it IWhitelist.sol

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.4;

    interface IWhitelist {
        function whitelistedAddresses(address) external view returns (bool);
    }
    
Now lets create a new file and call it CryptoDevs.sol

  // SPDX-License-Identifier: MIT
  pragma solidity ^0.8.4;

  import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
  import "@openzeppelin/contracts/access/Ownable.sol";
  import "./IWhitelist.sol";

  contract CryptoDevs is ERC721Enumerable, Ownable {
      /**
       * @dev _baseTokenURI for computing {tokenURI}. If set, the resulting URI for each
       * token will be the concatenation of the `baseURI` and the `tokenId`.
       */
      string _baseTokenURI;

      //  _price is the price of one Crypto Dev NFT
      uint256 public _price = 0.01 ether;

      // _paused is used to pause the contract in case of an emergency
      bool public _paused;

      // max number of CryptoDevs
      uint256 public maxTokenIds = 20;

      // total number of tokenIds minted
      uint256 public tokenIds;

      // Whitelist contract instance
      IWhitelist whitelist;

      // boolean to keep track of whether presale started or not
      bool public presaleStarted;

      // timestamp for when presale would end
      uint256 public presaleEnded;

      modifier onlyWhenNotPaused {
          require(!_paused, "Contract currently paused");
          _;
      }

      /**
       * @dev ERC721 constructor takes in a `name` and a `symbol` to the token collection.
       * name in our case is `Crypto Devs` and symbol is `CD`.
       * Constructor for Crypto Devs takes in the baseURI to set _baseTokenURI for the collection.
       * It also initializes an instance of whitelist interface.
       */
      constructor (string memory baseURI, address whitelistContract) ERC721("Crypto Devs", "CD") {
          _baseTokenURI = baseURI;
          whitelist = IWhitelist(whitelistContract);
      }

      /**
      * @dev startPresale starts a presale for the whitelisted addresses
       */
      function startPresale() public onlyOwner {
          presaleStarted = true;
          // Set presaleEnded time as current timestamp + 5 minutes
          // Solidity has cool syntax for timestamps (seconds, minutes, hours, days, years)
          presaleEnded = block.timestamp + 5 minutes;
      }

      /**
       * @dev presaleMint allows a user to mint one NFT per transaction during the presale.
       */
      function presaleMint() public payable onlyWhenNotPaused {
          require(presaleStarted && block.timestamp < presaleEnded, "Presale is not running");
          require(whitelist.whitelistedAddresses(msg.sender), "You are not whitelisted");
          require(tokenIds < maxTokenIds, "Exceeded maximum Crypto Devs supply");
          require(msg.value >= _price, "Ether sent is not correct");
          tokenIds += 1;
          //_safeMint is a safer version of the _mint function as it ensures that
          // if the address being minted to is a contract, then it knows how to deal with ERC721 tokens
          // If the address being minted to is not a contract, it works the same way as _mint
          _safeMint(msg.sender, tokenIds);
      }

      /**
      * @dev mint allows a user to mint 1 NFT per transaction after the presale has ended.
      */
      function mint() public payable onlyWhenNotPaused {
          require(presaleStarted && block.timestamp >=  presaleEnded, "Presale has not ended yet");
          require(tokenIds < maxTokenIds, "Exceed maximum Crypto Devs supply");
          require(msg.value >= _price, "Ether sent is not correct");
          tokenIds += 1;
          _safeMint(msg.sender, tokenIds);
      }

      /**
      * @dev _baseURI overides the Openzeppelin's ERC721 implementation which by default
      * returned an empty string for the baseURI
      */
      function _baseURI() internal view virtual override returns (string memory) {
          return _baseTokenURI;
      }

      /**
      * @dev setPaused makes the contract paused or unpaused
       */
      function setPaused(bool val) public onlyOwner {
          _paused = val;
      }

      /**
      * @dev withdraw sends all the ether in the contract
      * to the owner of the contract
       */
      function withdraw() public onlyOwner  {
          address _owner = owner();
          uint256 amount = address(this).balance;
          (bool sent, ) =  _owner.call{value: amount}("");
          require(sent, "Failed to send Ether");
      }

       // Function to receive Ether. msg.data must be empty
      receive() external payable {}

      // Fallback function is called when msg.data is not empty
      fallback() external payable {}
  }

Website

To develop the website we are using React and Next Js. React is a javascript framework which is used to make websites and Next Js is built on top of React.

First, We need to create a new next app. 

     - NFT-Collection
       - hardhat-tutorial
       - my-app

Execute these commands in the terminal

    cd my-app
    npm run dev
    
Our app is running on http://localhost:3000

Now lets install Web3Modal library(https://github.com/Web3Modal/web3modal). Web3Modal is an easy-to-use library to help developers add support for multiple providers in their apps with a simple customizable configuration. By default Web3Modal Library supports injected providers like (Metamask, Dapper, Gnosis Safe, Frame, Web3 Browsers, etc).



Our Crypto Devs NFT dapp is working!

