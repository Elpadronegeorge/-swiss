# -swiss erc71
$ rm -f contracts/Lock.sol
xaud@xaud-test-machine:~$ npm install --save-dev @nomicfoundation/hardhat-toolbox
npm WARN deprecated glob@5.0.15: Glob versions prior to v9 are no longer supported
npm WARN deprecated glob@7.1.7: Glob versions prior to v9 are no longer supported

added 257 packages, changed 2 packages, and audited 1061 packages in 1m

163 packages are looking for funding
  run `npm fund` for details

55 vulnerabilities (24 low, 14 moderate, 17 high)

To address issues that do not require attention, run:
  npm audit fix

To address all issues possible (including breaking changes), run:
  npm audit fix --force

Some issues need review, and may require choosing
a different dependency.

Run `npm audit` for details.

read -p "Enter your private key: " PRIVATE_KEY
echo "PRIVATE_KEY=$PRIVATE_KEY" > .env
echo ".env file created."
Creating .env file..
Enter your private key:
.env file created.

echo "Configuring Hardhat..."
cat <<EOL > hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();
Configuring Hardhat...
> 
module.exports = {
  solidity: "0.8.20",
  networks: {
    swisstronik: {
      url: "https://json-rpc.testnet.swisstronik.com/",
      accounts: [\`0x\${process.env.PRIVATE_KEY}\`],
    },
  },
};
EOL
echo "Hardhat configuration completed."
Hardhat configuration completed.
read -p "Enter the token name: " TOKEN_NAME
read -p "Enter the token symbol: " TOKEN_SYMBOL
Enter the token name: XAUDD
Enter the token symbol: XAD


echo "Creating Token.sol contract..."
mkdir -p contracts
cat <<EOL > contracts/Token.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract TestToken is ERC20 {
    constructor()ERC20("$TOKEN_NAME","$TOKEN_SYMBOL"){} 

    function mint100tokens() public {
        _mint(msg.sender, 100*10**18);
    }

    function burn100tokens() public{
        _burn(msg.sender, 100*10**18);
    }
}
EOL
echo "Token.sol contract created."
Creating Token.sol contract...
echo "Compiling the contract..."
npx hardhat compile
echo "Contract compiled."

echo "Creating deploy.js script..."
mkdir -p scripts
cat <<EOL > scripts/deploy.js
const hre = require("hardhat");
const fs = require("fs");

async function main() {
  const contract = await hre.ethers.deployContract("TestToken");
  await contract.waitForDeployment();
  const deployedContract = await contract.getAddress();
  fs.writeFileSync("contract.txt", deployedContract);
  console.log(\`Contract deployed to \${deployedContract}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "deploy.js script created."
Compiling the contract...
Downloading compiler 0.8.20
Compiled 6 Solidity files successfully (evm target: paris).
Contract compiled.
Creating deploy.js script...
deploy.js script created.

echo "Deploying the contract..."
npx hardhat run scripts/deploy.js --network swisstronik
echo "Contract deployed."
Deploying the contract...
Contract deployed to 0x51cEc24DbB6941EF29ffC58C74aB7BDCd058B001
Contract deployed.

echo "Creating mint.js script..."
cat <<EOL > scripts/mint.js
const hre = require("hardhat");
const fs = require("fs");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedTransaction = async (signer, destination, data, value) => {
  const rpcLink = hre.network.config.url;
  const [encryptedData] = await encryptDataField(rpcLink, data);
  return await signer.sendTransaction({
    from: signer.address,
    to: destination,
    data: encryptedData,
    value,
  });
};
Creating mint.js script...
> 
> async function main() {
  const contractAddress = fs.readFileSync("contract.txt", "utf8").trim();
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("TestToken");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "mint100tokens";
  const mint100TokensTx = await sendShieldedTransaction(
    signer,
    contractAddress,
    contract.interface.encodeFunctionData(functionName),
    0
  );
>  await mint100TokensTx.wait();
  console.log("Transaction Receipt: ", \`Minting token has been success! Transaction hash: https://explorer-evm.testnet.swisstronik.com/tx/\${mint100TokensTx.hash}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "mint.js script created."

echo "Minting tokens..."
npx hardhat run scripts/mint.js --network swisstronik
echo "Tokens minted."
mint.js script created.
Minting tokens...
Transaction Receipt:  Minting token has been success! Transaction hash: https://explorer-evm.testnet.swisstronik.com/tx/0x03b5c1b144ab6a570146a77e27e4c0aace4704a4f87a145ac922afb9dee69e85
Tokens minted.

echo "Creating transfer.js script..."
cat <<EOL > scripts/transfer.js
const hre = require("hardhat");
const fs = require("fs");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedTransaction = async (signer, destination, data, value) => {
  const rpcLink = hre.network.config.url;
  const [encryptedData] = await encryptDataField(rpcLink, data);
  return await signer.sendTransaction({
    from: signer.address,
    to: destination,
    data: encryptedData,
    value,
  });
};

Creating transfer.js script...
> async function main() {
  const contractAddress = fs.readFileSync("contract.txt", "utf8").trim();
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("TestToken");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "transfer";
  const amount = 1 * 10 ** 18;
  const functionArgs = ["0x16af037878a6cAce2Ea29d39A3757aC2F6F7aac1", amount.toString()];
  const transaction = await sendShieldedTransaction(
    signer,
    contractAddress,
    contract.interface.encodeFunctionData(functionName, functionArgs),
    0
  );
  await transaction.wait();
  console.log("Transaction Response: ", \`Transfer token has been success! Transaction hash: https://explorer-evm.testnet.swisstronik.com/tx/\${transaction.hash}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "transfer.js script created."
transfer.js script created.
echo "Transferring tokens..."
npx hardhat run scripts/transfer.js --network swisstronik
echo "Tokens transferred."
Transferring tokens...
Transaction Response:  Transfer token has been success! Transaction hash: https://explorer-evm.testnet.swisstronik.com/tx/0xbf8d44a7651a89637a48c3ca29df26c59a875ffd18bae24317b574e60aea8c6d
Tokens transferred.

