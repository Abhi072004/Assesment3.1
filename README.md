# Decentralized Auction Management System
A Decentralized Auction Management System is a blockchain-based application designed to conduct auctions in a transparent, secure, and decentralized manner. The core idea is to leverage the properties of blockchain technology, such as immutability, transparency, and decentralization, to create an auction platform that is free from centralized control and manipulation.

## Description
Solidity programming language is used to write this program. Solidity is mostly used to build smart contracts. In this program we have created a smart contract named as DeAuction which is the abbreviation of decentralized auction. In this contract we have created number of events, modifiers and functions. Different different events and modifiers are used with different different functions based on their functionality. For this project we have created a Auction entity whose basically use is to hold the specification of the item listed to sell. It's specification like name, id, base price, current highest bid, seller address, and its ending time. In this project we have created five functions for different purposes. First function is used to listing an item for sell with the base price(>0), and initial highest bidder as 0x000... . Second function is basically used to place a bid on a particular item whose id should have been given by the buyer. The main purpose of third function is explicitly end the auction and this function can be invoked by the seller of that item. Fourth function is used to verify the state of auction basically the main motive to creating this function wat that buyer might be willing to check the status of auction before try to place a bid. The last function is used to invalidate the auction. It is basically used to invalidate an auction which is mistakenly listed by the seller. If someone had already placed the bid and then the seller wants to invalidate his/her auction item then the bid amount will be refunded to the buyer and all the item's parameter will be set to its initial state and the ended variable will be true.

## Getting Started

### Executing Program
To execute this program you can go to the Remix IDE which is open IDE for Solidity. For that you can click on this link https://remix.ethereum.org/

Once you successfully reached to the IDE create a new file by clicking on the '+' icon and saving that file with .sol extension (e.g. DecentralizedAuction.sol). After this copy and paste the code in your file that is given below 

```javascript
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;
contract DeAuction {
    struct Auction {
        uint256 id;
        address payable seller;
        string item;
        uint256 basePrice;
        uint256 highestBid;
        address payable highestBidder;
        uint256 endTime;
        bool ended;
    }
    uint256 public itemsCount = 0;
    mapping(uint256 => Auction) public auctions;
    event AuctionCreated(uint256 id, address seller, string item, uint256 startPrice, uint256 endTime);
    event NewBidPlaced(uint256 auctionId, address bidder, uint256 amount);
    event AuctionEnded(uint256 auctionId, address winner, uint256 amount);
    modifier onlySeller(uint256 auctionId) {
        require(auctions[auctionId].seller == msg.sender, "Only the owner is allowed to end this auction");
        _;
    }
    modifier auctionExists(uint256 auctionId) {
        require(auctionId > 0 && auctionId <= itemsCount, "The auction was never existed");
        _;
    }
    modifier auctionNotEnded(uint256 auctionId) {
        require(!auctions[auctionId].ended, "Auction has already been ended");
        _;
    }
    modifier auctionInProgress(uint256 auctionId) {
        require(block.timestamp < auctions[auctionId].endTime, "Auction has already been ended");
        _;
    }
    function createAuction(string memory item, uint256 basePrice, uint256 duration) public {
        require(bytes(item).length > 0, "Item name cannot be empty");
        require(basePrice > 0, "Starting price must be greater than 0");
        require(duration > 60, "Duration must be greater than 1 minute");
        itemsCount++;
        auctions[itemsCount] = Auction({
            id: itemsCount,
            seller: payable(msg.sender),
            item: item,
            basePrice: basePrice,
            highestBid: basePrice,
            highestBidder: payable(address(0)),
            endTime: block.timestamp + duration,
            ended: false
        });
        emit AuctionCreated(itemsCount, msg.sender, item, basePrice, block.timestamp + duration);
    }
    function placeBid(uint256 auctionId) public payable auctionExists(auctionId) auctionNotEnded(auctionId) auctionInProgress(auctionId) {
        Auction storage auction = auctions[auctionId];
        if(msg.value <= auction.highestBid){
            revert("Your bid can't be less than highest bid");
        } 
        if (auction.highestBidder != address(0)) {
            // Refund the previous highest bidder
            auction.highestBidder.transfer(auction.highestBid);
        }
        auction.highestBid = msg.value;
        auction.highestBidder = payable(msg.sender);
        emit NewBidPlaced(auctionId, msg.sender, msg.value);
    }
    function endAuction(uint256 auctionId) public auctionExists(auctionId) onlySeller(auctionId) auctionNotEnded(auctionId) auctionInProgress(auctionId){
        Auction storage auction = auctions[auctionId];
        auction.ended = true;
        if (auction.highestBidder != address(0)) {
            auction.seller.transfer(auction.highestBid);
        }
        emit AuctionEnded(auctionId, auction.highestBidder, auction.highestBid);
    }
    function checkAuctionState(uint256 auctionId, bool expectedEndedStatus) public view auctionExists(auctionId) {
        Auction storage auction = auctions[auctionId];
        assert(auction.ended == expectedEndedStatus);
    }
    function invalidateAuction(uint256 auctionId) public auctionExists(auctionId) onlySeller(auctionId) auctionNotEnded(auctionId){
        Auction storage auction = auctions[auctionId];
        auction.ended = true;
        if (auction.highestBidder != address(0)) {
            auction.highestBidder.transfer(auction.highestBid);
        }
        auction.highestBid = 0;
        auction.highestBidder= payable(address(0));
    }
}
```

After pasting this code you have to compile the code from the left hand sidebar. click on the 'Solidity Compiler' then click on the 'Compile DecentralizedAuction.sol' button.

After the successful compilation of the code you have to deploy the program. For that you have again another option on the left sidebar that is 'Deploy & Run Transactions' and then you will see a deploy button; before clicking on it make sure that the file showing there is 'DecentralizedAuction.sol'. Then you will be able to see the file in the 'Deployed/Unpinned Contracts' click on that now all the public variable and functions are visible to you now execute and fetch the values according to you.


## Authors
Abhishek

## License
This project is licensed under the MIT License.
