---
PIP:  12
Topic: 一种新型 token 模型
Author: GavinXu
Status: Final
Type: Text
Description: token新标准，兼容EIP-20、EIP3009、EIP-2612。
Created: 2023-05-19
---

# PIP-12：token标准 PRC-12


## 摘要

允许持有者对自己的令牌做委托授权、委托划转的同质化令牌标准接口。

## 动机

现有的兼容EVM系列的同质化令牌标准均借鉴 EIP-20 。EIP-20 成功的主要原因之一在于 transfer、 approve 和 transferFrom 之间的相互作用，这使得代币不仅可以在外部拥有账户 (EOA) 之间转移，还可以用于其他账户通过将合约调用者抽象为令牌访问控制的定义机制。然而对于 approve 和 transferFrom 组合的操作不是特别友好，其原因在于 EIP-20 approve 函数本身是根据调用者 定义的，这意味着用户涉及 EIP-20 代币的初始操作必须由 EOA 执行。如果用户需要与智能合约交互，那么他们需要进行 2 次交易（ approve 和将在内部调用 transferFrom 的智能合约调用）。并且用户本身还是需要持有 ETH 来支付交易 gas 成本。此外，第三方调用者（应用合约）想要操作令牌持有者的令牌时，一定要进行 2 次交易 （ approve 和 transferFrom ）而不能类似于持有者自己调用 transfer 一样操作持有者的令牌。 PRC-12 兼容了主流的 EIP-20、EIP-2612 和 EIP-3009，可以在覆盖原有的 EIP-20 标准同质化令牌的基础上，通过支持元交易的方式由合约的第三方调用者支付 gas 费用并代替令牌的持有者操作持有者的令牌，包括委托授权 permit、委托划转 transferWithAuthorization/receiveWithAuthorization等。



## 规范


除了标准 PRC20 之外，整合了 EIP-2612 和 EIP-3009 的标准接口：

```

// keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)")
bytes32 public constant PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;

// keccak256("TransferWithAuthorization(address from,address to,uint256 value,uint256 validAfter,uint256 validBefore,bytes32 nonce)")
bytes32 public constant TRANSFER_WITH_AUTHORIZATION_TYPEHASH = 0x7c7c6cdb67a18743f49ec6fa9b35f50d52ed05cbed4cc592e13b44501c1a2267;

// keccak256("ReceiveWithAuthorization(address from,address to,uint256 value,uint256 validAfter,uint256 validBefore,bytes32 nonce)")
bytes32 public constant RECEIVE_WITH_AUTHORIZATION_TYPEHASH = 0xd099cc98ef71107a616c4f0f941f04c322d8e254fe26b3c6668db87aae413de8;

// keccak256("CancelAuthorization(address authorizer,bytes32 nonce)")
bytes32 public constant CANCEL_AUTHORIZATION_TYPEHASH = 0x158b0a9edf7a828aad02f63cd515c68ef2f50ba807396f6d12842833a1597429;


event AuthorizationUsed(
    address indexed authorizer,
    bytes32 indexed nonce
);

event AuthorizationCanceled(
    address indexed authorizer,
    bytes32 indexed nonce
);


/**
 * @notice Returns the state of an authorization
 * @dev Nonces are randomly generated 32-byte data unique to the authorizer's
 * address
 * @param authorizer    Authorizer's address
 * @param nonce         Nonce of the authorization
 * @return True if the nonce is used
 */
function authorizationState(
    address authorizer,
    bytes32 nonce
) external view returns (bool);

/**
 * @notice Execute a transfer with a signed authorization
 * @param from          Payer's address (Authorizer)
 * @param to            Payee's address
 * @param value         Amount to be transferred
 * @param validAfter    The time after which this is valid (unix time)
 * @param validBefore   The time before which this is valid (unix time)
 * @param nonce         Unique nonce
 * @param v             v of the signature
 * @param r             r of the signature
 * @param s             s of the signature
 */
function transferWithAuthorization(
    address from,
    address to,
    uint256 value,
    uint256 validAfter,
    uint256 validBefore,
    bytes32 nonce,
    uint8 v,
    bytes32 r,
    bytes32 s
) external;

/**
 * @notice Receive a transfer with a signed authorization from the payer
 * @dev This has an additional check to ensure that the payee's address matches
 * the caller of this function to prevent front-running attacks. (See security
 * considerations)
 * @param from          Payer's address (Authorizer)
 * @param to            Payee's address
 * @param value         Amount to be transferred
 * @param validAfter    The time after which this is valid (unix time)
 * @param validBefore   The time before which this is valid (unix time)
 * @param nonce         Unique nonce
 * @param v             v of the signature
 * @param r             r of the signature
 * @param s             s of the signature
 */
function receiveWithAuthorization(
    address from,
    address to,
    uint256 value,
    uint256 validAfter,
    uint256 validBefore,
    bytes32 nonce,
    uint8 v,
    bytes32 r,
    bytes32 s
) external;


/**
 * @notice Attempt to cancel an authorization
 * @param authorizer    Authorizer's address
 * @param nonce         Nonce of the authorization
 * @param v             v of the signature
 * @param r             r of the signature
 * @param s             s of the signature
 */
function cancelAuthorization(
    address authorizer,
    bytes32 nonce,
    uint8 v,
    bytes32 r,
    bytes32 s
) external;


/**
 * @notice Update allowance with a signed permit
 * @param _owner       Token owner's address (Authorizer)
 * @param spender     Spender's address
 * @param value       Amount of allowance
 * @param deadline    Expiration time, seconds since the epoch
 * @param v           v of the signature
 * @param r           r of the signature
 * @param s           s of the signature
 */
function permit(
    address _owner,
    address spender,
    uint256 value,
    uint256 deadline,
    uint8 v,
    bytes32 r,
    bytes32 s
) external virtual;

/**
 * @notice Nonces for permit
 * @param owner Token owner's address (Authorizer)
 * @return Next nonce
 */
function nonces(address owner) view returns (uint)

/**
 * where DOMAIN_SEPARATOR is defined according to EIP-712. 
 * The DOMAIN_SEPARATOR should be unique to the contract and chain to prevent replay attacks from other domains, 
 * and satisfy the requirements of EIP-712.
 */
function DOMAIN_SEPARATOR() external view returns (bytes32)
```

对于 `transferWithAuthorization`、 `receiveWithAuthorization`、 `cancelAuthorization`和 `permit` 函数的参数 v 、 r 和 s 必须使用 EIP-712 类型的消息签名规范获得。

如： 

```
// Transfer With Authorization
TypeHash := Keccak256(
  "TransferWithAuthorization(address from,address to,uint256 value,uint256 validAfter,uint256 validBefore,bytes32 nonce)"
)
Params := { From, To, Value, ValidAfter, ValidBefore, Nonce }

// ReceiveWithAuthorization
TypeHash := Keccak256(
  "ReceiveWithAuthorization(address from,address to,uint256 value,uint256 validAfter,uint256 validBefore,bytes32 nonce)"
)
Params := { From, To, Value, ValidAfter, ValidBefore, Nonce }

// CancelAuthorization
TypeHash := Keccak256(
  "CancelAuthorization(address authorizer,bytes32 nonce)"
)
Params := { Authorizer, Nonce }

// Permit
TypeHash := Keccak256(
	"Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
)
Params := { Owner, Spender, Value, Nonce, Deadline }
```

```
// "‖" denotes concatenation.
Digest := Keecak256(
  0x1901 ‖ DomainSeparator ‖ Keccak256(ABIEncode(TypeHash, Params...))
)

{ v, r, s } := Sign(Digest, PrivateKey)
```


对于 `transferWithAuthorization`、 `receiveWithAuthorization`、 `cancelAuthorization` 函数来说，nonce 值由链下自行生成唯一的，合约中应当对已经使用或者取消过的nonce做记录，以防止nonce重放：

```
// authorizer address => nonce => bool (true if nonce is used)
mapping(address => mapping(bytes32 => bool)) private _authorizationStates;
```

其中 `transferWithAuthorization` 和 `receiveWithAuthorization` 当且仅当满足以下条件时：

1. 当前出块时间大于 validAfter。
2. 当前出块时间小于 validBefore。
3. nonce 没有被使用或者取消过。
4. from 不是零地址。
5. r 、 s 和 v 是来自消息的 from 的有效 secp256k1 签名。
6. 对于 `receiveWithAuthorization` 的调用者必须等于 to。

如果不满足这些条件中的任何一个， `transferWithAuthorization` 和 `receiveWithAuthorization` 调用必须 revert。

其中 `cancelAuthorization` 当且仅当满足以下条件时： 

1. nonce 没有被使用或者取消过。
2. authorizer 不是零地址。
3. r 、 s 和 v 是来自消息的 authorizer 的有效 secp256k1 签名。

如果不满足这些条件中的任何一个， `cancelAuthorization` 调用必须 revert。

其中 `permit` 当且仅当满足以下条件时：

1. 当前出块时间小于或等于 deadline。
2. owner 不是零地址。
3. nonces[owner] （状态更新前）等于 nonce。
4. r 、 s 和 v 是来自消息的 owner 的有效 secp256k1 签名。

如果不满足这些条件中的任何一个， `permit` 调用必须 revert。




**与支持 EIP-1271 的合约一起使用**


例子：

```
pragma solidity ^0.8.12;

contract Ownable {

    address private _owner;

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    /**
     * @dev Initializes the contract setting the deployer as the initial owner.
     */
    constructor() {
        setOwner(msg.sender);
        emit OwnershipTransferred(address(0), msg.sender);
    }

    /**
     * @dev Returns the address of the current owner.
     */
    function getOwner() public view returns (address) {
        return _owner;
    }

    /**
     * @dev Sets a new owner address
     */
    function setOwner(address newOwner) internal {
        _owner = newOwner;
    }

    /**
     * @dev Throws if called by any account other than the owner.
     */
    modifier onlyOwner() {
        require(_owner == msg.sender, "Ownable: caller is not the owner");
        _;
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Can only be called by the current owner.
     */
    function transferOwnership(address newOwner) public virtual onlyOwner {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        emit OwnershipTransferred(_owner, newOwner);
        setOwner(newOwner);
    }
}



pragma solidity ^0.8.0;

/**
 * @dev Interface of the ERC20 standard as defined in the EIP.
 */
interface IERC20Burnable {
    function burn(uint256 amount) external returns (bool);
    function burnFrom(address account, uint256 amount) external returns (bool);
}


pragma solidity ^0.8.0;

/**
 * @dev Interface of the ERC20 standard as defined in the EIP.
 */
interface IERC20Mintable {
    function mint(address account, uint256 amount) external returns (bool);
}


pragma solidity ^0.8.12;

/**
 * @title ECRecover
 * @notice A library that provides a safe ECDSA recovery function
 */
library ECRecover {
    /**
     * @notice Recover signer's address from a signed message
     * @dev Adapted from: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/65e4ffde586ec89af3b7e9140bdc9235d1254853/contracts/cryptography/ECDSA.sol
     * Modifications: Accept v, r, and s as separate arguments
     * @param digest    Keccak-256 hash digest of the signed message
     * @param v         v of the signature
     * @param r         r of the signature
     * @param s         s of the signature
     * @return Signer address
     */
    function recover(
        bytes32 digest,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal pure returns (address) {
        // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
        // unique. Appendix F in the Ethereum Yellow paper (https://ethereum.github.io/yellowpaper/paper.pdf), defines
        // the valid range for s in (281): 0 < s < secp256k1n ÷ 2 + 1, and for v in (282): v ∈ {27, 28}. Most
        // signatures from current libraries generate a unique signature with an s-value in the lower half order.
        //
        // If your library generates malleable signatures, such as s-values in the upper range, calculate a new s-value
        // with 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141 - s1 and flip v from 27 to 28 or
        // vice versa. If your library also generates signatures with 0/1 for v instead 27/28, add 27 to v to accept
        // these malleable signatures as well.
        if (
            uint256(s) >
            0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0
        ) {
            revert("ECRecover: invalid signature 's' value");
        }

        if (v != 27 && v != 28) {
            revert("ECRecover: invalid signature 'v' value");
        }

        // If the signature is valid (and not malleable), return the signer address
        address signer = ecrecover(digest, v, r, s);
        require(signer != address(0), "ECRecover: invalid signature");

        return signer;
    }
}



pragma solidity ^0.8.12;

import { ECRecover } from "./ECRecover.sol";

/**
 * @title EIP712
 * @notice A library that provides EIP712 helper functions
 */
library EIP712 {
    /**
     * @notice Make EIP712 domain separator
     * @param name      Contract name
     * @param version   Contract version
     * @return Domain separator
     */
    function makeDomainSeparator(string memory name, string memory version)
        internal
        view
        returns (bytes32)
    {
        uint256 chainId;
        assembly {
            chainId := chainid()
        }
        return
            keccak256(
                abi.encode(
                    // keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)")
                    0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f,
                    keccak256(bytes(name)),
                    keccak256(bytes(version)),
                    chainId,
                    address(this)
                )
            );
    }

    /**
     * @notice Recover signer's address from a EIP712 signature
     * @param domainSeparator   Domain separator
     * @param v                 v of the signature
     * @param r                 r of the signature
     * @param s                 s of the signature
     * @param typeHashAndData   Type hash concatenated with data
     * @return Signer's address
     */
    function recover(
        bytes32 domainSeparator,
        uint8 v,
        bytes32 r,
        bytes32 s,
        bytes memory typeHashAndData
    ) internal pure returns (address) {
        bytes32 digest = keccak256(
            abi.encodePacked(
                "\x19\x01",
                domainSeparator,
                keccak256(typeHashAndData)
            )
        );
        return ECRecover.recover(digest, v, r, s);
    }

    function structHash(bytes32 domainSeparator, bytes memory typeHashAndData) internal pure returns (bytes32) {
        return keccak256(
            abi.encodePacked(
                "\x19\x01",
                domainSeparator,
                keccak256(typeHashAndData)
            )
        );
    }
}


pragma solidity ^0.8.12;

import { ECDSA } from "openzeppelin-contracts/contracts/utils/cryptography/ECDSA.sol";
import { IERC1271 } from "openzeppelin-contracts/contracts/interfaces/IERC1271.sol";
import { Address } from "openzeppelin-contracts/contracts/utils/Address.sol";
import { EIP712 } from "./EIP712.sol";


/**
 * @title Signature
 * @notice Processing signature related libraries
 */
library SignatureLib {

    function verifySignature(address owner, bytes32 domainSeparator, uint8 v, bytes32 r, bytes32 s, bytes memory data) internal returns (bool) {


        if (Address.isContract(owner)) {

            bytes memory signature;
            assembly{
                // get free memory address value
                signature := mload(0x40)          // get free memory pointer
                mstore(0x40, add(signature, 97))  // update 0x40 slot value store to freezing memory allocate  ==> (old free_pointer|signature length(32bytes)|r(32bytes)|s(32bytes)|v(1byte)|new free pointer)          
                mstore(signature, 65)             // set value length
                mstore(add(signature, 0x20), r)   // set r
                mstore(add(signature, 0x40), s)   // set s
                mstore8(add(signature, 0x60), v)  // set v
            }
            return IERC1271(owner).isValidSignature(EIP712.structHash(domainSeparator, data), signature) == IERC1271.isValidSignature.selector;
        } else {
            return EIP712.recover(domainSeparator, v, r, s, data) == owner;
        }
    }
}


pragma solidity ^0.8.12;

import { IERC20 } from "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
import { SafeMath } from "openzeppelin-contracts/contracts/utils/math/SafeMath.sol";
import { Address } from "openzeppelin-contracts/contracts/utils/Address.sol";
import { Ownable } from "./Ownable.sol";
import { IERC20Mintable } from "./IERC20Mintable.sol";
import { IERC20Burnable } from "./IERC20Burnable.sol";
import { SignatureLib } from "./SignatureLib.sol";

/**
 * @title FiatToken
 * @dev ERC20 Token backed by fiat reserves
 */
contract FiatTokenV1 is Ownable IERC20Mintable, IERC20Burnable{

    using SafeMath for uint256;


    bytes32 public DOMAIN_SEPARATOR;

    // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)")
    bytes32
        public constant PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;

    // keccak256("TransferWithAuthorization(address from,address to,uint256 value,uint256 validAfter,uint256 validBefore,bytes32 nonce)")
    bytes32
        public constant TRANSFER_WITH_AUTHORIZATION_TYPEHASH = 0x7c7c6cdb67a18743f49ec6fa9b35f50d52ed05cbed4cc592e13b44501c1a2267;

    // keccak256("ReceiveWithAuthorization(address from,address to,uint256 value,uint256 validAfter,uint256 validBefore,bytes32 nonce)")
    bytes32
        public constant RECEIVE_WITH_AUTHORIZATION_TYPEHASH = 0xd099cc98ef71107a616c4f0f941f04c322d8e254fe26b3c6668db87aae413de8;

    // keccak256("CancelAuthorization(address authorizer,bytes32 nonce)")
    bytes32
        public constant CANCEL_AUTHORIZATION_TYPEHASH = 0x158b0a9edf7a828aad02f63cd515c68ef2f50ba807396f6d12842833a1597429;




    string public name;
    string public symbol;
    uint8 public decimals;
    uint256 public totalSupply;
    bool internal _initialized;

    /* Some sets */
    mapping(address => uint256) internal balances;
    mapping(address => mapping(address => uint256)) internal allowed;
    mapping(address => uint256) private _permitNonces;
    mapping(address => mapping(bytes32 => bool)) private _authorizationStates;

    /* events */
    event Mint(address indexed minter, address indexed to, uint256 amount);
    event Burn(address indexed burner, address indexed account, uint256 amount);

    event AuthorizationUsed(address indexed authorizer, bytes32 indexed nonce);
    event AuthorizationCanceled(address indexed authorizer, bytes32 indexed nonce);


    function initialize(
        string memory name,
        string memory symbol,
        uint8 decimals,
        address newOwner
    ) public {
        require(!_initialized, "contract is already initialized");
        require(
            newOwner != address(0),
            "new owner is the zero address"
        );

        name = name;
        symbol = symbol;
        decimals = decimals;
        setOwner(newOwner);
        _initialized = true;

        DOMAIN_SEPARATOR = EIP712.makeDomainSeparator(name, "1");
    }

    function _transfer(
        address from,
        address to,
        uint256 value
    ) internal override {
        require(from != address(0), "transfer from the zero address");
        require(to != address(0), "transfer to the zero address");
        require(value > 0, "transfer amount not greater than 0");
        require(
            value <= balances[from],
            "transfer amount exceeds balance"
        );

        balances[from] = balances[from].sub(value);
        balances[to] = balances[to].add(value);
        emit Transfer(from, to, value);
    }


    function _approve(
        address owner,
        address spender,
        uint256 value
    ) internal override {
        require(owner != address(0), "approve from the zero address");
        require(spender != address(0), "approve to the zero address");
        require(value > 0, "approve amount not greater than 0");
        allowed[owner][spender] = value;
        emit Approval(owner, spender, value);
    }

    function _mint(address to, uint256 amount)
        internal override {
        require(to != address(0), "mint to the zero address");
        require(amount > 0, "mint amount not greater than 0");

        uint256 mintingAllowedAmount = minterAllowed[msg.sender];
        require(
            amount <= mintingAllowedAmount,
            "mint amount exceeds minterAllowance"
        );

        _totalSupply = _totalSupply.add(amount);
        balances[to] = balances[to].add(amount);
        minterAllowed[msg.sender] = mintingAllowedAmount.sub(amount);
        emit Mint(msg.sender, to, amount);
        emit Transfer(address(0), to, amount);
    }

    function _burn(address account, uint256 amount)
        internal override {
        uint256 balance = balances[account];
        require(amount > 0, "burn amount not greater than 0");
        require(amount <= balance, "burn amount exceeds balance");

        _totalSupply = _totalSupply.sub(amount);
        balances[account] = balance.sub(amount);
        emit Burn(msg.sender, account, amount);
        emit Transfer(account, address(0), amount);
    } 

    function _increaseAllowance(
        address owner,
        address spender,
        uint256 increment
    ) internal override {
        _approve(owner, spender, allowed[owner][spender].add(increment));
    }

    function _decreaseAllowance(
        address _owner,
        address spender,
        uint256 decrement
    ) internal override {
        _approve(
            _owner,
            spender,
            allowed[_owner][spender].sub(
                decrement,
                "decreased allowance below zero"
            )
        );
    }

    function _permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal {
        require(deadline >= block.timestamp, "permit is expired");
        bytes memory data = abi.encode(
            PERMIT_TYPEHASH,
            owner,
            spender,
            value,
            _permitNonces[owner]++,
            deadline
        );
        require(
            SignatureLib.verifySignature(owner, DOMAIN_SEPARATOR, v, r, s, data),
            "invalid signature"
        );

        _approve(owner, spender, value);
    }

    function _transferWithAuthorization(
        address from,
        address to,
        uint256 value,
        uint256 validAfter,
        uint256 validBefore,
        bytes32 nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal {
        _requireValidAuthorization(from, nonce, validAfter, validBefore);

        bytes memory data = abi.encode(
            TRANSFER_WITH_AUTHORIZATION_TYPEHASH,
            from,
            to,
            value,
            validAfter,
            validBefore,
            nonce
        );

        require(
            SignatureLib.verifySignature(from, DOMAIN_SEPARATOR, v, r, s, data),
            "invalid signature"
        );

        _markAuthorizationAsUsed(from, nonce);
        _transfer(from, to, value);
    }

    function _receiveWithAuthorization(
        address from,
        address to,
        uint256 value,
        uint256 validAfter,
        uint256 validBefore,
        bytes32 nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal {
        require(to == msg.sender, "caller must be the payee");
        _requireValidAuthorization(from, nonce, validAfter, validBefore);

        bytes memory data = abi.encode(
            RECEIVE_WITH_AUTHORIZATION_TYPEHASH,
            from,
            to,
            value,
            validAfter,
            validBefore,
            nonce
        );
        require(
            SignatureLib.verifySignature(from, DOMAIN_SEPARATOR, v, r, s, data),
            "invalid signature"
        );

        _markAuthorizationAsUsed(from, nonce);
        _transfer(from, to, value);
    }

    function _cancelAuthorization(
        address authorizer,
        bytes32 nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal {
        _requireUnusedAuthorization(authorizer, nonce);

        bytes memory data = abi.encode(
            CANCEL_AUTHORIZATION_TYPEHASH,
            authorizer,
            nonce
        );
        require(
            SignatureLib.verifySignature(authorizer, DOMAIN_SEPARATOR, v, r, s, data),
            "invalid signature"
        );

        _authorizationStates[authorizer][nonce] = true;
        emit AuthorizationCanceled(authorizer, nonce);
    }

    function _requireUnusedAuthorization(address authorizer, bytes32 nonce)
        private
        view
    {
        require(
            !_authorizationStates[authorizer][nonce],
            "authorization is used or canceled"
        );
    }

    /**
     * @notice Check that authorization is valid
     * @param authorizer    Authorizer's address
     * @param nonce         Nonce of the authorization
     * @param validAfter    The time after which this is valid (unix time)
     * @param validBefore   The time before which this is valid (unix time)
     */
    function _requireValidAuthorization(
        address authorizer,
        bytes32 nonce,
        uint256 validAfter,
        uint256 validBefore
    ) private view {
        require(
            validAfter < block.timestamp,
            "authorization is not yet valid"
        );
        require(validBefore > block.timestamp, "authorization is expired");
        _requireUnusedAuthorization(authorizer, nonce);
    }

    /**
     * @notice Mark an authorization as used
     * @param authorizer    Authorizer's address
     * @param nonce         Nonce of the authorization
     */
    function _markAuthorizationAsUsed(address authorizer, bytes32 nonce)
        private
    {
        _authorizationStates[authorizer][nonce] = true;
        emit AuthorizationUsed(authorizer, nonce);
    }

    /**
     * @notice Amount of remaining tokens spender is allowed to transfer on
     * behalf of the token owner
     * @param owner     Token owner's address
     * @param spender   Spender's address
     * @return Allowance amount
     */
    function allowance(address owner, address spender)
        external
        override
        view
        returns (uint256)
    {
        return allowed[owner][spender];
    }

    /**
     * @dev Get totalSupply of token
     */
    function totalSupply() external override view returns (uint256) {
        return _totalSupply;
    }

    /**
     * @dev Get token balance of an account
     * @param account address The account
     */
    function balanceOf(address account)
        external
        override
        view
        returns (uint256)
    {
        return balances[account];
    }

    /**
     * @notice Set spender's allowance over the caller's tokens to be a given
     * value.
     * @param spender   Spender's address
     * @param value     Allowance amount
     * @return True if successful
     */
    function approve(address spender, uint256 value)
        external
        override
        returns (bool)
    {
        _approve(msg.sender, spender, value);
        return true;
    }


    /**
     * @notice Transfer tokens by spending allowance
     * @param from  Payer's address
     * @param to    Payee's address
     * @param value Transfer amount
     * @return True if successful
     */
    function transferFrom(
        address from,
        address to,
        uint256 value
    )
        external
        override
        returns (bool)
    {
        require(
            value <= allowed[from][msg.sender],
            "transfer amount exceeds allowance"
        );

        _transfer(from, to, value);
        allowed[from][msg.sender] = allowed[from][msg.sender].sub(value);
        return true;
    }

    /**
     * @notice Transfer tokens from the caller
     * @param to    Payee's address
     * @param value Transfer amount
     * @return True if successful
     */
    function transfer(address to, uint256 value)
        external
        override
        returns (bool)
    {
        _transfer(msg.sender, to, value);
        return true;
    }

    /**
     * @dev Function to mint tokens
     * @param to The address that will receive the minted tokens.
     * @param amount The amount of tokens to mint. Must be less than or equal
     * to the minterAllowance of the caller.
     */
    function mint(address to, uint256 amount)
        external
        override
        returns (bool)
    {
       _mint(to, amount);
        return true;
    }

    /**
     * @dev allows a burner to burn some of its own tokens
     * Validates that caller is a burner and that sender is not blacklisted
     * amount is less than or equal to the burner's account balance
     * @param amount uint256 the amount of tokens to be burned
     */
    function burn(uint256 amount)
        external
        override
        returns (bool)
    {
        _burn(msg.sender, amount);
        return true;
    }

    /**
     * @dev allows a burner to burn some of another one own tokens
     * Validates that caller is a burner and that sender is not blacklisted
     * amount is less than or equal to the account's account balance
     * and the amount is less than or equal to the allowed amount the account
     * approve to the burner
     * @param account address the account with burned balances
     * @param amount uint256 the amount of tokens to be burned
     */
    function burnFrom(address account, uint256 amount)
        external
        override
        returns (bool)
    {
        require(
            amount <= allowed[account][msg.sender],
            "burn amount exceeds allowance"
        );
        _burn(account, amount);
        allowed[account][msg.sender] = allowed[account][msg.sender].sub(amount);
        return true;
    }


    /**
     * @notice Increase the allowance by a given increment
     * @param spender   Spender's address
     * @param increment Amount of increase in allowance
     * @return True if successful
     */
    function increaseAllowance(address spender, uint256 increment)
        external
        returns (bool)
    {
        _increaseAllowance(msg.sender, spender, increment);
        return true;
    }


    /**
     * @notice Decrease the allowance by a given decrement
     * @param spender   Spender's address
     * @param decrement Amount of decrease in allowance
     * @return True if successful
     */
    function decreaseAllowance(address spender, uint256 decrement)
        external
        returns (bool)
    {
        _decreaseAllowance(msg.sender, spender, decrement);
        return true;
    }

    /**
     * @notice Nonces for permit
     * @param owner Token owner's address (Authorizer)
     * @return Next nonce
     */
    function nonces(address owner) external view returns (uint256) {
        return _permitNonces[owner];
    }

    /**
     * @notice Update allowance with a signed permit
     * @param _owner       Token owner's address (Authorizer)
     * @param spender     Spender's address
     * @param value       Amount of allowance
     * @param deadline    Expiration time, seconds since the epoch
     * @param v           v of the signature
     * @param r           r of the signature
     * @param s           s of the signature
     */
    function permit(
        address _owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    )
        external
    {
        _permit(_owner, spender, value, deadline, v, r, s);
    }

    function authorizationState(address authorizer, bytes32 nonce)
        external
        view
        returns (bool)
    {
        return _authorizationStates[authorizer][nonce];
    }

    /**
     * @notice Execute a transfer with a signed authorization
     * @dev The function may require paying a fee to the transaction intermediary,
     * which will be deducted from the value. When a user calls this function,
     * they need to calculate the value to be transmitted, including the fee, by themselves.     
     * @param from          Payer's address (Authorizer)
     * @param to            Payee's address
     * @param value         Amount to be transferred
     * @param validAfter    The time after which this is valid (unix time)
     * @param validBefore   The time before which this is valid (unix time)
     * @param nonce         Unique nonce
     * @param v             v of the signature
     * @param r             r of the signature
     * @param s             s of the signature
     */
    function transferWithAuthorization(
        address from,
        address to,
        uint256 value,
        uint256 validAfter,
        uint256 validBefore,
        bytes32 nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    )
        external
    {
        _transferWithAuthorization(
            from,
            to,
            value,
            validAfter,
            validBefore,
            nonce,
            v,
            r,
            s
        );
    }

    /**
     * @notice Receive a transfer with a signed authorization from the payer
     * @dev This has an additional check to ensure that the payee's address
     * matches the caller of this function to prevent front-running attacks.
     * @param from          Payer's address (Authorizer)
     * @param to            Payee's address
     * @param value         Amount to be transferred
     * @param validAfter    The time after which this is valid (unix time)
     * @param validBefore   The time before which this is valid (unix time)
     * @param nonce         Unique nonce
     * @param v             v of the signature
     * @param r             r of the signature
     * @param s             s of the signature
     */
    function receiveWithAuthorization(
        address from,
        address to,
        uint256 value,
        uint256 validAfter,
        uint256 validBefore,
        bytes32 nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    )
        external
    {
        _receiveWithAuthorization(
            from,
            to,
            value,
            validAfter,
            validBefore,
            nonce,
            v,
            r,
            s
        );
    }

    /**
     * @notice Attempt to cancel an authorization
     * @dev Works only if the authorization is not yet used.
     * @param authorizer    Authorizer's address
     * @param nonce         Nonce of the authorization
     * @param v             v of the signature
     * @param r             r of the signature
     * @param s             s of the signature
     */
    function cancelAuthorization(
        address authorizer,
        bytes32 nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    )
        external
    {
        _cancelAuthorization(authorizer, nonce, v, r, s);
    }

}

```





## 理由

`transferWithAuthorization`、 `receiveWithAuthorization`、 `cancelAuthorization`和 `permit` 等函数足以使任何涉及 PRC20 代币的操作都可以使用代币本身而不是使用 LAT 来支付。

一个常见用例是中继器代表 PRC20 持有者提交 TransferWithAuthorization、ReceiveWithAuthorization、 CancelAuthorization 和 Permit 等消息。在这种情况下，中继方基本上可以自由选择提交或保留消息的提交 。

## 向后兼容

新合约受益于能够直接利用 PIP-12 来创建原子交易，但现有合约可能仍依赖于传统的 PRC-20 授权模式 ( approve / transferFrom )。为了向使用 PRC-20 配额模式的现有合约（“父合约”）添加对 PIP-12 的支持，可以构建一个转发合约（“转发器”），该合约需要授权并执行以下操作：

1. 从授权中授权或提取用户和存款金额 
2. 调用 receiveWithAuthorization 将指定资金从用户转移到转发器
3. 批准父合约从转发器处花费资金
4. 调用父合约上的方法，该方法花费了转发器设置的配额
5. 将任何生成的令牌的所有权转移回用户


## 安全考虑

尽管 Permit 的签名者可能会考虑由某一方提交他们的交易，但另一方始终可以先运行此交易并在预期方之前调用 permit 。但是， Permit 签名者的最终结果是相同的。

由于 ecrecover 预编译无提示地失败，并且在给出格式错误的消息时仅将零地址返回为 signer ，因此重要的是确保 owner != address(0) 避免 permit 创建批准花费属于零地址的“僵尸资金”。

签名的 Permit 消息是可审查的。中继方总是可以选择在收到 Permit 后不提交它，保留提交它的选项。 deadline 参数是对此的一种缓解措施。如果签名方持有 LAT，他们也可以自己提交 Permit ，这会使之前签名的 Permit 无效。

如果 DOMAIN_SEPARATOR 包含 chainId 并且在合约部署时定义而不是为每个签名重建，则在未来链分裂的情况下，链之间可能存在重放攻击的风险。

从其他智能合约调用时使用 receiveWithAuthorization 而不是 transferWithAuthorization 。监视交易池的攻击者有可能提取转账授权并预先运行 transferWithAuthorization 调用以执行转账，而无需调用包装函数。这可能会导致未处理的锁定存款。 receiveWithAuthorization 通过执行额外的检查来防止这种情况发生，以确保合约调用者 是收款人。此外，如果有多个合约函数接受接收授权，应用程序开发人员可以将 nonce 的一些前导字节用作标识符，以防止交叉使用。

当同时提交多个传输时，请注意中继器和矿工将决定处理它们的顺序。如果交易之间没有相互依赖，这通常不是问题，但是对于高度依赖的交易，建议一次提交一个已签署的授权。

使用 ecrecover 时必须拒绝零地址，以防止从零地址未经授权的转移和资金批准。当提供格式错误的签名时，内置的 ecrecover 返回零地址。