# Web3

# Resources

## People

### Guy Runs A Start Up

I personally love what I do, I find building on blockchains way more interesting and cool than building yet more web-apps, developer tooling or data pipelines for enterprise. I find the startup culture of web3 is FAR more invigorating than the relatively stale corporate experience of tech nowadays. Your mileage may vary of course.

I work fully remotely and am based in the UAE. I've worked contracting gigs for mid six figures, paid gigs for low six figures + tokens, and gigs paid only in tokens, some of which went to zero, some I made good money on.

In terms of other developers, they fall into a few camps:

- Some genuinely are just following a trend. They don't have a strong understanding of what blockchains or Web3 are, and just dunning kruger hard because they understand some tech. These people can be ignored.
- Some people raise great points. Dan Olsen made some great ones about Web3 culture in Line Goes Up. Molly White writes an excellent critique of Web3 nonsense in web3IsGoingGreat. Some people have had experience in crypto startups and have pointed out that many are shambolic cash grabs. If you want to get into this space: **listen to these people and work hard to try to address the problems they are talking about**:
    - Don't mindlessly follow the crypto crowd.
    - Don't create valueless 'governance' tokens just to dump on retail
    - Don't build a business purely to attract VC capital - build a real business
    - Don't just fork other protocols and call it a day

Take your time, examine the market and work with people (or build something) that you believe adds real value to the space. Right now our big challenges are:

- Privacy
- UX
- Scalability
- Trust of the public

Try and address one of these points and you will find crypto very meaningful.

### Experienced Ex FAANG Guy

- Context
    
    I quit my FAANG job 2 years ago to full time web3 because I believe the technology can bring about a real positive social paradigm shift.
    
    2 years later, I still believe in the tech, but have to acknowledge that the bull market marketing/PR hype literally took the industry two steps backwards. The world sees crypto/web3/nft as a broken promise at best, and a boogie man at worst.
    
    But the future is coming soon(tm). There are still plenty of jobs out there that pay really well, 300k USD/year for example. However to get the job you need relevant past experience and relationships with people in the industry. Also it's all defi regardless of how they sell it. Everything is token & number go up. So it's up to you to decide if that aligns your moral compass or not.
    

There are still plenty of jobs out there that pay really well, 300k USD/year for example. However to get the job you need relevant past experience and relationships with people in the industry. Also it's all defi regardless of how they sell it. Everything is token & number go up. So it's up to you to decide if that aligns your moral compass or not.

In my opinion, you have two paths in web3. 

1. Be either entirely tech focused: build blockchains, love cosmos, obsess over zero knowledge.
2. Be product focused: build smart contracts on evm, learn defi, obsess over tokenomics. The latter makes infinitely more money, but at what cost :'3 

Looking into my crystal ball, if you're starting from 0 I think you're honestly best off learning Go and Solidity and build evm smart contracts/blockchains for large corporation and make salary while the market improves. Or go to conventions, lean into your Senior dev accolades, and network furiously and land a job at some web3 startup that got funding a year ago as you whether the market that way.

### Another Guy

There is no such thing as a web3 engineering. It is just software development all the way down to the rabbit hole. It is TypeScript and Python like in any other software development job.

There are domain-specific concepts like smart contracts, EVM and Ethereum JSON-RPC APIs you need to learn, but this is not different from learning concepts needed in other domain-specific knowledge like XML, machine learning and so on.

Working with web3 for money is not good motivation and it is unlikely you will be paid more than any other software job in the current market situation.

If you like the topic and believe it's helping the world, then it is for you. However, as you have observed things have changed since 2020-2021 and currently it is very popular to be negative about crypto, so you need to tolerate this and be ready to defense your values.

## Pointers

- it is not enough that you know the tech stack, you have to have passion for a specific project to get employed
- it is easy to get a job if you just know the basics of a web3 tech stack
- web3 senior developer roles have a higher bar than web2 ones
- web3 companies don't look for employees in the "classical" way, they find people through open-source
- the fastest way to find a job in web3 is to reach out to higher-up people in companies
- hackathons are a great way to attract attention and get an interview
- even if you win hackathons, that will not get you a flood of offers
- if you have no actual experience you must build portfolio projects
- building portfolio projects is pointless, join a bunch of hackathons instead
- portfolio projects should be really complex and impressive
- even a small portfolio project is fine if you have an original idea and apply the technology to build that idea
- you can be a developer that works on just designing and building smart contracts
- since the industry is relatively new, you have to wear many hats and do stuff that is not pure smart contract development
- many high quality 'crypto' projects are open source, find some and contribute code, get noticed.
- check the websites of some legit crypto companies, they are often hiring. (might be very slow at the moment though)
- start learning Rust, which is getting very popular in the crypto space. (Golang is very common as well)
- web2 frontend stack, i.e. React/Typescript are very popular in web3
- know merkle trees/graph theory, it's very popular algorithms/data structures in 'crypto'
- any new language or technical knowledge you acquire is never "a waste of time"
- My point being you can continue to grow your skills and amass knowledge which will be applicable to many companies including 'web3/crypto' companies.
- Because most of the person's working in the fields are not on Reddit (as far as I read) and tend to use Twitter. You might get something useful on [r/Chainlink](https://www.reddit.com/r/Chainlink/) but not too sure about it.
- Great positive thread to keep motivated
[https://www.reddit.com/r/ethdev/comments/175ekts/looking_for_honest_opinion_on_web3/](https://www.reddit.com/r/ethdev/comments/175ekts/looking_for_honest_opinion_on_web3/)
- 

# Bitcoins

## The Bitcoin Protocol

- **All users can add transactions to the ledger**
Users that want their transactions added must give some fee.
- **Only signed transactions are valid ([Security](Security.md)  → Digital Signatures)**
All users must sign the transaction with their private key to prevent forgery. In addition, each message must have an id to distinguish two transactions that are the same (A pays B C amount of money). This prevents replay attacks.
- **No overspending**
People can refer to the ledger to find out the total debt a user has on the blockchain, and disallow addition of any transactions that cause them to overspend.
- **Trust whichever ledger has the most computational work put into it (Proof Of Work)**
Since anyone can add blocks to the chain, how can you trust a ledger? Trust whoever has put the most computational work into generating their ledger.
If you have conflicting chains, trust whichever one is the longest. The idea is that to have a fake chain that is also the longest, you’ll have to have more than 50% of the computational resources in the blockchain, which is impossible.
Don’t trust a new block immediately, when you hear of it. Wait for newer blocks to be added to the chain. As newer blocks get added to this chain, it increases the legitimacy of this block, to a point where you can now trust this block.

## Bitcoin Blockchain

![Untitled](Web3/Untitled.png)

A block in a blockchain typically consists of the following components.

1. **Block Header:**
    - The block header contains metadata about the block and is crucial for the validation and linking of blocks in the blockchain. It includes:
        - **Version Number:** Indicates the version of the block structure.
        - **Previous Block Hash:** The cryptographic hash of the header of the previous block in the chain, creating a link between blocks.
        - **Merkle Root:** The Merkle root of the transactions in the block. It's a hash that summarises all the transactions, providing a single reference for the entire set.
        - **Timestamp:** The timestamp when the block was created.
        - **Difficulty Target:** A representation of the target difficulty for mining the block. Miners must find a hash value below this target to add a block to the blockchain.
        For bitcoin, this will be the number if prefix bits that need to be zero for a block to be acceptable.
2. **Transaction List:**
    - The block contains a list of transactions that are being added to the blockchain. These transactions represent the transfer of assets or data between participants in the network.
    - Each transaction includes details such as sender, recipient, amount, and a digital signature to verify the authenticity of the transaction.
3. **Nonce:**
    - A nonce (number used once) is a value included in the block header that miners modify during the mining process to find a hash that meets the difficulty target.
    - The nonce introduces an element of randomness, and miners must repeatedly try different nonce values until a valid block hash is found.
4. **Block Size Limit:**
    - A maximum size or weight limit for the block. This limit is in place to control the size of the blockchain and prevent the inclusion of an excessive number of transactions in a single block.
5. **Additional Information (Optional):**
    - Depending on the blockchain protocol, there may be additional fields or information included in the block. For example:
        - **Witness Data:** In the context of segregated witness (SegWit), witness data is separated from the transaction data to increase efficiency.
        - **Coinbase Transaction:** The first transaction in the block is often a special transaction called the coinbase transaction, which rewards the miner with newly created cryptocurrency and transaction fees.
6. **Block Hash:**
    - The block header, including the nonce, is hashed using a cryptographic hash function (e.g., SHA-256) to produce a unique identifier for the block, known as the block hash.
    - The block hash is used to link blocks in the blockchain. Any change in the block's content, even a small one, would result in a completely different hash.

Together, these components form the structure of a block in a blockchain. Once a block is created, miners broadcast it to the network, and other nodes validate its content and link it to the existing blockchain by referencing the previous block's hash in the block header. This process continues, creating a chain of blocks, each linked to its predecessor, forming the blockchain.

## Currency

The ledger is the currency! As long as people don’t end up overspending, or owing too much to each other, we can just get be passing money in between each other without ever settling up with real money! This is the same as having an insanely well balanced splitwise.

## Difficulty Adjustment

Difficulty Adjustment in PoW-based blockchains is a mechanism to dynamically regulate the complexity of the mathematical problem miners must solve. 

This adjustment is made to maintain a steady rate of block creation, ensuring both security and predictability in the blockchain network, even as the number of miners and their collective computational power fluctuates over time.

This is called the Block Time of a block chain. Bitcoin has a block time of around 10 minutes. But other, newer block chains have shorter block times as well. 

## Block Rewards

In bitcoin, all of the money created comes from block rewards. Earlier it used to be 50BTC per block. This reward is halved every 4 years. 

Because this reward decreases geometrically, it guarantees that there will never be more than 21 million bitcoin inexistence.

## Transaction Fees

In addition to block reward, miners can also earn transaction fees. 

Whenever you make a payment, you can optionally choose to include a txn fees which goes to a miner, of whichever block includes you payment.

This is done to incentivize miners to include your transactions in the next block.

In bitcoin, each block is limited to 2400 transactions, which people argue is unneccessarily restrictive.