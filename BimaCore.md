# BimaCore.sol

### What does this contract do?

- Initialization of address
- Transfer of Ownership
- Ownership of this contract should be the Bima DAO via `AdminVoting`. Other ownable Bima contracts inherit their ownership from this contract using `BimaOwnable`.

<details>
<summary> VARIABLES used in this contract </summary>

- `address public feeReceiver` - stores the address which receives fee across the protocol. It is initialized in the constructor but can be modified by the owner.
- `address public priceFeed` - stores the address of price feed across protocol.
- `address  public owner`- address of Owner
- `address public pendingOwner` - address of new owner
- `address ownershipTransferDeadline` - Its used in the `commitTransferOwnership`, it is used to store the time after which the ownership will be transferred.
- `address public guardian`- address of guardian
- `bool public paused` - System-wide pause.
- `Uint256 public immutable startTime` - System-wide start time, rounded down the nearest epoch week.
  Other contracts that require access to this should inherit `SystemStart`.
- `uint256 public constant OWNERSHIP_TRANSFER_DELAY` = 86400 \* 3
  Delay is set to 3 days.
  `Reason` - We enforce a three day delay between committing and applying an ownership change, as a sanity check on a proposed new owner and to give users time to react in case the act is malicious.

</details>

### Difference between `OWNER` and `GUARDIAN`?

As per my limited knowledge
`GUARDIAN` can pause the system but cannot set the address of priceFeed, guardian and feeReceiver, it can be done only via `OWNER`.  
Only `OWNER` can commitTransferOwnership.

<details>
<summary>FUNCTION</summary>
<details>
<summary>Constructor</summary>

```javascript
constructor(address _owner, address _guardian, address _priceFeed, address _feeReceiver) {
        owner = _owner;
        startTime = (block.timestamp / 1 weeks) * 1 weeks;
        guardian = _guardian;
        priceFeed = _priceFeed;
        feeReceiver = _feeReceiver;
        emit GuardianSet(_guardian);
        emit PriceFeedSet(_priceFeed);
        emit FeeReceiverSet(_feeReceiver);
    }

```

**Description**
`Constructor` is just initializing the address of `owner`,`guardian`,`priceFeed`, `startTime` and `feeReceiver`. As well as it is emiting the event when all the parameter is initialized.

</details>

<details>
<summary>setFeeReceiver()</summary>

```javascript
 /**
     * @notice Set the receiver of all fees across the protocol
     * @param _feeReceiver Address of the fee's recipient
     */
    function setFeeReceiver(address _feeReceiver) external onlyOwner {
        feeReceiver = _feeReceiver;
        emit FeeReceiverSet(_feeReceiver);
    }

```

**Description**
Only `owner` can call this function and set the address of `feeReceiver`.

</details>

<details>
<summary>setPriceFeed()</summary>

```javascript
   /**
     * @notice Set the price feed used in the protocol
     * @param _priceFeed Price feed address
     */
    function setPriceFeed(address _priceFeed) external onlyOwner {
        priceFeed = _priceFeed;
        emit PriceFeedSet(_priceFeed);
    }
```

**Description**
This function is mainly involved in setting the price feed address. Only `Owner` can call it.

</details>

<details>
<summary>setGuardian()</summary>

```javascript
 /**
     * @notice Set the guardian address
               The guardian can execute some emergency actions
     * @param _guardian Guardian address
     */
    function setGuardian(address _guardian) external onlyOwner {
        guardian = _guardian;
        emit GuardianSet(_guardian);
    }
```

**Description**
It can be only be called by `Owner`.
It is used to set the address of guardian.

</details>

<details>
<summary>setGuardian()</summary>

```javascript
  /**
     * @notice Set the guardian address
               The guardian can execute some emergency actions
     * @param _guardian Guardian address
     */
    function setGuardian(address _guardian) external onlyOwner {
        guardian = _guardian;
        emit GuardianSet(_guardian);
    }
```

**Description**
Only `Owner` can set it . This function is used to set the address of `guardian`.

</details>

<details>
<summary>setPaused()</summary>

```javascript

    /**
     * @notice Sets the global pause state of the protocol
     *         Pausing is used to mitigate risks in exceptional circumstances
     *         Functionalities affected by pausing are:
     *         - New borrowing is not possible
     *         - New collateral deposits are not possible
     *         - New stability pool deposits are not possible
     * @param _paused If true the protocol is paused
     */
    function setPaused(bool _paused) external {
        require((_paused && msg.sender == guardian) || msg.sender == owner, "Unauthorized");
        paused = _paused;
        if (_paused) {
            emit Paused();
        } else {
            emit Unpaused();
        }
    }
```

**Description**

- This function is used to pause and unpause prtocol. It can only be done via any one `owner` or `guardian`.
- It takes a parameter `bool _paused` (either trur or false).
- Check is done at the begining of fucntion.
- Global vairiable `paused` is updated and event is emited.
</details>

<details>
<summary>commitTransferOwnership()</summary>

```javascript
 function commitTransferOwnership(address newOwner) external onlyOwner {
        pendingOwner = newOwner;
        ownershipTransferDeadline = block.timestamp + OWNERSHIP_TRANSFER_DELAY;

        emit NewOwnerCommitted(msg.sender, newOwner, block.timestamp + OWNERSHIP_TRANSFER_DELAY);
    }
```

**Description**

- `New owner` can be initialized via this function.
- parameter - address of new owner
- `pendingOwner = newOwner` updating the storage variable.
- `ownershipTransferDeadline` setting the deadline for ownership transfer.
- event is emited when all the storage variable is updated.
</details>

<details>
<summary>acceptTransferOwnership</summary>

```javascript
    function acceptTransferOwnership() external {
        require(msg.sender == pendingOwner, "Only new owner");
        require(block.timestamp >= ownershipTransferDeadline, "Deadline not passed");

        emit NewOwnerAccepted(owner, msg.sender);

        owner = pendingOwner;
        pendingOwner = address(0);
        ownershipTransferDeadline = 0;
    }
```

**Description**

- Inorder to execute this function caller should be `newOwner` and time should be passed that is `3 days`.
- when both of this condtion is met new owner is set.
- After setting the new onwer `pendingOwner` is set to `address(0)` and `ownershipTransferDeadline` is set to 0.
</details>

<details>
<summary>revokeTransferOwnership()</summary>

```javascript
 function revokeTransferOwnership() external onlyOwner {
        emit NewOwnerRevoked(msg.sender, pendingOwner);

        pendingOwner = address(0);
        ownershipTransferDeadline = 0;
    }
```

**Description**

- it can be called only by `owner`.
- This function is used to revoke transfer of ownership.
</details>

</details>
