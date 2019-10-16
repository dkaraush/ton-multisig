TON Contest Submission
Author: @dkaraush (dkaraush@gmail.com)

Task completed: 
	1) Multi-signature wallet

HOWTO:

1) Creating wallet
	
		fift -s new-wallet.fif <workchain-id> <n> <k> [filebase]

	The script creates:
		- [filebase]-<i>.pk containing private keys
		- [filebase].addr containing address of the wallet in blockchain
		- [filebase]-query.boc containing external message for creating the wallet

	After submitting query file to the blockchain (and sending funds to the address before), smartcontract will store:
		- (32bit unsigned) Message sequence number
		- (8bit unsigned) N - number of keys
		- (8bit unsigned) K - minimal number of required signatures for a transaction
		- (dict) Public keys
		- (dict) Unfinished orders

2) Get-methods
	Smartcontract supports such get-methods:
	- seqno: returns uint32 - message sequence number
	- n: returns uint8 - number of keys
	- k: returns uint8 - number of required signatures for an order
	- orders: returns tuple of tuples:
		- id of the order (uint16)
		- expiration time of the order (uint32)
		- number of already received signatures (uint8)
		- number, which contains exactly whose signatures were received (uint, n bits)
			For example, #0, #1 and #3 signatures were sent. Number will be 1011 => 11.
		- send-mode of internal message (uint8)
		- tuple of destination address
			- workchain id (int8)
			- address (uint256)
		- amount of grams

3) Making new orders and adders
	External messages to smartcontract are divided into 3 groups:
		[mode=0] first message of creation
		[mode=1] creating new order (order request)
		[mode=2] signing already sent orders (adder request)

	Signing the order could be done in two ways:
		1) One person creates a boc file of order request and asks others to sign it. When all signatures will be collected (or partially), boc file can be sent.
			[!] While signing, seqno can be changed and request will not be accepted.

		2) One person creates a boc file of order request and immidiatelly send it to the blockchain. Knowing the id of the order, he/she can ask others to push adder request with their signatures to confirm the order.
			[!] This method will use more gas.

		~) As I wrote, order request could be sent even if not all signatures are collected. After that, adder request to add new signatures should be sent.

	To manage orders (and adders), I have created these scripts:

		[new-order.fif] creates a BOC external message to the wallet to append new order
		[new-adder.fif] creates a BOC external message to the wallet to sign already existed sent order
		[sign.fif] adds signature to a BOC external message (order or adder)
		[info.fif] shows info of a BOC external message

