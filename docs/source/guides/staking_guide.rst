.. _staking-guide:

=======================
NuCypher Staking Guide
=======================

Participation in staking nucypher is divided into two primary roles: "Staker" and "Worker":
The account which holds NU tokens and manages stakes is called a "Staker", and the account
which participates in the network as an Ursula node is called "Worker". The recommended configuration is that
the Staker and Worker have separate Ethereum accounts: Staker controlled by a hardware wallet, and Worker by a software wallet.

All staking-related operations done by Staker are performed through the ``nucypher stake`` command:

.. code:: bash

    (nucypher)$ nucypher stake ACTION [OPTIONS]


**Stake Command Actions**

+----------------------+-------------------------------------------------------------------------------+
| Action               |  Description                                                                  |
+======================+===============================================================================+
|  ``init-stakeholder``| Create a new stakeholder configuration                                        |
+----------------------+-------------------------------------------------------------------------------+
|  ``create``          | Initialize NuCypher stakes (used with ``--value`` and ``--duration``)         |
+----------------------+-------------------------------------------------------------------------------+
|  ``list``            | List active stakes for current stakeholder                                    |
+----------------------+-------------------------------------------------------------------------------+
|  ``accounts``        | Show ETH and NU balances for stakeholder's accounts                           |
+----------------------+-------------------------------------------------------------------------------+
|  ``sync``            | Synchronize stake data with on-chain information                              |
+----------------------+-------------------------------------------------------------------------------+
|  ``set-worker``      | Bond a worker to a staker                                                     |
+----------------------+-------------------------------------------------------------------------------+
|  ``detach-worker``   | Detach worker currently bonded to a staker                                    |
+----------------------+-------------------------------------------------------------------------------+
|  ``collect-reward``  | Withdraw staking compensation from the contract to your wallet                |
+----------------------+-------------------------------------------------------------------------------+
|  ``divide``          | Create a new stake from part of an existing one                               |
+----------------------+-------------------------------------------------------------------------------+
|  ``restake``         | Manage automatic reward re-staking                                            |
+----------------------+-------------------------------------------------------------------------------+
|  ``prolong``         | Prolong an existing stake's duration                                          |
+----------------------+-------------------------------------------------------------------------------+
|  ``winddown``        | Manage winding down of stakes                                                 |
+----------------------+-------------------------------------------------------------------------------+

**Stake Command Options**

+-----------------+--------------------------------------------+
| Option          |  Description                               |
+=================+============================================+
|  ``--value``    | Stake value                                |
+-----------------+--------------------------------------------+
|  ``--duration`` | Stake duration of extension                |
+-----------------+--------------------------------------------+
|  ``--index``    | Stake index                                |
+-----------------+--------------------------------------------+
| ``--hw-wallet`` | Use a hardware wallet                      |
+-----------------+--------------------------------------------+

**Re-stake Command Options**

+-------------------------+---------------------------------------------+
| Option                  |  Description                                |
+=========================+=============================================+
|  ``--enable``           | Enable re-staking                           |
+-------------------------+---------------------------------------------+
|  ``--disable``          | Disable re-staking                          |
+-------------------------+---------------------------------------------+
|  ``--lock-until``       | Enable re-staking lock until release period |
+-------------------------+---------------------------------------------+


Staking Overview
-----------------


Most stakers on a NuCypher testnet will complete the following steps:

1) Install ``nucypher`` on Staker node (See :doc:`/guides/installation_guide`)
2) Install and run Geth, Parity or another ethereum node (can be used with software or hardware Ethereum wallet)
3) Request testnet tokens by joining the `Discord server <https://discord.gg/7rmXa3S>`_ and type ``.getfunded <YOUR_CHECKSUM_ETH_ADDRESS>`` in the #testnet-faucet channel
4) Stake tokens (See Below)
5) Install another Ethereum node at the Worker instance
6) Initialize a Worker node [:ref:`ursula-config-guide`] and bond it to your Staker (``set-worker``)
7) Optionally, enable re-staking
8) Configure and run the Worker, and keep it online [:ref:`ursula-config-guide`]!


Run an Ethereum node for Staking
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Assuming you have ``geth`` installed, let's run a node on the Görli testnet.

.. code:: bash

    $ geth --goerli

If you want to use your hardware wallet, just connect it to your machine. You'll
see something like this in logs:

.. code:: bash

    INFO [08-30|15:50:39.153] New wallet appeared      url=ledger://0001:000b:00      status="Ethereum app v1.2.7 online"

If you see something like ``New wallet appeared, failed to open`` in the logs,
you need to reconnect the hardware wallet (without turning the ``geth`` node
off).

If you don't have a hardware wallet, you can create a software one:

Whilst running the initialized node:

.. code:: bash

    Linux:
    $ geth attach /home/<username>/.ethereum/goerli/geth.ipc
    > personal.newAccount();
    > eth.accounts
    ["0x287a817426dd1ae78ea23e9918e2273b6733a43d"]

    MacOS:
    $ geth attach /Users/<username>/Library/Ethereum/goerli/geth.ipc
    > personal.newAccount();
    > eth.accounts
    ["0x287a817426dd1ae78ea23e9918e2273b6733a43d"]

Where ``0x287a817426dd1ae78ea23e9918e2273b6733a43d`` is your newly created
account address and ``<username>`` is your user.


Interactive Method
------------------

Initialize a new stakeholder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    (nucypher)$ nucypher stake init-stakeholder --provider <PROVIDER> --network <NETWORK_NAME>

If you ran ``geth`` node as above, your ``<PROVIDER>`` is
``ipc:///home/<username>/.ethereum/goerli/geth.ipc``
(on MacOS, ``ipc:///Users/<username>/Library/Ethereum/goerli/geth.ipc``)

``<NETWORK_NAME>`` is the name of the NuCypher network domain where the staker will participate.

.. note:: If you're participating in NuCypher's incentivized testnet, this name is ``cassandra``.

Please note that you want to use ``--hw-wallet`` if you use a hardware wallet in
order for ``nucypher`` to not ask you for the password.

Initialize a new stake
~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    (nucypher)$ nucypher stake create --hw-wallet

    Select staking account [0]: 0
    Enter stake value in NU [15000]: 15000
    Enter stake duration (30 periods minimum): 30

    ============================== STAGED STAKE ==============================

    Staking address: 0xbb01c4fE50f91eF73c5dD6eD89f38D55A6b1EdCA
    ~ Chain      -> ID # 5 | Goerli
    ~ Value      -> 15000 NU (1.50E+22 NuNits)
    ~ Duration   -> 30 Days (30 Periods)
    ~ Enactment  -> 2019-08-19 09:51:16.704875+00:00 (period #18127)
    ~ Expiration -> 2019-09-18 09:51:16.705113+00:00 (period #18157)

    =========================================================================

    * Ursula Node Operator Notice *
    -------------------------------

    By agreeing to stake 15000 NU (15000000000000000000000 NuNits):

    - Staked tokens will be locked for the stake duration.

    - You are obligated to maintain a networked and available Ursula-Worker node
      bonded to the staker address 0xbb01c4fE50f91eF73c5dD6eD89f38D55A6b1EdCA for the duration
      of the stake(s) (30 periods).

    - Agree to allow NuCypher network users to carry out uninterrupted re-encryption
      work orders at-will without interference.

    Failure to keep your node online, or violation of re-encryption work orders
    will result in the loss of staked tokens as described in the NuCypher slashing protocol.

    Keeping your Ursula node online during the staking period and successfully
    producing correct re-encryption work orders will result in rewards
    paid out in ethers retro-actively and on-demand.

    Accept ursula node operator obligation? [y/N]: y
    Publish staged stake to the blockchain? [y/N]: y

    Stake initialization transaction was successful.

    Transaction details:
    OK | deposit stake | 0xe05babab52d00157d0c6e95b7c5165a95adc0ee7ff64ca4d89807805f0ef0fcf (229181 gas)
    Block #16 | 0xbf8252bc84831c26fc91a2272047e394ec0356af515d785d4a179596e722d836

    StakingEscrow address: 0xDe09E74d4888Bc4e65F589e8c13Bce9F71DdF4c7

If you used a hardware wallet, you will need to confirm two transactions here.


List existing stakes
~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    (nucypher)$ nucypher stake list

    ======================================= Active Stakes =========================================

    | ~ | Staker | Worker | # | Value    | Duration     | Enactment
    |   | ------ | ------ | - | -------- | ------------ | -----------------------------------------
    | 0 | 0xbb01 | 0xdead | 0 | 15000 NU | 41 periods . | Aug 04 12:15:16 CEST - Sep 13 12:15:16 CEST
    | 1 | 0xbb02 | 0xbeef | 1 | 15000 NU | 30 periods . | Aug 20 12:15:16 CEST - Sep 18 12:15:16 CEST
    | 2 | 0xbb03 | 0x0000 | 0 | 30000 NU | 30 periods . | Aug 09 12:15:16 CEST - Sep 9 12:15:16 CEST

If the Worker in the list is shown as ``0x0000``, it means that you haven't yet
attached a Worker node to your Staker, so you still have to do it!


Bond an Ursula to a Staker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After you create an Ethereum node for your staker (see below about the worker),
you can set the worker. Stakers which don't yet have a worker will be
highlighted in yellow:

.. code:: bash

    (nucypher)$ nucypher stake set-worker --hw-wallet

    ======================================= Active Stakes =========================================

    | ~ | Staker | Worker | # | Value    | Duration     | Enactment
    |   | ------ | ------ | - | -------- | ------------ | -----------------------------------------
    | 0 | 0xbb01 | 0xdead | 0 | 15000 NU | 41 periods . | Aug 04 12:15:16 CEST - Sep 13 12:15:16 CEST
    | 1 | 0xbb02 | 0xbeef | 1 | 15000 NU | 30 periods . | Aug 20 12:15:16 CEST - Sep 18 12:15:16 CEST
    | 2 | 0xbb03 | 0x0000 | 0 | 30000 NU | 30 periods . | Aug 09 12:15:16 CEST - Sep 9 12:15:16 CEST

    Select Stake: 2
    Enter Worker Address: 0xbeefc4fE50f91eF73c5dD6eD89f38D55A6b1EdCA
    Worker 0xbb04c4fE50f91eF73c5dD6eD89f38D55A6b1EdCA successfully bonded to staker 0xbb03...

    OK!

Please note that the worker's address should be in the format where checksum is encoded
in the address. However, geth shows addresses in the lower case. You can convert
the address to checksum format in geth console:

.. code:: bash

    $ geth attach ~/.ethereum/goerli/geth.ipc
    > eth.accounts
    ["0x287a817426dd1ae78ea23e9918e2273b6733a43d", "0xc080708026a3a280894365efd51bb64521c45147"]
    > web3.toChecksumAddress(eth.accounts[0])
    "0x287A817426DD1AE78ea23e9918e2273b6733a43D"

After this step, you're finished with the Staker, and you can proceed to :ref:`ursula-config-guide`.


Manage automatic reward re-staking
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As your Ursula performs work, all rewards are automatically added to your existing stake to optimize earnings.
This feature, called `re-staking`, is enabled by default.

To disable re-staking:

.. code:: bash

    (nucypher)$ nucypher stake restake --disable

To enable re-staking again:

.. code:: bash

    (nucypher)$ nucypher stake restake --enable


Additionally, you can enable **re-stake locking**, an on-chain commitment to continue re-staking
until a future period (``release_period``). Once enabled, the `StakingEscrow` contract will not
allow **re-staking** to be disabled until the release period begins, even if you are the stake owner.

.. code:: bash

    (nucypher)$ nucypher stake restake --lock-until 12345

No action is needed to release the re-staking lock once the release period begins.


Modifying Active Stakes
~~~~~~~~~~~~~~~~~~~~~~~~

Several administrative operations can be performed on active stakes:

+----------------------+-------------------------------------------------------------------------------+
| Action               |  Description                                                                  |
+======================+===============================================================================+
|  ``divide``          | Create a new stake from part of an existing one                               |
+----------------------+-------------------------------------------------------------------------------+
|  ``prolong``         | Prolong an existing stake's duration                                          |
+----------------------+-------------------------------------------------------------------------------+
|  ``winddown``        | Manage winding down of stakes                                                 |
+----------------------+-------------------------------------------------------------------------------+


Divide
******

To divide an existing stake into discrete substakes:

.. code:: bash

    (nucypher)$ nucypher stake divide


Prolong
*******

To prolong an existing stake's duration:

.. code:: bash

    (nucypher)$ nucypher stake prolong


Wind Down
**********

To start winding down an existing stake:

.. code:: bash

    (nucypher)$ nucypher stake winddown




Collect rewards earned by the staker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

NuCypher nodes earn two types of rewards: staking rewards (in NU) and policy rewards (i.e., service fees in ETH).
To collect these rewards use ``nucypher stake collect-reward`` with flags ``--staking-reward`` and ``--policy-reward``
(or even both).

While staking rewards can only be collected to the original staker account, you can decide which account receives
policy rewards using the ``--withdraw-address <ETH_ADDRESS>`` flag.

.. code:: bash

    (nucypher)$ nucypher stake collect-reward --staking-reward --policy-reward --staking-address 0x287A817426DD1AE78ea23e9918e2273b6733a43D --hw-wallet

     ____    __            __
    /\  _`\ /\ \__        /\ \
    \ \,\L\_\ \ ,_\    __ \ \ \/'\      __   _ __
     \/_\__ \\ \ \/  /'__`\\ \ , <    /'__`\/\`'__\
       /\ \L\ \ \ \_/\ \L\.\\ \ \\`\ /\  __/\ \ \/
       \ `\____\ \__\ \__/.\_\ \_\ \_\ \____\\ \_\
        \/_____/\/__/\/__/\/_/\/_/\/_/\/____/ \/_/

    The Holder of Stakes.

    Collecting 12.345 NU from staking rewards...

    OK | 0xb0625030224e228198faa3ed65d43f93247cf6067aeb62264db6f31b5bf411fa (55062 gas)
    Block #1245170 | 0x63e4da39056873adaf869674db4002e016c80466f38256a4c251516a0e25e547
     See https://goerli.etherscan.io/tx/0xb0625030224e228198faa3ed65d43f93247cf6067aeb62264db6f31b5bf411fa

    Collecting 0.978 ETH from policy rewards...

    OK | 0xe6d555be43263702b74727ce29dc4bcd6e32019159ccb15120791dfda0975372 (25070 gas)
    Block #1245171 | 0x0d8180a69213c240e2bf2045179976d5f18de56a82f17a9d59db54756b6604e4
     See https://goerli.etherscan.io/tx/0xe6d555be43263702b74727ce29dc4bcd6e32019159ccb15120791dfda0975372

You can run ``nuycpher stake accounts`` to verify that your staking compensation
is indeed in your wallet. Use your favorite Ethereum wallet (MyCrypto or Metamask
are suitable) to transfer out the compensation earned (NU tokens or ETH) after
that.

Note that you will need to confirm two transactions if you collect both types of
staking compensation if you use a hardware wallet.


Divide an existing stake
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    (nucypher)$ nucypher stake divide --hw-wallet

    Select Stake: 2
    Enter target value (must be less than or equal to 30000 NU): 15000
    Enter number of periods to extend: 1

    ============================== ORIGINAL STAKE ============================

    Staking address: 0xbb0300106378096883ca067B198d9d98112760e7
    ~ Original Stake: | - | 0xbb03 | 0xbb04 | 0 | 30000 NU | 39 periods . | Aug 09 12:29:44 CEST - Sep 16 12:29:44 CEST


    ============================== STAGED STAKE ==============================

    Staking address: 0xbb0300106378096883ca067B198d9d98112760e7
    ~ Chain      -> ID # 5 | Goerli
    ~ Value      -> 15000 NU (1.50E+22 NuNits)
    ~ Duration   -> 39 Days (39 Periods)
    ~ Enactment  -> 2019-08-09 10:29:49.844348+00:00 (period #18117)
    ~ Expiration -> 2019-09-17 10:29:49.844612+00:00 (period #18156)

    =========================================================================
    Is this correct? [y/N]: y
    Enter password to unlock account 0xbb0300106378096883ca067B198d9d98112760e7:

    Successfully divided stake
    OK | 0xfa30927f05967b9a752402db9faecf146c46eda0740bd3d67b9e86dd908b6572 (85128 gas)
    Block #1146153 | 0x2f87bccff86bf48d18f8ab0f54e30236bce6ca5ea9f85f3165c7389f2ea44e45
    See https://goerli.etherscan.io/tx/0xfa30927f05967b9a752402db9faecf146c46eda0740bd3d67b9e86dd908b6572

    ======================================= Active Stakes =========================================

    | ~ | Staker | Worker | # | Value    | Duration     | Enactment
    |   | ------ | ------ | - | -------- | ------------ | -----------------------------------------
    | 0 | 0xbb01 | 0xbb02 | 0 | 15000 NU | 41 periods . | Aug 04 12:29:44 CEST - Sep 13 12:29:44 CEST
    | 1 | 0xbb01 | 0xbb02 | 1 | 15000 NU | 30 periods . | Aug 20 12:29:44 CEST - Sep 18 12:29:44 CEST
    | 2 | 0xbb03 | 0xbb04 | 0 | 15000 NU | 39 periods . | Aug 09 12:30:38 CEST - Sep 16 12:30:38 CEST
    | 3 | 0xbb03 | 0xbb04 | 1 | 15000 NU | 40 periods . | Aug 09 12:30:38 CEST - Sep 17 12:30:38 CEST


Staking using a preallocation contract
---------------------------------------

Each NuCypher staker with a preallocation will have some amount of tokens locked
in a preallocation contract named ``PreallocationEscrow``, which is used to stake and
perform other staker-related operations.
From the perspective of the main NuCypher contracts, each ``PreallocationEscrow``
contract represents a staker, no different from "regular" stakers.
However, from the perspective of the preallocation user, things are different
since the contract can't perform transactions, and it's the preallocation user
(also known as the "`beneficiary`" of the contract)
who has to perform staking operations.

As part of the preallocation process, beneficiaries receive an allocation file,
containing the ETH addresses of their beneficiary account and corresponding
preallocation contract.

In general, preallocation users can use all staking-related operations offered
by the CLI in the same way as described above, except that they have to specify
the path to the allocation file using the option ``--allocation-filepath PATH``.

For example, to create a stake:

.. code:: bash

    (nucypher)$ nucypher stake create --hw-wallet --allocation-filepath PATH


Or to set a worker:

.. code:: bash

    (nucypher)$ nucypher stake set-worker --hw-wallet --allocation-filepath PATH


As an alternative to the ``--allocation-filepath`` flag, preallocation users
can directly specify their beneficiary and staking contract addresses with the
``--beneficiary-address ADDRESS`` and ``--staking-address ADDRESS``, respectively.

Finally, note that collected staking rewards are always placed in the original
staking account, which for preallocation users is the staking contract.
Run the following command to view the balance of the ``PreallocationEscrow`` contract:

.. code:: bash

    (nucypher)$ nucypher stake preallocation --status --allocation-filepath PATH

    -------------------------- Addresses ---------------------------
    Staking contract: ... 0x0f4Ebe8a28a8eF33bEcD6A3782D74308FC35D021
    Beneficiary: ........ 0x4f5e87f833faF9a747463f7E4387a0d9323a3979

    ------------------------ Locked Tokens -------------------------
    Initial locked amount: 35000 NU
    Current locked amount: 35000 NU
    Locked until: ........ 2020-12-31 16:33:37+00:00

    ---------------------- NU and ETH Balance ----------------------
    NU balance: .......... 17.345 NU
        Available: ....... 12.345 NU
    ETH balance: ......... 0 ETH


To withdraw the unlocked tokens, you need to retrieve them from the
``PreallocationEscrow`` contract using the following command:

.. code:: bash

    (nucypher)$ nucypher stake preallocation --withdraw-tokens --allocation-filepath PATH


.. note:: If you're a preallocation user, recall that you're using a contract to stake.
  Replace ``<YOUR STAKER ADDRESS>`` with the contract address when configuring your node.
  If you don't know this address, you'll find it in the preallocation file.


Inline Method
--------------

+--------------------+----------------+--------------+
| Option             | Flag           | Description  |
+====================+================+==============+
| ``stake value``    | ``--value``    | in NU        |
+--------------------+----------------+--------------+
| ``stake duration`` | ``--duration`` | in periods   |
+--------------------+----------------+--------------+
| ``stake index``    | ``--index``    | to divide    |
+--------------------+----------------+--------------+


Stake 30000 NU for 90 Periods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    (nucypher)$ nucypher stake init --value 30000 --duration 90 --hw-wallet
    ...


Divide stake at index 0, at 15000 NU for 30 additional Periods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    (nucypher)$ nucypher stake divide --index 0 --value 15000 --duration 30 --hw-wallet
    ...

Worker configuration
------------------------

See :ref:`ursula-config-guide`.
