```

pragma solidity >=0.4.24 <0.7.0;

contract LuckyNumber {
    // currency unit
    uint decimal = 1e8;

    // guess round
    uint round=1;
    // last guess result
    uint lastNum=0;
    // in current round
    uint guessCount=0;
    
    //24 history guess result
    //mapping (uint8 => uint8) public historyResult;
    
    // administrator address
    address admin;
    
    // current round guess data
    mapping (uint => mapping(address => uint)) currentGuess;
    
    // guessRecord
    address payable[] guessList;
    
    modifier isAdmin() {
        require(msg.sender == admin, "Only admin is permited"); 
        _;
    }
    
    constructor() payable public {
        admin = msg.sender;
    }
    
    //fallback () payable external{
    //    return guess();
    // }
    
    // guess value between 1.00 ~ 1.09
    function guess() payable public{
        require(msg.sender != admin, "administrator is not allowed");
        address addr = msg.sender;
        uint value = msg.value;
        require(value > decimal, "min value is 1.0 bty");
        require(value <= decimal+decimal/10, "max value is 1.09 bty");
        
        for (uint i=0;i<10;i++){
            require (currentGuess[i][addr] <= 0, "duplicate guess");
        }

        uint num = value / (decimal/100) % 10;
        currentGuess[num][addr] = value;
        
        //guessList.length = guessCount+1;
        guessList.push();
        guessList[guessCount]=msg.sender;
        guessCount++;
    }
    
    // check my guess
    function checkGuess(address addr) public view returns(uint){
        for (uint i=0;i<10;i++){
            if (currentGuess[i][addr] > 0){
                return i;
            }
        }
        return 999;
    }
    
    function checkMyGuess()  public view returns(uint){
        return checkGuess(msg.sender);
    }
    
    
    function endRound() isAdmin() payable public {
        if(guessCount ==0) {
            return;
        }
        lastNum = guessCount % 10;
        address payable[] memory lucky = new address payable[](guessCount);
        uint index=0;
        mapping(address => uint) storage  data = currentGuess[lastNum];
      

        // find the lucky people
        for(uint idx=0; idx<guessCount;idx++){
            if (data[guessList[idx]] > 0 ){
                lucky[index++] = guessList[idx];
            }
        }
        
        // give bonus
        uint size = index;
        uint totalBonus = guessCount*decimal;
        uint oneBonus = totalBonus/size;
        //address payable receiver; 
        for(uint i=0;i<size;i++){
            lucky[i].transfer(oneBonus);
        }
        
        delete lucky;
    }
    
    function startRound() isAdmin() public {
        // clear current guess record
        for (uint i=0;i<10;i++){
            mapping(address => uint) storage data = currentGuess[i];
            for(uint idx=0; idx<guessCount;idx++){
                delete data[guessList[idx]];
            }
        }
        
        delete guessList;
        guessCount=0;
    }

    function getRound() public view returns(uint) {
        return round;
    }

    function getLastNumber() public view returns(uint) {
        return lastNum;
    }

    function ding() isAdmin() public{
        endRound();
        round++;
        startRound();
    }
}
```
