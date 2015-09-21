Call Execution
==============

Call execution is the process through which scheduled calls are executed at
their desired block number.  After a call has been scheduled, it can be executed
by account which chooses to initiate the transaction.  In exchange for
executing the scheduled call, they are paid a small fee of approximately 1% of
the gas cost used for executing the transaction.


Executing a call
----------------

Use the ``doCall`` function to execute a scheduled call.

* **Solidity Function Signature:** ``doCall(bytes32 callKey)``
* **ABI Signature:** ``0xfcf36918``

When this function is called, the following things happen.

1. A few are done to be sure that all of the necessary pre-conditions pass.  If
   any fail, the function exits early without executing the scheduled call:

   * the scheduler has enough funds to pay for the execution.
   * the call has not already been executed.
   * the call has not been cancelled.
   * the current block number is within the range this call is allowed to be
     executed.
2. The necessary funds to pay for the call are put on hold.
3. The call is executed via either the **authorizedAddress** or
   **unauthorizedAddress** depending on whether the scheduler is an authorized
   caller.
4. The gas cost and fees are computed, deducted from the scheduler's account,
   and deposited in the caller's account.


Setting transaction gas and gas price
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is best to supply the maximum allowed gas when executing a scheduled call as
the payment amount for executing the call is porportional to the amount of gas
used.  If the transaction runs out of gas, no payment is issued.

The payment is also dependent on the gas price for the executing transaction.
The lower the gas price supplied, the higher the payment will be.  (though you
should make sure that the gas price is high enough that the transaction will
get picked up by miners).


Getting your payment
^^^^^^^^^^^^^^^^^^^^

Payment for executing a call is deposited in your Alarm service account and can
be withdrawn using the account management api.


Determining what scheduled calls are next
-----------------------------------------

There following functions on the Alarm service facilitate querying for the next
scheduled calls.  None of these functions filter their return values based on
things like whether the scheduled call has been cancelled, but merely serve to
allow querying for calls in order.


getNextBlockWithCall
^^^^^^^^^^^^^^^^^^^^

* **Solidity Function Signature:** ``getNextBlockWithCall(uint blockNumber) returns (uint)``
* **ABI Signature:** ``0xe19eb0dd``

Returns the next block number for the next scheduled call on or after the
provided ``blockNumber``.  If there are no scheduled calls on or after
``blockNumber`` it returns ``0``.

getNextCallKey
^^^^^^^^^^^^^^

* **Solidity Function Signature:** ``getNextCallKey(uint blockNumber) returns (bytes32)``
* **ABI Signature:** ``0xaa6704da``

Returns the call key for the next scheduled call on or after the provided
``blockNumber``.  If there are no scheduled calls on or after ``blockNumber``
it returns ``0x0``;

getNextCallSibling
^^^^^^^^^^^^^^^^^^

* **Solidity Function Signature:** ``getNextCallSibling(bytes32 callKey) returns (bytes32)``
* **ABI Signature:** ``0x22bc71f``

Returns the call key for the next scheduled call that occurs on the same block
as the call referenced by the provided ``callKey``.  Returns ``0x0`` if there
are no subsequent calls on the same block.


Designated Callers
------------------

If the Caller Pool has any bonded callers in the current active pool, then only
designated callers will be allowed to execute a scheduled call.  The exception
to this restriction is the last few blocks within the call's grace period which
the call enters *free-for-all* mode during which anyone may execute it.

If there are no bonded callers in the Caller Pool then the Alarm service will
operate in *free-for-all* mode for all calls meaning anyone may execute any
call at any block during the call window.

How callers designated
^^^^^^^^^^^^^^^^^^^^^^

Each call has a window during which it is allowed to be executed.  This window
begins at the specified ``targetBlock`` and extends through ``targetBlock +
gracePeriod``.   This window is inclusive of it's bounding blocks.

For each 4 block section of the call window, the caller pool associated with
the ``targetBlock`` is selected.  The members of the pool can be though of as a
circular queue, meaning that when you iterate through them, when you reach the
last member, you start back over at the first member.  For each call, a random
starting position is selected and the 4 block sections of the call window are
assigned to the membes of the call pool.

The last two 4 block sections (5-8 blocks depending on the gracePeriod) are not
allocated, but are considered *free-for-all* allowing anyone to call.

Use the ``getDesignatedCaller`` function to determine which caller from the
caller pool has been designated for the block.

* **Solidity Function Signature:** ``getDesignatedCaller(bytes32 callKey, uint targetBlock, uint8 gracePeriod, uint blockNumber) public returns (address)``
* **ABI Signature:** ``0xe8543d0d``

* **callKey:** specifies the scheduled call.
* **targetBlock:** the target block for the specified call.
* **gracePeriod:** the grace period for the specified call.
* **blockNumber:** the block number (during the call window) in question.

This returns the address of the caller who is designated for this block, or
``0x0`` if this call can be executed by anyone on the specified block.

Missing the call window
^^^^^^^^^^^^^^^^^^^^^^^

Anytime a caller fails to execute a scheduled call during the 4 block window
reserved for them, the next caller has the opportunity to claim a portion of
their bond merely by executing the call during their window.  When this
happens, the previous caller who missed their call window has the current
minimum bond amount deducted from their bond balance and transferred to the
caller who executed the call.  The caller who missed their call is also removed
from the pool.  This removal takes 512 blocks to take place as it occurs within
the same mechanism as if they removed themselves from the pool.

Free For All
^^^^^^^^^^^^

When a call enters the last two 4-block chunks of its call window it enters
free-for-all mode.  During these blocks anyone, even unbonded callers, can
execute the call.  The sender of the executing transaction will be rewarded the
bond bonus from all callers who missed their call window.


Safeguards
----------

There are a limited set of safeguards that Alarm protects those executing calls
from.

* Enforces the ability to pay for the maximum possible transaction cost up
  front.
* Ensures that the call cannot cause the executing transaction to fail due to
  running out of gas (like an infinite loop).
* Ensures that the funds to be used for payment are locked during the call
  execution.

Tips for executing scheduled calls
----------------------------------

The following tips may be useful if you wish to execute calls.

Only look in the next 40 blocks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Since calls cannot be scheduled less than 40 blocks in the future, you can
count on the call ordering remaining static for the next 40 blocks.

No cancellation in next 8 blocks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Since calls cannot be cancelled less than 8 blocks in the future, you don't
need to check cancellation status during the 8 blocks prior to its target
block.

Check that it was not already called
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are executing a call after the target block but before the grace period
has run out, it is good to check that it has not already been called.

Check that the scheduler can pay
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is good to check that the scheduler has sufficient funds to pay for the
call's potential gas cost plus fees.
