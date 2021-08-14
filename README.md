# CryptoWallet
Crypto Coin Wallet is online wallets which use to send coins to recipient and receive coins from various addresses and also display the current status of your wallet. it performs various transfer actions with the current action status.
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^ 0.8.4;

//CryptoCoin is newly mined coin. it sent to the recicver with differnt address
contract CryptoCoinWallet {
    
    //Miner is the owner of the contract which makes all transaction
    address public Miner; //indicate the address of the miner 
    
    constructor() {
        Miner = msg.sender;
    }
    
    // use enum for action status
   enum Action {Transfered, NotTransfered, Deposited, NotDeposited, Used, NotUsed}
   Action Status;
   //Default action is coins NotTransfred
   Action constant defaultAction = Action.NotTransfered;
   
    /* 
    This declares a state variable that
    stores a `Miner` for possible address.
    */
    mapping (address => uint) public AvailableCoins;
    
    //event help to monitor the status of the contracts 
    event Sent_Coin(address from, address to, uint coins);
    
    
    //function to mine the coins and add to the owner account here i.e miner 
    //coins will deposit to the collector address
    address collector;
    function MineCoins(address _collector, uint coins) public {
        collector = _collector;
        //requiremet of Miner should be fullfil otherwise
        //it will throw statement "Only Miner"
        require(msg.sender == Miner ,"Only Miner can Mine");

        //newly mined coins directly added to collector address
        AvailableCoins[collector] += coins;
    }
    
   
   //If collector wallet status is not equal to zero then coins 
   //are transfered otherwise not Transfered
    function CollectorStatus() public view returns(Action) {
       if(AvailableCoins[collector] != 0) {
        return Action.Transfered;
   } else {
       return Action.NotTransfered;
   }
   }
   

    //condition for error when avialble coins are less than requested
    error LessCoins(uint requested, uint Available);
    //condition for error when requestor of coins is different
    error RecipientNotValid();
    //condition for error when available coins in the wallet is less 
    //withdraw quantity
    error LessCoinsInWallet(uint withdraw, uint AvailableInWallet);
    
    
    //send coins to the recipient address 
    address recipient;
    function Send_Coins(address _recipient, uint coins) public {
        recipient = _recipient;
        //revert when Available coins less than requested
        //in this way, contract will show the reason of failure of transaction
        if(coins > AvailableCoins[msg.sender])
         // Errors that describe failures.
        revert LessCoins({
            requested : coins,
            Available : AvailableCoins[msg.sender]
        });
        if(recipient == msg.sender)
        revert RecipientNotValid();
        
        
        
    // deduct coins from collector address 
    AvailableCoins[msg.sender] -= coins;
    
    //credit coins to the recipient address
    AvailableCoins[recipient] += coins;
    
    //emit the contract after complition of transactions
    emit Sent_Coin(msg.sender, recipient, coins);
    }
    
    
    //If recipient wallet status is equal to zero then coins 
    //are not deposited otherwise deposited
    function RecipientStatus() public view returns(Action) {
       if(AvailableCoins[recipient] != 0) {
        return Action.Deposited;
       } else {
           return Action.NotDeposited;
       }
   }
    
    
    //recipient withdraw some coins from his wallet address
    function CoinsUsedByRecipient(address _recipient, uint coins) public {
        recipient = _recipient;
        //revert when when available coins in the wallet is less 
        //withdraw quantity
        if(coins > AvailableCoins[recipient])
         // Errors that describe failures.
        revert LessCoinsInWallet ({
            withdraw : coins,
            AvailableInWallet : AvailableCoins[recipient]
        });
        if(recipient == msg.sender)
        revert RecipientNotValid();
        
        // deduct coins from receiver wallet 
        AvailableCoins[recipient] -= coins;
    }
    
    
    //If collector current wallet status is equal to zero then coins 
    //are not Deposited otherwise Deposited
    function RecipientCurrentStatus() public view returns(Action) {
       if(AvailableCoins[recipient] != 0) {
        return Action.Used;
       } else {
           return Action.NotUsed;
       }
   }
    
    
     // Check remaining coins at collector wallet address
     function DisplayCollectorWallet() external view returns(uint) {
          return AvailableCoins[msg.sender];
     }
     
    // Check remaining coins at recipient wallet address
    function DisplayRecipientWallet() external view returns(uint) {
          return AvailableCoins[recipient];
    }
}
