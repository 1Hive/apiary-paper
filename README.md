
🛠 Status: *Research* 

**Abstract**: 1Hive's Apiary is a platform for emergent organization built on Aragon. Contributors stake tokens into organizations on the Apiary platform using a bonding curve. Funds held in the bonding curve's reserve pool are released over time into a discretionary pool that the Aragon DAO can use to reward contributors. Splitting funds into reserve and discretionary pools provides smart-contract enforced accountability between project contributors and patrons throughout the lifecycle of a project while simultaneously ensuring sufficient liquidity to support the emergence of a long-tail of micro-organizations.

# 1 Introduction
## 1.1 Background
The ability to collaborate with one another has enabled humanity to rise above the limits of our individual capacity. However, our ability to collaborate is currently limited by our existing systems and institutions. The emergence of smart-contract enabled blockchain platforms like Ethereum can help strip away many of those limitations, reduce information asymmetry and improve trust between individuals without relying on centralized intermediaries.  
## 1.2 Curation Markets
Curation markets can be thought of as markets for voice and attention within communities. At a high level the ICO trend on the Ethereum platform is itself a curation market where individuals become active stakeholders within communities by purchasing project specific tokens. Unfortunately, Ethereum ICOs burden participants with excessive risk due to a lack of accountability and poor liquidity.
## 1.3 Accountability
Tokens are not equities so they do not grant the same privileges and protections of centralized regulatory authorities. One approach to solving this is to only issue securitized-tokens that fit into existing frameworks, but this approach restricts participation to a small-subset of individuals and offers little improvement on our collective ability to collaborate globally. A better approach is to provide mechanisms that improve accountability and transparency so that heavy-handed regulation becomes unnecessary.
## 1.4 Long-tail Liquidity
Market liquidity is an important factor in assessing the risk for an individual participating in a market. However, small markets are typically very illiquid due to a relatively small number of buyers and sellers. In an extreme case all participants may want to buy at the same time or sell at the same time causing extreme volatility. Since organizations may be very small it is important to supplement market liquidity in order to smooth out market volatility and reduce risk to participants.
# 2 Bonding Curves
## 2.1 Automated Market Makers
Bonding Curves use a pricing algorithm to serve as an automated market maker and provide an always available source of liquidity. Depending on the algorithm used markets can be made to be more or less sensitive to price volatility. Users can interact with a bonding curve by staking tokens into the bonding curve’s reserve pool, when they do the bonding curve mints corresponding tokens for the user based on the the pricing algorithm. The newly minted tokens can have specific utility and even be traded among users, but can always be exchanged back through the bonding curve for tokens in the bonding curve’s reserve pool.
## 2.2 Reserve Pool
A reserve pool is necessary for the bonding curve to function as an automated market maker. If funds are removed from the reserve pool it impacts the pricing algorithm, and if the reserve is depleted entirely the bonding curve will no longer function at all. Therefore, it is critical that the process of removing funds from the reserve pool is strictly limited.
### 2.2.1 Withdrawing from the Reserve Pool
For organizations to make use of funds in the reserve to reward contributors the funds must first be withdrawn from the reserve pool and into a discretionary pool. To ensure projects remain accountable to token holders, the rate at which tokens can be withdrawn from the reserve is rate limited and cannot ever go below a floor. For each token accepted by the organizations Bonding Curve the following variables are used to define the withdrawal rate.

| Variable Name | Datatype  | Purpose                                                                                |
|---------------|-----------|----------------------------------------------------------------------------------------|
| tap           | number    | The number of tokens per second which can be withdrawn from the reserve                |
| lastWithdraw  | timestamp | Tracks the block timestamp of the most recent withdrawal                               |
| minRatio      | percent   | The minimum ratio of tokens in the reserve pool relative to outstanding project tokens |

These variables enable the organization to rate-limit the flow of funds from the reserve pool into the discretionary pool, this keeps the project contributors accountable by releasing funds over time and giving individuals an opportunity to exit by drawing down funds in the reserve pool before they are released to the discretionary pool. The minRatio parameter ensures that the Bonding Curve remains a functional source of liquidity for the project.

### 2.2.2 Liquidating the Reserve Pool
In some cases a majority of the token holders will all want to sell at once, this can occur if the project encounters an issue that makes it clear that it should not continue or if key members of the team decide to leave the project. Rather than rushing to exit via the bonding curve, the community can vote to liquidate the reserve pool returning funds to the bonding curves token holders on a pro-rata basis.

## 2.3 Discretionary Pool
Funds which have moved from the bonding curve’s reserve pool to the organization’s discretionary pool can be directly governed by the organization (see section 3 for more details). These funds can be used to reward contributors to the project.

## 2.4 Pricing Algorithm
In order for a bonding curve to function an algorithmic pricing function must be used to determine how many tokens should be minted when tokens are added to the reserve pool and how many tokens from the reserve pool should be given to the user when they return project tokens back to the curve. The pricing formula used by Bancor provides a good starting point.
### 2.4.1 Bancor Pricing Formula
In the Bancor Protocol smart tokens manage by a bonding curve which maintains a constant reserve ratio between tokens held in reserve (Connector Tokens) and the corresponding smart tokens total value (market cap). The ratio between them is called the connector weight (CW).

CW = Connector Balance / (price * Smart Token supply)

We can then algebraically solve for price as follows:

Price = connector balance / (smart token supply x CW)
### 2.4.2 Apiary Pricing Formula
Because the Bancor pricing formula maintains a constant reserve ratio (CW) as tokens are bought and burned, the process of moving funds from the reserve pool into the discretionary pool would decrease CW. To account for that we need to adjust the pricing formula so that CW increases based on a bondPremium parameter as people buy tokens so that CW floats between minRatio and maxRatio values.

When an individual uses the bonding curve to mint tokens and the current CW is less than maxRatio users are required to stake additional tokens relative to the current price, this amount is determined by the bondPremium.

As funds move from the reserve pool to the discretionary pool the CW will decrease, when it reaches minRatio no more withdrawals to the discretionary pool will be permitted.

## 2.5 Front-running
Since the Bonding Curve is managed by a smart contract and transactions are public the mechanism is vulnerable to front-running. These attacks were explored by Ivan Bogatyy along with some possible mitigations. Front-running occurs when someone sees a pending transaction and then makes a new transaction that is included before the pending transaction, changing how the original transaction executes. This can happen if front-runners use a higher gas price, or if they are miners/stakers who are in a position to select and order transactions for block inclusion.
### 2.5.1 minReturn
In the web3.0 UI a minReturn value can be calculated for the user that will cause the trade to fail if the price differs significantly between when the transaction was processed and when the transaction was created. This ensures that users do not accidently purchase a price which is significantly different than expected, but ultimately is only a solution in the cases where users would choose not to buy at the new price and effectively allows front-runners to block transactions.
### 2.5.2 maxGasPrice
Non-miner front-running can be eliminated by requiring a fixed maxGasPrice for transactions accepted by the contract. Since a higher price cannot be used to ensure the front-runner’s transaction occurs first, they are unable to effectively front-run transactions. However, this decision means that in times of high network congestion the contract may become unavailable.
### 2.5.3 Commit-reveal transactions
A more robust solution is to use a commit-reveal scheme where transactions are ordered while hidden, preventing attempts to front-run by both regular users and miners. This requires users to make two transactions adding cost and complexity to the user experience.
### 2.5.4 Apiary approach
Front-running is a legitimate theoretical concern for bonding curve mechanisms but we must weigh the risk versus practical usability.

Specifying a maxGasPrice will prevent front-running by non-block producers and given the possibility of liquidating the reserve pool pro-rata in the event of a black swan event the issue of intermittent illiquidity seems like a reasonable tradeoff for security.

Implementing minReturn can be effective mitigation against front-running by block-producers who are only able to front-run when they are in the position to produce a new block. Additionally as Ethereum moves to proof-of-stake, the incentives of the long-term incentives of block-producers will more closely align with the long-term usability of the network, so sustained malicious front running by block producers may be uncommon.

If despite these mitigations front-running becomes a significant problem more robust solutions such as commit-reveal transactions can be implemented to resolve the issue.
# 3 Governance
Due to the flexibility of aragonOS, projects created on the Apiary platform can have a wide variety of governance structures from a personal DAO where an individual manages a project but is funded and supported by patrons (similar to patreon), to community projects where token holders are expected to actively participate in key governance decisions.

To ensure users have a good understanding of what participation in a project entails it is important for Apiary to have a standard that is common among all projects on the platform.
## 3.1 Project Tokens
Projects listed on the Apiary platform will have a project token which is minted by the projects Bonding Curve (Section 2). These tokens can be used by the project to grant additional project specific perks, utilities, and/or discounts to patrons, but said perks are not guaranteed or endorsed by the Apiary platform. Users of the platform understand and agree that the only guaranteed utility of the project tokens are to support the creators of a project.
## 3.2 Groups and Privileges
Within an organization holders of the project token form a group within the organization called patrons. The organization can use the Access Control List (ACL) provided by aragonOS to grant patrons specific privileges. The set of privileges granted to patrons can be read from the organizations ACL and displayed on the Apiary project page. By providing more attractive privileges to patrons, project organizers can attract more patronage to their project.

Privileges can include voting on how funds in the projects discretionary pool are allocated, signalling for various directions the project organizers could take, or used directly to interact with key pieces of the project such as making challenges on a token curated registry (TCR).

As the ecosystem of aragonOS compatible applications grows the number of privileges organizations can grant and list on the Apiary platform is infinite.
# 4 Project Composition
Projects on Apiary can range in size from large ecosystem or networks to small personal DAOs. In some cases it may make sense for project tokens from one project to be bonded into a sub-project.
## 4.1 Utilizing tokens within reserves
An organization's ability to leverage tokens held in the reserve pool is limited, however, certain token have utility even when they are staked or locked. For example tokens which grant voting privileges can safely be used to vote even while they are locked in the organization reserve pool.

This makes it far more efficient for a project to be funded using governance tokens for related projects than to be funded by tokens such as ETH or DAI which provide no additional utility while held in the organization’s reserve pool.

Since all organization on the platform are built on aragon, the default token used to fund projects is ANT, but project owners can configure a project’s bonding curve to accept other tokens as well, including project tokens generated by other Apiary projects.    
# 5 Summary
By leveraging bonding curves, we provide an incentive for patrons and contributors to support projects at their earliest stages while ensuring that early stage projects are kept accountable throughout their full lifecycle by releasing funds gradually over time. Patrons can use the bonding curve as a source of guaranteed liquidity even for projects that would be too small to otherwise get listed on an exchange.

By integrating aragon, projects can give their patrons a direct voice in the operation and direction of the community. As more aragon compatible applications are created communities on the Apiary platform will have additional opportunities to enable patrons to actively participate in projects.
