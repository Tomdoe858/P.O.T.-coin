
pragma solidity ^0.6.0;

interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function allowance(address owner, address spender) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

contract POTCoin is IERC20 {
    string public name = "P.O.T. COIN";
    string public symbol = "POT";
    uint8 public decimals = 18; // 18 decimals is the default
    uint256 public totalSupply;

    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    address public owner; // Your address
    address public dividendAccount = 0xf6017863ca739c19cfbc8797c99425a7c98fa49b; // Dividend account address
    uint256 public dividendThreshold = 3000; // Threshold for dividend distribution

    constructor(uint256 initialSupply) public {
        totalSupply = initialSupply * 10**uint256(decimals);
        balanceOf[msg.sender] = totalSupply / 2; // Allocate 50% to your address
        balanceOf[dividendAccount] = totalSupply / 2; // Allocate 50% to dividend account
        owner = msg.sender;
    }

    function transfer(address recipient, uint256 amount) external returns (bool) {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;

        // Calculate transaction fee
        uint256 fee = amount / 100; // 1% fee
        uint256 dividendShare = (fee * 60) / 100; // 60% to dividend account
        balanceOf[dividendAccount] += dividendShare;
        balanceOf[owner] += fee - dividendShare; // 30% to owner

        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function distributeDividends() external {
        require(msg.sender == owner, "Only owner can distribute dividends");
        require(balanceOf[dividendAccount] >= dividendThreshold, "Threshold not reached");

        // Distribute dividends to all holders
        // Implement proportional distribution based on token balances

        // Burn remaining 10% of the fee
        uint256 burnAmount = (balanceOf[dividendAccount] * 10) / 100;
        balanceOf[dividendAccount] -= burnAmount;
        totalSupply -= burnAmount;
    }
}
