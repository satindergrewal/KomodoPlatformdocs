**********************
Asset Chain Parameters
**********************

For instructions on how to create an assetchain see :doc:`create-Komodo-Assetchain`

-ac_name
========

This is the ticker symbol for the coin you wish to create. It is recommended to use only numbers and uppercase letters.

-ac_supply
==========

This is the amount of premined coins you would like the chain to have. The node that sets ``-gen`` during the creation process will mine these coins in the genesis block. If this is not set, ``-ac_reward`` must be set, and the default value of 10 coins will be used. If ``-ac_pubkey`` is set, the  premined coins will be mined to the address of the corresponding pubkey. This should be to set to less than ``500000000`` to avoid 64 bit overflows. 

Note: An additional fraction of a coin will be added to this based on the chain's parameters. This is used by nodes to verify the genesis block. For example, the DEX chain's ``-ac_supply`` parameter is set to ``999999``, but in reality the genesis block was ``999999.13521376``.

-ac_reward
==========
This is the block reward for each mined block in satoshis. If this is not set, the block reward will be ``10000`` satoshis, and blocks will be on-demand after block 127 (a new block will not be mined unless there is a transaction in the mempool.)

-ac_end
=======
This is the block height in which block rewards will end. Every block after this height will have 0 block reward.

-ac_halving
===========
This is the number of blocks between each block reward halving. This parameter will have no effect if ``-ac_reward`` is not set. The lowest possible value is ``1440`` (~1 day). If this parameter is set, but ``-ac_decay`` is not, the reward will decrease by 50% each halving. 

-ac_decay
=========
This is the percentage the block reward will decrease by each block reward halving. For example, if this is set to ``750000000``, the block reward will drop 25% from the previous block reward each halving. This parameter will have no effect if ``-ac_reward`` is not set.  
This is the formula that ``-ac_decay`` follows:

.. code-block:: shell

	numhalvings = (height / -ac_halving);
	for (i=0; i<numhalvings; i++)
	reward = (reward * -ac_decay) / 100000000;


-ac_perc
========

The ``-ac_perc`` parameter is the percentage added to the block reward and transactions that will be sent to the ``-ac_pubkey`` address. If this parameter is set, ``-ac_pubkey`` must also be set. For example, if ``-ac_reward=100000000`` and ``-ac_perc=10000000``, for each block mined, the miner receives 1 coin along with the ``-ac_pubkey`` address receiving 0.1 coin. For every transaction sent, the pubkey address will receive 10% of the overall transaction value. This 10% is not taken from the user, rather it is created at this point. Each transaction inflates the overall supply. 

Note: Vout 1 of each coinbase transaction must be the correct amount sent to the corresponding pubkey. The ``vout`` type for all coinbase vouts must be ``pubkey`` as opposed to ``pubkeyhash``. This only affects a miner trying to use a stratum. Z-nomp is currently incompatible.  

-ac_pubkey
==========

If ``-ac_pubkey`` is set, but ``-ac_perc`` is not, this simply means the genesis block will be mined to the set pubkey's address. This must be set to a 33 byte hex string. You can get the pubkey of an address by using the ``validateaddress`` command in ``komodod``. The address must be imported to the wallet before using ``validateaddress``.

-ac_cc
======

This is the network cluster on which the chain can interact with other chains via cross chain smart contracts. This functionality is still in testing. If this is set to 1, the chain will have smart contracts enabled, but it will not be able to interact with other chains. If this is set to any number other than 0 or 1, the chain can interact with other chains on the same network cluster. For example, all ``-ac_cc=2`` chains can interact with each other but may not interact with ``-ac_cc=3`` chains. 
If you'd like to explicitly disable smart contracts set this value to ``0``. 

-ac_staked
==========

This is the percentage of blocks the chain will aim to have as POS. For example, a ``ac_staked=90`` chain will have 90% POS blocks/10% POW blocks. This isn't exact, but the POW difficulty will automatically adjust based on the overall percentage of POW mined blocks.

Each staked block will have an additional transaction added to the coinbase in which the coins that staked the block are sent back to the same address. This is used to verify which coins staked the block, and this allows for compatibility with existing Komodo infrastructure such as Agama, BarterDEX and explorers. If ``-ac_staked`` is used in conjunction with ``-ac_perc``, the ``-ac_pubkey`` address will receive slightly more coins for each staked block compared to a mined block because of this extra transaction.

In order to stake, you must use the ``-pubkey`` option while launching the daemon. The address associated with this pubkey must have coins that are able to be staked. The privkey for this address must be imported to the daemon.

The following are the (current) rules for staking a block:

	#. Block timestamps are used as the monotonically increasing timestamp. It is important to have a synced system clock.

	#. In order to stake ``-pubkey`` must be set while starting the daemon. This pubkey's address must be imported to the daemon and have coins that are able to be staked. The ``validateaddress`` command can be used to get the pubkey of an address.

	#. A utxo is not eligible without ``nLockTime`` set and until 6000 seconds has passed from this lock time. ``(100 * expected blocktimes) to be exact``

	#. There are 64 different segments(``segids``) of addresses, based on the hash of the destination address. ``((nHeight + addrhash.uints[0]) & 0x3f)`` The segid of an address can be found with the ``validateaddress`` command. Each segid will take turns being segid0 at each height. ``(height % 64) = the segid0 for that height.`` All other segid will adjust the elapsed time by ``segid`` seconds.

	#. A new block is eligible to be staked 1 second after median blocktime. For example, segid0 for a given height will be eligible to submit a block 1 second after median blocktime, whereas segid1 will be eligible to submit a block 2 seconds after median blocktime. For the next block, segid0 from the previous block will now be segid63 and will be eligible to submit a block 64 seconds after median blocktime. This means by 64 seconds after the median blocktime, all segids are eligible.

	#. Coinage calculated from the adjusted time is used to divide hash(address + pastblockhash) to create the value compared against the difficulty to determine if a block is won or not. This means a UTXO is more likely to win a block within a segid based on age of the UTXO and amount of coins.

To create a chain using this parameter, start the first node with ``-pubkey`` and without ``-gen``. Start the second node with with ``-gen`` and without ``-pubkey``. Wait until the second node mines block 1 and 2. Now, send coins from the second node to the first node's pubkey address. On the first node, import the privkey for the set pubkey and type the command ``setgenerate true 1``. The node will begin to stake. If the chain has a very high precentage for POS, it's important to do ``setgenerate false`` on the mining node immediately after block 2 is mined. Send the premined coins to the staking node's pubkey address then do ``setgenerate true`` on the both nodes.



.. [credit] 
          Document written by Alright based on previous guides by siu and PTYX. Please send any critiques to Alright on matrix, slack or discord.
