# Foundry_de-problemList

## Problem

### 1. Download Foundry Problem

如果在下載過程中像我一樣遇到以下錯誤：
```javascript
error: linker link.exe not found
  |
  = note: program not found

note: the msvc targets depend on the msvc linker but link.exe was not found

note: please ensure that VS 2013, VS 2015, VS 2017 or VS 2019 was installed with the Visual C++ option

error: could not compile proc-macro2 due to previous error
warning: build failed, waiting for other jobs to finish...
error: failed to compile `foundry-cli v0.1.0 (https://github.com/gakonst/foundry#d66f9d58)`, intermediate artifacts can be found at C:\Users\qazws\AppData\Local\Temp\cargo-installe6Rd6Y

Caused by:
  build failed
```

只要下載 [ Visual Studio 2019 Build tools](https://www.blogger.com/null)，選擇 C++ Build Tools 然後重開機就可以解決了！下載大小約是 6 GB。

### 2. Functional Inheritance Problem

在使用某些繼承來的函式時，要確保被繼承裡面的參數或變數受體為何，簡單來說就是用合約去測試合約（Foundry）的時候，如果沒有實作 `_checkOnERC721Received` 的話，在合約裡面直接宣告 `_safeMint()` 以後會出現以下錯誤：
```
Running 1 test for CarManTest.json:CarManTest
�[31m[FAIL. Reason: ERC721: transfer to non ERC721Receiver implementer]�[0m testDeployerCanMint() (gas: 192214)

Failed tests:
�[31m[FAIL. Reason: ERC721: transfer to non ERC721Receiver implementer]�[0m testDeployerCanMint() (gas: 192214)

Encountered a total of �[31m1�[0m failing tests, �[32m0�[0m tests succeeded
```

`_checkOnERC721Received` 有一個 verification logic 存在，如果今天 `to` 的地址是一個合約而不是 EOA，那就需要實作它的 body，這樣才可以在 ERC-721 的介面裡面回傳正確的 4 bytes hash。
```solidity=
  function onERC721Received(
      address, 
      address, 
      uint256, 
      bytes calldata
  )external pure returns(bytes4) {
      return bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"));
  } 
```

頭痛的問題是各種補充實作的 selector，未來有機會再跟大家分享。
其中一個我提出來並解決的問題後來被加到官方文件的 NFT 測試範例裡面了！
1. [Error From Testing ERC-721 Contract Mint Function #964](https://github.com/foundry-rs/foundry/discussions/964)
2. [Catching custom error](https://ethereum.stackexchange.com/questions/125238/catching-custom-error)

### 3. bash: make: command not found

到此[連結](https://chocolatey.org/install#individual)下載 choco，完成之後輸入以下指令：
```
$ choco install make
```

### 4. cargo install Problem || (vm || forge-std/Test.sol is not working)

1. 更新 Foundry，去官方文件用最新的方法再下載一次
3. 查看 [cheatcode && forge-std Reference](https://book.getfoundry.sh/reference/index.html)
4. 查看 vm 和 test 原始碼
5. 查看 forge-std 原始碼

### 5. msg.sender

```
/** Unit Testing For Mock Contracts*/

    function testPrank1() public {
        assertEq(DEPLOYER_ADDRESS, farm.owner()); // now owner() is DEPLOYER
        farm.transferOwnership(msg.sender); 
        // the transaction originator of transferOwnership is the "contract" ---> DEPLOYER == address(this)  
        assertEq(msg.sender, farm.owner()); // transfer successfully
        // now owner is the "msg.sender" ---> <sender_in_foundry.toml>

        vm.prank(DEPLOYER_ADDRESS);  // change the "msg.sender" ---> DEPLOYER == address(this)
        vm.expectRevert(farm.transferOwnership(msg.sender));
        /* ------ expectRevert 部分等價於以下註解程式碼 ------
        try farm.transferOwnership(msg.sender){
            assertTrue(false, "prank failed!");
            // transferOwnership should revert, 
            // when "msg.sender" is now being the DEPLOYER, but the owner is <sender_in_foundry.toml>

            // if transferOwnership not revert,
            // it means the function called successfully,
            // ---> the "msg.sender" is still the <sender_in_foundry.toml>
            // ---> we prank failed.
        } catch {
            assertTrue(true);
            // revert successfully
        }
        */

        // The most important thing is that the prank is only useful for "external" call
        // the only "external" call above is "farm.transferOwnership()"
    }

```

### 6. old cheat code


前一個版本的 Foundry 是利用宣告 `CheatCodes` 的介面，後在測試合約裡面宣告 `cheats`。最後只要在我們想要測試的合約裡面加上 `cheats.prank(address(0));` 就可以把自己的角度轉成 `address(0)`。
```solidity=
interface CheatCodes {
  function prank(address) external;
}

contract ContractTest is DSTest {
  CheatCodes cheats = CheatCodes(HEVM_ADDRESS);
    
  // skip the code...
    
  function testFailNotWLMint() public {
    cheats.prank(address(0));
    carman.preSaleMint(10);
  }  
}
```

---

## Others

### 1. Use Hardhat to Deploy

如果需要[環境變數](https://book.getfoundry.sh/forge/deploying.html?highlight=env#deploying)可以增設 `.env`：
```javascript
PRIVATE_KEY=<your_private_key>
RPC_URL=<your_API_key>
```

下載 Hardhat 作為開發工具，選擇 `Create an empty hardhat.config.js`：
```javascript=
$ yarn add hardhat
$ yarn hardhat
>
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

Welcome to Hardhat v2.9.6

? What do you want to do? ... 
  Create a basic sample project
  Create an advanced sample project
  Create an advanced sample project that uses TypeScript
> Create an empty hardhat.config.js
  Quit
```

使用以下指令在本地執行模擬的 JSON RPC 環境，得到一些可供測試的 Accounts：
```javascript=
$ yarn hardhat node
>
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.

Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

Account #1: 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (10000 ETH)
Private Key: 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d

...

Account #19: 0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199 (10000 ETH)
Private Key: 0xdf57089febbacf7ba0bc227dafbffa9fc08a93fdc68e1e42411a14efcf23656e

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.
```

開啟一個新的 Terminal 來測試 Deploy：
```javascript
$ forge create <ContractName> --private-key <your_private_key> --rpc-url <RPC_URL>
$ forge create Contract --private-key <dyour_private_key> --rpc-url http://127.0.0.1:8545/
```

也可以打開 Ganache 使用 http://127.0.0.1:7545/ 來配合。
