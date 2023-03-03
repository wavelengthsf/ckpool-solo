# ckpool-solo
Run CKPOOL solo
This was written for a user in KanoPool's Bitcoin discord. Adapted from: https://github.com/edgycorner/CK-Pool-installation-with-Bitcoin-Core-22.0-with-fixes-on-Ubuntu-

**Step 1:**

`sudo apt-get update`

**Step 2:**

Installing git, build-essentials and yasm

`sudo apt install git build-essential yasm sudo libtool`
 
Enter y/yes whenever prompted for response

We will proceed with the installation of CK pool


Clone the ckpool-solo repo with this command:

`git clone https://bitbucket.org/ckolivas/ckpool/src/master/`

*if you get an error saying git not found or installed, run `sudo apt-get install git` and then run the above command


**Step 3:**

move into ckpool directory with following command

`cd master`


**Step 4:**

We will make a few amends to make it work with bitcoin core 22.0, since it's depreciated

`nano src/bitcoin.c`

Search for "JSON failed to decode GBT", you can search with ctrl+w in nano editor

We just have to replace three lines of code, which are


```
if (unlikely(!previousblockhash || !target || !version || !curtime || !bits || !coinbase_aux || !flags)) {
LOGERR("JSON failed to decode GBT %s %s %d %d %s %s", previousblockhash, target, version, curtime, bits, flags);
goto out;
}
```
	
	
	
with the following:

```
if(!flags) {
flags = calloc(1, 1);
}

if (unlikely(!previousblockhash || !target || !version || !curtime || !bits || !coinbase_aux)) {
LOGERR("JSON failed to decode GBT %s %s %d %d %s %s", previousblockhash, target, version, curtime, bits, flags);
goto out;	
 	}
	
```
	

ctlr+x then y then enter

Next, edit ckpool.h and comment out line 357:

Replace:

	ckpool_t *global_ckp; 

With

	//ckpool_t *global_ckp; //Commented to ignore multiple definitions error
	ckpool_t &global_ckp_ref = *global_ckp;


	
dpkg -l pkg-config
if not - sudo apt-get install pkg-config

**Step 5:**

Configue the package

`autoreconf -i`


**Step 6:**

Build

`sudo ./autogen.sh; sudo ./configure; sudo make`


**Step 7:**

Install ckpool so that we can call it from anywhere

`sudo make install`



**Step 8:**

Configure bitcoind rpc and set notifier

We will begin with stopping already running bitcoind

`bitcoin-cli stop`


**Step 9:**

Edit your bitcoin.conf

`cd $HOME/.bitcoin/; nano bitcoin.conf`

add the following lines

```
server=1
rpcuser=user
rpcpassword=pass
rpcallowip=127.0.0.1
rpcauth=user:somelonghashthatyoumadesecure
```

You also need to make sure you have rpcauth setup:
https://jlopp.github.io/bitcoin-core-rpc-auth-generator/
https://github.com/bitcoin/bitcoin/blob/master/share/rpcauth/rpcauth.py

ctlr+x then y then enter


**Step 10:**

Set up a notifier script

`sudo nano /usr/bin/notify.sh`

Copy paste this

```
#!/bin/bash
/usr/bin/notifier -s /opt
```

ctrl+x then press y then enter


**Step 11:**

Start bitcoind again

`bitcoind -daemon -blocknotify=/usr/bin/notify.sh`



**Step 12:**

Edit ckpool.conf to replace serverurl, nodeserver, trusted with "localhost:3336"

it should look like this, where localhost is the address of your server 

```
"serverurl" : [

        "localhost:3333"
],

"nodeserver" : [

        "localhost:3335"
	
],
"trusted" : [

        "localhost:3336"
	
]

```

Edit upstream url too, with localhost:3336

"upstream" : "localhost:3336"

Make sure to edit btc address in the config file, to match yours, and the notifier section.



**Step 13:**

Run the pool

`sudo ckpool -L`

Point your miner to stratum+tcp://<ip of pool>:3333 with any username and password. 
	
But really, don't do this. There is a high risk you'll lose a block if you get astronomically lucky and actually solve one. Mine at kano's pool: kano.is
