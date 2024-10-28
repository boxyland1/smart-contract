# smart-contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MonToken is ERC20, Ownable {
    uint256 public constant INITIAL_SUPPLY = 1000000 * 10**18; // 1 million tokens
    uint256 public constant MAX_SUPPLY = 2000000 * 10**18;    // 2 million tokens max
    uint256 public mintPrice = 0.001 ether;                   // Prix pour minter 1 token

    // Événement pour le changement de prix
    event MintPriceChanged(uint256 newPrice);
    
    constructor() ERC20("MonToken", "MTK") Ownable(msg.sender) {
        _mint(msg.sender, INITIAL_SUPPLY);
    }

    // Fonction pour minter de nouveaux tokens
    function mint(uint256 amount) public payable {
        require(totalSupply() + amount <= MAX_SUPPLY, "Depasse l'approvisionnement maximum");
        require(msg.value >= amount * mintPrice, "Montant ETH insuffisant");
        
        _mint(msg.sender, amount);
    }

    // Fonction pour changer le prix du minting (seulement owner)
    function setMintPrice(uint256 newPrice) public onlyOwner {
        mintPrice = newPrice;
        emit MintPriceChanged(newPrice);
    }

    // Fonction pour retirer les ETH du contrat (seulement owner)
    function withdraw() public onlyOwner {
        uint256 balance = address(this).balance;
        payable(owner()).transfer(balance);
    }

    // Fonction pour bruler des tokens
    function burn(uint256 amount) public {
        _burn(msg.sender, amount);
    }

    // Fonction pour voir le prix actuel du minting
    function getMintPrice() public view returns (uint256) {
        return mintPrice;
    }

    // Fonction pour voir combien de tokens peuvent encore être créés
    function remainingSupply() public view returns (uint256) {
        return MAX_SUPPLY - totalSupply();
    }
}
