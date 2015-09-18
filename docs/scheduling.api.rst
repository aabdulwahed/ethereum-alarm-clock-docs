Scheduled Call API
==================

The Alarm service exposes getter functions for all call information that may be
important to those scheduling or executing calls.


Properties of a Scheduled Call
------------------------------

* **bytes32 callKey:** the unique identifier for this function call.
* **address contractAddress:** the address of the contract the function should be called on.
* **address scheduledBy:** the address who scheduled the call.
* **uint calledAtBlock:** the block number on which the function was called.
  (``0`` if the call has not yet been executed.)
* **uint targetBlock:** the block that the function should be called on.
* **uint8 gracePeriod:** the number of blocks after the ``targetBlock`` during
  which it is stll ok to execute the call.
* **uint nonce:** value to differentiate multiple *identical* calls that should
  happen simultaneously.
* **uint baseGasPrice:** the gas price that was used when the call was
  scheduled.
* **uint gasPrice:** the gas price that was used when the call was executed.
  (``0`` if the call has not yet been executed.)
* **uint gasUsed:** the amount of gas that was used to execute the function
  call (``0`` if the call has not yet been executed.)
* **uint payout:** the amount in wei that was paid to the address that executed
  the function call. (``0`` if the call has not yet been executed.)
* **uint fee:** the amount in wei that was kept to pay the creator of the Alarm
  service. (``0`` if the call has not yet been executed.)
* **bytes4 sig:** the 4 byte ABI function signature of the function on the
  ``contractAddress`` for this call.
* **bool isCancelled:** whether the call was cancelled.
* **bool wasCalled:** whether the call was called.
* **bool wasSuccessful:** whether the call was successful.
* **bytes32 dataHash:** the ``sha3`` hash of the data that should be used for
  this call.


Call Key
^^^^^^^^

**bytes32 callKey**

The following functions are available on the Alarm service.  The vast majority
of them take the **callKey** which is an identifier used to reference a
scheduled call.

The **callKey** is computed as ``sha3(scheduledBy, contractAddress, signature, dataHash, targetBlock, gracePeriod, nonce)`` where:

* **scheduledBy:** the ``address`` that scheduled the call.
* **contractAddress:** the ``address`` of the contract that the function should
  be called on when this call is executed.
* **signature:** the ``byte4`` ABI function signature of the function that
  should be called.
* **dataHash:** the ``bytes32`` sha3 hash of the call data that should be used
  for this scheduled call.
* **targetBlock:** the ``uint256`` block number that this call should be executed on.
* **gracePeriod:** the ``uint8`` number of blocks after ``targetBlock`` during
  which it is still ok to execute this scheduled call.
* **nonce:** the ``uint256`` value that can be used to distinguish between
  multiple calls with identical data that should occur during the same time.
  This value only matters if you are registering multiple calls for which all
  of the other fields are the same.

Contract Address
^^^^^^^^^^^^^^^^

**address contractAddress**

The address of the contract that the scheduled function call should be executed
on.  Retrieved with the ``getCallContractAddress`` function.

* **Soldity Function Signature:** ``getCallContractAddress(bytes32 callKey) returns (address)``
* **ABI Signature:** ``0x9c975df``


Scheduled By
^^^^^^^^^^^^

**address scheduledBy**

The address of the contract that the scheduled function call should be executed
on.  Retrieved with the ``getCallScheduledBy`` function.

* **Soldity Function Signature:** ``getCallScheduledBy(bytes32 callKey) returns (address)``
* **ABI Signature:** ``0x8b37e656``

Called at Block
^^^^^^^^^^^^^^^

**uint calledAtBlock**

The block number that this call was executed.  Retrieved with the
``getCallCalledAtBlock`` function. Returns ``0`` if the call has not been
executed yet.

* **Soldity Function Signature:** ``getCallCalledAtBlock(bytes32 callKey) returns (uint)``
* **ABI Signature:** ``0xe4098655``

Grace Period
^^^^^^^^^^^^

**uint8 gracePeriod**

The number of blocks after the ``targetBlock`` that it is still ok to execute
this call.  Retrieved with the ``getCallGracePeriod`` function.

* **Soldity Function Signature:** ``getCallGracePeriod(bytes32 callKey) returns (uint8)``
* **ABI Signature:** ``0x34c19b93``

Target Block
^^^^^^^^^^^^

**uint targetBlock**

The block number that this call should be executed on.  Retrieved with the
``getCallTargetBlock`` function.

* **Soldity Function Signature:** ``getCallTargetBlock(bytes32 callKey) returns (uint)``
* **ABI Signature:** ``0x234917d4``

Base Gas Price
^^^^^^^^^^^^^^

The value of ``tx.gasprice`` that was used to schedule this function call.
Retrieved with the ``getCallBaseGasPrice`` function. Returns ``0`` if the call
has not been executed yet.

* **Soldity Function Signature:** ``getCallBaseGasPrice(bytes32 callKey) returns (uint)``
* **ABI Signature:** ``0x77b19cd5``

Gas Price
^^^^^^^^^

The value of ``tx.gasprice`` that was used to execute this function call.
Retrieved with the ``getCallGasPrice`` function. Returns ``0`` if the call has
not been executed yet.

* **Soldity Function Signature:** ``getCallGasPrice(bytes32 callKey) returns (uint)``
* **ABI Signature:** ``0x78bc6460``

Gas Used
^^^^^^^^^

The amount of gas that was used during execution of this function call.
Retrieved with the ``getCallGasUsed`` function.  Returns ``0`` if the call has
not been executed yet.

* **Soldity Function Signature:** ``getCallGasUsed(bytes32 callKey) returns (uint)``
* **ABI Signature:** ``0x86ae9e4``

Signature
^^^^^^^^^

**bytes4 signature**

The ABI function signature that should be used to execute this function call.
Retrieved with the ``getCallSignature`` function.

* **Soldity Function Signature:** ``getCallSignature(bytes32 callKey) returns (uint)``
* **ABI Signature:** ``0xc88edaed``

Was Called
^^^^^^^^^^

**bool wasCalled**

Boolean flag for whether or not this function has been called yet.  Retrieved
with the ``checkIfCalled`` function.

* **Soldity Function Signature:** ``checkIfCalled(bytes32 callKey) returns (uint)``
* **ABI Signature:** ``0x2a472ae8``



function checkIfSuccess(bytes32 callKey) public returns (bool) {
function checkIfCancelled(bytes32 callKey) public returns (bool) {
function getCallDataHash(bytes32 callKey) public returns (bytes32) {
function getCallPayout(bytes32 callKey) public returns (uint) {
function getCallFee(bytes32 callKey) public returns (uint) {
