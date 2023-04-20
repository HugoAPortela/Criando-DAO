# Construindo um DAO

![](https://i.imgur.com/6uXR2G9.png)

O que é um DAO?
DAO significa Organização Autônoma Descentralizada . _ Você pode pensar nos DAOs como análogos às empresas do mundo real. Essencialmente, os DAOs permitem que os membros criem e votem nas decisões de governança.

Nas empresas tradicionais, quando uma decisão precisa ser tomada, o conselho de administração ou os executivos da empresa são os responsáveis ​​por tomar essa decisão. Em um DAO, no entanto, esse processo é democratizado, e qualquer membro pode criar uma proposta e todos os outros membros podem votar nela. Cada proposta criada tem um prazo para votação, e após o prazo a decisão é tomada a favor do resultado da votação (SIM ou NÃO).

A associação em DAOs é normalmente restrita pela propriedade de tokens ERC20 ou pela propriedade de NFTs. Exemplos de DAOs em que o poder de associação e voto é proporcional a quantos tokens você possui incluem Uniswap e ENS . Exemplos de DAOs em que são baseados em NFTs incluem Meebits DAO .

Construindo nosso DAO
Você deseja lançar um DAO para os titulares de seus CryptoDevsNFTs. Do ETH obtido por meio da ICO, você construiu um DAO Treasury. O DAO agora tem muito ETH, mas atualmente não faz nada com ele.

Você deseja permitir que seus detentores de NFT criem e votem em propostas para usar esse ETH para comprar outros NFTs de um mercado de NFT e especular sobre o preço. Talvez no futuro, quando você vender o NFT de volta, você divida os lucros entre todos os membros do DAO.

Requisitos
Qualquer pessoa com um CryptoDevsNFT pode criar uma proposta para comprar um NFT diferente de um mercado NFT
Todos com um CryptoDevsNFT podem votar a favor ou contra as propostas ativas
Cada NFT conta como um voto para cada proposta
O eleitor não pode votar várias vezes na mesma proposta com o mesmo NFT
Se a maioria dos votantes votar pela proposta dentro do prazo, a compra da NFT é automaticamente executada
o que faremos
Para poder comprar NFTs automaticamente quando uma proposta é aprovada, você precisa de um mercado NFT on-chain no qual possa chamar uma purchase()função. Existem muitos mercados NFT por aí, mas para evitar complicar as coisas, criaremos um mercado NFT falso simplificado para este tutorial, pois o foco está no DAO.
Também faremos o contrato inteligente DAO real usando Hardhat.
Faremos o site usando Next.js para permitir que os usuários criem e votem em propostas
Pré-requisitos
Você concluiu o tutorial NFT-Collection
Você deve ter algum ETH para dar ao DAO Treasury
CONSTRUIR
Desenvolvimento de contrato inteligente
Começaremos criando primeiro os contratos inteligentes. Faremos dois contratos inteligentes:

FakeNFTMarketplace.sol
CryptoDevsDAO.sol
Para fazer isso, usaremos a estrutura de desenvolvimento Hardhat que usamos nos últimos tutoriais.

Crie uma pasta para este projeto chamada DAO-Tutoriale abra uma janela do Terminal nessa pasta.

Configure um novo projeto de capacete executando os seguintes comandos em seu terminal:

mkdir hardhat-tutorial
cd hardhat-tutorial
npm init --yes
npm install --save-dev hardhat
Agora que você instalou o Hardhat, podemos configurar um projeto. Execute o seguinte comando em seu terminal.

No mesmo diretório onde você instalou o Hardhat execute:

npx hardhat
SelecioneCreate a Javascript project

Pressione enter para o já especificadoHardhat Project root

Pressione enter para a pergunta se você deseja adicionar um.gitignore

Pressione enter paraDo you want to install this sample project's dependencies with npm (@nomicfoundation/hardhat-toolbox)?

Agora você tem um projeto de capacete pronto para começar!

Se você estiver no Windows, execute esta etapa extra e instale essas bibliotecas também :)

npm install --save-dev @nomicfoundation/hardhat-toolbox
e pressione Enterpara todas as perguntas (Escolha a Create a basic sample projectopção ).

Agora, vamos instalar o @openzeppelin/contractspacote do NPM, pois usaremos o Ownable Contract do OpenZeppelin para o contrato DAO.

npm install @openzeppelin/contracts
Primeiro, vamos fazer um simples Fake NFT Marketplace. Crie um arquivo nomeado FakeNFTMarketplace.solno contractsdiretório dentro de hardhat-tutoriale adicione o código a seguir.

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract FakeNFTMarketplace {
    /// @dev Maintain a mapping of Fake TokenID to Owner addresses
    mapping(uint256 => address) public tokens;
    /// @dev Set the purchase price for each Fake NFT
    uint256 nftPrice = 0.1 ether;

    /// @dev purchase() accepts ETH and marks the owner of the given tokenId as the caller address
    /// @param _tokenId - the fake NFT token Id to purchase
    function purchase(uint256 _tokenId) external payable {
        require(msg.value == nftPrice, "This NFT costs 0.1 ether");
        tokens[_tokenId] = msg.sender;
    }

    /// @dev getPrice() returns the price of one NFT
    function getPrice() external view returns (uint256) {
        return nftPrice;
    }

    /// @dev available() checks whether the given tokenId has already been sold or not
    /// @param _tokenId - the tokenId to check for
    function available(uint256 _tokenId) external view returns (bool) {
        // address(0) = 0x0000000000000000000000000000000000000000
        // This is the default value for addresses in Solidity
        if (tokens[_tokenId] == address(0)) {
            return true;
        }
        return false;
    }
}
O FakeNFTMarketplaceexpõe algumas funções básicas que usaremos do contrato DAO para comprar NFTs se uma proposta for aprovada. Um mercado NFT real seria mais complicado - pois nem todos os NFTs têm o mesmo preço.

Vamos garantir que tudo seja compilado antes de começarmos a escrever o Contrato DAO. Execute o seguinte comando dentro da hardhat-tutorialpasta do seu Terminal.

npx hardhat compile
e verifique se não há erros de compilação.

Agora, vamos começar a escrever o CryptoDevsDAOcontrato. Como este é um contrato totalmente personalizado e relativamente mais complicado do que o que fizemos até agora, explicaremos isso passo a passo.

Primeiro, vamos escrever o código padronizado para o contrato. Crie um novo arquivo nomeado CryptoDevsDAO.solno contractsdiretório in hardhat-tutoriale adicione o seguinte código a ele.

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/Ownable.sol";

// We will add the Interfaces here

contract CryptoDevsDAO is Ownable {
    // We will write contract code here
}
Agora, precisaremos chamar funções no FakeNFTMarketplacecontrato e seu CryptoDevs NFTcontrato implantado anteriormente. Lembre-se do Advanced Solidity Topicstutorial de que precisamos fornecer uma interface para esses contratos, para que esse contrato saiba quais funções estão disponíveis para chamar e o que elas recebem como parâmetros e o que retornam.

Adicione as duas interfaces a seguir ao seu código adicionando o seguinte código

/**
 * Interface for the FakeNFTMarketplace
 */
interface IFakeNFTMarketplace {
    /// @dev getPrice() returns the price of an NFT from the FakeNFTMarketplace
    /// @return Returns the price in Wei for an NFT
    function getPrice() external view returns (uint256);

    /// @dev available() returns whether or not the given _tokenId has already been purchased
    /// @return Returns a boolean value - true if available, false if not
    function available(uint256 _tokenId) external view returns (bool);

    /// @dev purchase() purchases an NFT from the FakeNFTMarketplace
    /// @param _tokenId - the fake NFT tokenID to purchase
    function purchase(uint256 _tokenId) external payable;
}

/**
 * Minimal interface for CryptoDevsNFT containing only two functions
 * that we are interested in
 */
interface ICryptoDevsNFT {
    /// @dev balanceOf returns the number of NFTs owned by the given address
    /// @param owner - address to fetch number of NFTs for
    /// @return Returns the number of NFTs owned
    function balanceOf(address owner) external view returns (uint256);

    /// @dev tokenOfOwnerByIndex returns a tokenID at given index for owner
    /// @param owner - address to fetch the NFT TokenID for
    /// @param index - index of NFT in owned tokens array to fetch
    /// @return Returns the TokenID of the NFT
    function tokenOfOwnerByIndex(address owner, uint256 index)
        external
        view
        returns (uint256);
}
Agora, vamos pensar em qual funcionalidade precisamos no contrato DAO.

Armazenar propostas criadas no estado do contrato
Permitir que os detentores do CryptoDevs NFT criem novas propostas
Permitir que os detentores do CryptoDevs NFT votem em propostas, desde que ainda não tenham votado e que a proposta ainda não tenha passado do prazo
Permitir que os detentores do CryptoDevs NFT executem uma proposta após o término do prazo, acionando uma compra de NFT caso ela seja aprovada
Vamos começar criando um structque representa um arquivo Proposal. No seu contrato, adicione o seguinte código:

// Create a struct named Proposal containing all relevant information
struct Proposal {
    // nftTokenId - the tokenID of the NFT to purchase from FakeNFTMarketplace if the proposal passes
    uint256 nftTokenId;
    // deadline - the UNIX timestamp until which this proposal is active. Proposal can be executed after the deadline has been exceeded.
    uint256 deadline;
    // yayVotes - number of yay votes for this proposal
    uint256 yayVotes;
    // nayVotes - number of nay votes for this proposal
    uint256 nayVotes;
    // executed - whether or not this proposal has been executed yet. Cannot be executed before the deadline has been exceeded.
    bool executed;
    // voters - a mapping of CryptoDevsNFT tokenIDs to booleans indicating whether that NFT has already been used to cast a vote or not
    mapping(uint256 => bool) voters;
}
Vamos também criar um mapeamento de IDs de proposta para propostas para armazenar todas as propostas criadas e um contador para contar o número de propostas existentes.

// Create a mapping of ID to Proposal
mapping(uint256 => Proposal) public proposals;
// Number of proposals that have been created
uint256 public numProposals;
Agora, como chamaremos funções no contrato FakeNFTMarketplacee CryptoDevsNFT, vamos inicializar variáveis ​​para esses contratos.

IFakeNFTMarketplace nftMarketplace;
ICryptoDevsNFT cryptoDevsNFT;
Crie uma constructorfunção que inicializará essas variáveis ​​de contrato e também aceitará um depósito ETH do implantador para preencher o tesouro DAO ETH. (Em segundo plano, como importamos o Ownablecontrato, isso também definirá o implantador do contrato como o proprietário deste contrato)

// Create a payable constructor which initializes the contract
// instances for FakeNFTMarketplace and CryptoDevsNFT
// The payable allows this constructor to accept an ETH deposit when it is being deployed
constructor(address _nftMarketplace, address _cryptoDevsNFT) payable {
    nftMarketplace = IFakeNFTMarketplace(_nftMarketplace);
    cryptoDevsNFT = ICryptoDevsNFT(_cryptoDevsNFT);
}
Agora, como queremos que praticamente todas as nossas outras funções sejam chamadas apenas por quem possui NFTs do CryptoDevs NFTcontrato, vamos criar um modifierpara evitar duplicação de código.

// Create a modifier which only allows a function to be
// called by someone who owns at least 1 CryptoDevsNFT
modifier nftHolderOnly() {
    require(cryptoDevsNFT.balanceOf(msg.sender) > 0, "NOT_A_DAO_MEMBER");
    _;
}
Agora temos o suficiente para escrever nossa createProposalfunção, o que permitirá aos membros criar novas propostas.

/// @dev createProposal allows a CryptoDevsNFT holder to create a new proposal in the DAO
/// @param _nftTokenId - the tokenID of the NFT to be purchased from FakeNFTMarketplace if this proposal passes
/// @return Returns the proposal index for the newly created proposal
function createProposal(uint256 _nftTokenId)
    external
    nftHolderOnly
    returns (uint256)
{
    require(nftMarketplace.available(_nftTokenId), "NFT_NOT_FOR_SALE");
    Proposal storage proposal = proposals[numProposals];
    proposal.nftTokenId = _nftTokenId;
    // Set the proposal's voting deadline to be (current time + 5 minutes)
    proposal.deadline = block.timestamp + 5 minutes;

    numProposals++;

    return numProposals - 1;
}
Agora, para votar uma proposta, queremos adicionar uma restrição adicional de que a proposta em votação não pode ter seu prazo excedido. Para fazer isso, criaremos um segundo modificador.

// Create a modifier which only allows a function to be
// called if the given proposal's deadline has not been exceeded yet
modifier activeProposalOnly(uint256 proposalIndex) {
    require(
        proposals[proposalIndex].deadline > block.timestamp,
        "DEADLINE_EXCEEDED"
    );
    _;
}
Observe como esse modificador recebe um parâmetro!

Além disso, como um voto pode ser apenas um dos dois valores (YAY ou NAY) - podemos criar uma enumrepresentação de opções possíveis.

// Create an enum named Vote containing possible options for a vote
enum Vote {
    YAY, // YAY = 0
    NAY // NAY = 1
}
Vamos escrever a voteOnProposalfunção

/// @dev voteOnProposal allows a CryptoDevsNFT holder to cast their vote on an active proposal
/// @param proposalIndex - the index of the proposal to vote on in the proposals array
/// @param vote - the type of vote they want to cast
function voteOnProposal(uint256 proposalIndex, Vote vote)
    external
    nftHolderOnly
    activeProposalOnly(proposalIndex)
{
    Proposal storage proposal = proposals[proposalIndex];

    uint256 voterNFTBalance = cryptoDevsNFT.balanceOf(msg.sender);
    uint256 numVotes = 0;

    // Calculate how many NFTs are owned by the voter
    // that haven't already been used for voting on this proposal
    for (uint256 i = 0; i < voterNFTBalance; i++) {
        uint256 tokenId = cryptoDevsNFT.tokenOfOwnerByIndex(msg.sender, i);
        if (proposal.voters[tokenId] == false) {
            numVotes++;
            proposal.voters[tokenId] = true;
        }
    }
    require(numVotes > 0, "ALREADY_VOTED");

    if (vote == Vote.YAY) {
        proposal.yayVotes += numVotes;
    } else {
        proposal.nayVotes += numVotes;
    }
}
Estamos quase terminando! Para executar uma proposta cujo prazo foi excedido, criaremos nosso modificador final.

// Create a modifier which only allows a function to be
// called if the given proposals' deadline HAS been exceeded
// and if the proposal has not yet been executed
modifier inactiveProposalOnly(uint256 proposalIndex) {
    require(
        proposals[proposalIndex].deadline <= block.timestamp,
        "DEADLINE_NOT_EXCEEDED"
    );
    require(
        proposals[proposalIndex].executed == false,
        "PROPOSAL_ALREADY_EXECUTED"
    );
    _;
}
Observe que esse modificador também recebe um parâmetro!

Vamos escrever o código paraexecuteProposal

/// @dev executeProposal allows any CryptoDevsNFT holder to execute a proposal after it's deadline has been exceeded
/// @param proposalIndex - the index of the proposal to execute in the proposals array
function executeProposal(uint256 proposalIndex)
    external
    nftHolderOnly
    inactiveProposalOnly(proposalIndex)
{
    Proposal storage proposal = proposals[proposalIndex];

    // If the proposal has more YAY votes than NAY votes
    // purchase the NFT from the FakeNFTMarketplace
    if (proposal.yayVotes > proposal.nayVotes) {
        uint256 nftPrice = nftMarketplace.getPrice();
        require(address(this).balance >= nftPrice, "NOT_ENOUGH_FUNDS");
        nftMarketplace.purchase{value: nftPrice}(proposal.nftTokenId);
    }
    proposal.executed = true;
}
Neste ponto, implementamos toda a funcionalidade principal. No entanto, existem alguns recursos adicionais que podemos e devemos implementar.

Permita que o proprietário do contrato retire o ETH do DAO, se necessário
Permitir que o contrato aceite mais depósitos de ETH
O Ownablecontrato do qual herdamos contém um modificador onlyOwnerque restringe uma função para poder ser chamada apenas pelo proprietário do contrato. Vamos implementar withdrawEtherusando esse modificador.

/// @dev withdrawEther allows the contract owner (deployer) to withdraw the ETH from the contract
function withdrawEther() external onlyOwner {
    payable(owner()).transfer(address(this).balance);
}
Isso transferirá todo o saldo de ETH do contrato para o endereço do proprietário

Por fim, para permitir a adição de mais depósitos de ETH à tesouraria DAO, precisamos adicionar algumas funções especiais. Normalmente, os endereços de contrato não podem aceitar ETH enviado a eles, a menos que seja por meio de uma payablefunção. Mas não queremos que os usuários chamem funções apenas para depositar dinheiro, eles devem poder transferir ETH diretamente de sua carteira. Para isso, vamos adicionar essas duas funções:

// The following two functions allow the contract to accept ETH deposits
// directly from a wallet without calling a function
receive() external payable {}

fallback() external payable {}
Implantação de contrato inteligente
Agora que escrevemos nossos dois contratos, vamos implantá-los no Rinkeby Testnet . Certifique-se de ter algum ETH no Rinkeby Testnet.

Instale o dotenvpacote do NPM para poder usar variáveis ​​de ambiente especificadas em .envarquivos no formato hardhat.config.js. Execute o seguinte comando em seu Terminal no hardhat-tutorialdiretório.

npm install dotenv
Agora crie um .envarquivo no hardhat-tutorialdiretório e defina as duas variáveis ​​de ambiente a seguir. Siga as instruções para obter seus valores. Certifique-se de que a chave privada Rinkeby que você usa tem ETH no Rinkeby Testnet.

// Go to https://www.alchemyapi.io, sign up, create
// a new App in its dashboard and select the network as Rinkeby, and replace "add-the-alchemy-key-url-here" with its key url
ALCHEMY_API_KEY_URL="add-the-alchemy-key-url-here"

// Replace this private key with your RINKEBY account private key
// To export your private key from Metamask, open Metamask and
// go to Account Details > Export Private Key
// Be aware of NEVER putting real Ether into testing accounts
RINKEBY_PRIVATE_KEY="add-the-rinkeby-private-key-here"
Agora, vamos escrever um script de implantação para implantar automaticamente nossos dois contratos para nós. Crie um novo arquivo ou substitua o arquivo padrão existente, nomeado deploy.jssob hardhat-tutorial/scripts, e adicione o seguinte código:

const { ethers } = require("hardhat");
const { CRYPTODEVS_NFT_CONTRACT_ADDRESS } = require("../constants");

async function main() {
  // Deploy the FakeNFTMarketplace contract first
  const FakeNFTMarketplace = await ethers.getContractFactory(
    "FakeNFTMarketplace"
  );
  const fakeNftMarketplace = await FakeNFTMarketplace.deploy();
  await fakeNftMarketplace.deployed();

  console.log("FakeNFTMarketplace deployed to: ", fakeNftMarketplace.address);

  // Now deploy the CryptoDevsDAO contract
  const CryptoDevsDAO = await ethers.getContractFactory("CryptoDevsDAO");
  const cryptoDevsDAO = await CryptoDevsDAO.deploy(
    fakeNftMarketplace.address,
    CRYPTODEVS_NFT_CONTRACT_ADDRESS,
    {
      // This assumes your account has at least 1 ETH in it's account
      // Change this value as you want
      value: ethers.utils.parseEther("1"),
    }
  );
  await cryptoDevsDAO.deployed();

  console.log("CryptoDevsDAO deployed to: ", cryptoDevsDAO.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
Como você deve ter notado, deploy.jsimporta uma variável chamada CRYPTODEVS_NFT_CONTRACT_ADDRESSde um arquivo chamado constants. Vamos fazer isso. Crie um novo arquivo nomeado constants.jsno hardhat-tutorialdiretório.

// Replace the value with your NFT contract address
const CRYPTODEVS_NFT_CONTRACT_ADDRESS =
  "YOUR_CRYPTODEVS_NFT_CONTRACT_ADDRESS_HERE";

module.exports = { CRYPTODEVS_NFT_CONTRACT_ADDRESS };
Agora, vamos adicionar a rede Rinkeby à sua configuração de capacete para que possamos implantar em Rinkeby. Abra seu hardhat.config.jsarquivo e substitua-o pelo seguinte:

require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config({ path: ".env" });

const ALCHEMY_API_KEY_URL = process.env.ALCHEMY_API_KEY_URL;

const RINKEBY_PRIVATE_KEY = process.env.RINKEBY_PRIVATE_KEY;

module.exports = {
  solidity: "0.8.9",
  networks: {
    rinkeby: {
      url: ALCHEMY_API_KEY_URL,
      accounts: [RINKEBY_PRIVATE_KEY],
    },
  },
};
Vamos garantir que tudo seja compilado antes de prosseguir. Execute o seguinte comando do seu Terminal dentro da hardhat-tutorialpasta.

npx hardhat compile
e verifique se não há erros de compilação. Se você enfrentar erros de compilação, tente comparar seu código com a versão final presente aqui

Vamos implantar! Execute o seguinte comando no seu Terminal a partir do hardhat-tutorialdiretório

npx hardhat run scripts/deploy.js --network rinkeby
Salve os endereços FakeNFTMarketplacee CryptoDevsDAOcontratos que são impressos em seu Terminal. Você precisará deles mais tarde.

Desenvolvimento Frontend
Ufa! Quanta codificação!

Desenvolvemos e implantamos com sucesso nossos contratos no Rinkeby Testnet. Agora, é hora de construir a interface do Frontend para que os usuários possam criar e votar as propostas do site.

Para desenvolver o site, usaremos o Next.js como temos feito até agora, que é um meta-framework construído sobre o React .

Vamos começar criando um novo nextaplicativo. Sua estrutura de pastas deve ficar assim depois de configurar o nextaplicativo:

- DAO-Tutorial
    - hardhat-tutorial
    - my-app
Para criar my-app, execute o seguinte comando em seu Terminal dentro do DAO-Tutorialdiretório

npx create-next-app@latest
e pressione Enterpara todas as perguntas. Isso deve criar a my-apppasta e configurar um projeto Next.js básico.

Vamos ver se tudo funciona. Execute o seguinte no seu Terminal

cd my-app
npm run dev
Seu site deve estar funcionando em http://localhost:3000. No entanto, este é um projeto inicial básico do Next.js e precisamos adicionar código para que ele faça o que queremos.

Vamos instalar a biblioteca web3modale ethers. O Web3Modal nos permitirá suportar a conexão com carteiras no navegador, e os Ethers serão usados ​​para interagir com o blockchain. Execute isso no seu Terminal a partir do my-appdiretório.

npm install web3modal ethers
Baixe e salve o seguinte arquivo como 0.svgem my-app/public/cryptodevs. Vamos exibir esta imagem na página da web. NOTA: Você precisa criar a cryptodevspasta dentro de public. Baixar imagem

Adicione os seguintes estilos CSS emmy-app/styles/Home.modules.css

.main {
  min-height: 90vh;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  font-family: "Courier New", Courier, monospace;
}

.footer {
  display: flex;
  padding: 2rem 0;
  border-top: 1px solid #eaeaea;
  justify-content: center;
  align-items: center;
}

.image {
  width: 70%;
  height: 50%;
  margin-left: 20%;
}

.title {
  font-size: 2rem;
  margin: 2rem 0;
}

.description {
  line-height: 1;
  margin: 2rem 0;
  font-size: 1.2rem;
}

.button {
  border-radius: 4px;
  background-color: blue;
  border: none;
  color: #ffffff;
  font-size: 15px;
  padding: 10px;
  width: 200px;
  cursor: pointer;
  margin-right: 2%;
}

.button2 {
  border-radius: 4px;
  background-color: indigo;
  border: none;
  color: #ffffff;
  font-size: 15px;
  padding: 10px;
  cursor: pointer;
  margin-right: 2%;
  margin-top: 1rem;
}

.proposalCard {
  width: fit-content;
  margin-top: 0.25rem;
  border: black 2px solid;
  flex: 1;
  flex-direction: column;
}

.container {
  margin-top: 2rem;
}

.flex {
  flex: 1;
  justify-content: space-between;
}

@media (max-width: 1000px) {
  .main {
    width: 100%;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
}
O site também precisa ler/gravar dados de dois contratos inteligentes - CryptoDevsDAOe CryptoDevsNFT. Vamos armazenar seus endereços de contrato e ABIs em um arquivo de constantes. Crie um constants.jsarquivo no my-appdiretório.

export const CRYPTODEVS_DAO_CONTRACT_ADDRESS = "";
export const CRYPTODEVS_NFT_CONTRACT_ADDRESS = "";

export const CRYPTODEVS_DAO_ABI = [];
export const CRYPTODEVS_NFT_ABI = [];
Substitua o endereço do contrato e os valores de ABI pelos endereços de contrato e ABIs relevantes.

Agora, para o código legal do site. Abra my-app/pages/index.jse escreva o seguinte código. A explicação do código pode ser encontrada nos comentários.

import { Contract, providers } from "ethers";
import { formatEther } from "ethers/lib/utils";
import Head from "next/head";
import { useEffect, useRef, useState } from "react";
import Web3Modal from "web3modal";
import {
  CRYPTODEVS_DAO_ABI,
  CRYPTODEVS_DAO_CONTRACT_ADDRESS,
  CRYPTODEVS_NFT_ABI,
  CRYPTODEVS_NFT_CONTRACT_ADDRESS,
} from "../constants";
import styles from "../styles/Home.module.css";

export default function Home() {
  // ETH Balance of the DAO contract
  const [treasuryBalance, setTreasuryBalance] = useState("0");
  // Number of proposals created in the DAO
  const [numProposals, setNumProposals] = useState("0");
  // Array of all proposals created in the DAO
  const [proposals, setProposals] = useState([]);
  // User's balance of CryptoDevs NFTs
  const [nftBalance, setNftBalance] = useState(0);
  // Fake NFT Token ID to purchase. Used when creating a proposal.
  const [fakeNftTokenId, setFakeNftTokenId] = useState("");
  // One of "Create Proposal" or "View Proposals"
  const [selectedTab, setSelectedTab] = useState("");
  // True if waiting for a transaction to be mined, false otherwise.
  const [loading, setLoading] = useState(false);
  // True if user has connected their wallet, false otherwise
  const [walletConnected, setWalletConnected] = useState(false);
  const web3ModalRef = useRef();

  // Helper function to connect wallet
  const connectWallet = async () => {
    try {
      await getProviderOrSigner();
      setWalletConnected(true);
    } catch (error) {
      console.error(error);
    }
  };

  // Reads the ETH balance of the DAO contract and sets the `treasuryBalance` state variable
  const getDAOTreasuryBalance = async () => {
    try {
      const provider = await getProviderOrSigner();
      const balance = await provider.getBalance(
        CRYPTODEVS_DAO_CONTRACT_ADDRESS
      );
      setTreasuryBalance(balance.toString());
    } catch (error) {
      console.error(error);
    }
  };

  // Reads the number of proposals in the DAO contract and sets the `numProposals` state variable
  const getNumProposalsInDAO = async () => {
    try {
      const provider = await getProviderOrSigner();
      const contract = getDaoContractInstance(provider);
      const daoNumProposals = await contract.numProposals();
      setNumProposals(daoNumProposals.toString());
    } catch (error) {
      console.error(error);
    }
  };

  // Reads the balance of the user's CryptoDevs NFTs and sets the `nftBalance` state variable
  const getUserNFTBalance = async () => {
    try {
      const signer = await getProviderOrSigner(true);
      const nftContract = getCryptodevsNFTContractInstance(signer);
      const balance = await nftContract.balanceOf(signer.getAddress());
      setNftBalance(parseInt(balance.toString()));
    } catch (error) {
      console.error(error);
    }
  };

  // Calls the `createProposal` function in the contract, using the tokenId from `fakeNftTokenId`
  const createProposal = async () => {
    try {
      const signer = await getProviderOrSigner(true);
      const daoContract = getDaoContractInstance(signer);
      const txn = await daoContract.createProposal(fakeNftTokenId);
      setLoading(true);
      await txn.wait();
      await getNumProposalsInDAO();
      setLoading(false);
    } catch (error) {
      console.error(error);
      window.alert(error.data.message);
    }
  };

  // Helper function to fetch and parse one proposal from the DAO contract
  // Given the Proposal ID
  // and converts the returned data into a Javascript object with values we can use
  const fetchProposalById = async (id) => {
    try {
      const provider = await getProviderOrSigner();
      const daoContract = getDaoContractInstance(provider);
      const proposal = await daoContract.proposals(id);
      const parsedProposal = {
        proposalId: id,
        nftTokenId: proposal.nftTokenId.toString(),
        deadline: new Date(parseInt(proposal.deadline.toString()) * 1000),
        yayVotes: proposal.yayVotes.toString(),
        nayVotes: proposal.nayVotes.toString(),
        executed: proposal.executed,
      };
      return parsedProposal;
    } catch (error) {
      console.error(error);
    }
  };

  // Runs a loop `numProposals` times to fetch all proposals in the DAO
  // and sets the `proposals` state variable
  const fetchAllProposals = async () => {
    try {
      const proposals = [];
      for (let i = 0; i < numProposals; i++) {
        const proposal = await fetchProposalById(i);
        proposals.push(proposal);
      }
      setProposals(proposals);
      return proposals;
    } catch (error) {
      console.error(error);
    }
  };

  // Calls the `voteOnProposal` function in the contract, using the passed
  // proposal ID and Vote
  const voteOnProposal = async (proposalId, _vote) => {
    try {
      const signer = await getProviderOrSigner(true);
      const daoContract = getDaoContractInstance(signer);

      let vote = _vote === "YAY" ? 0 : 1;
      const txn = await daoContract.voteOnProposal(proposalId, vote);
      setLoading(true);
      await txn.wait();
      setLoading(false);
      await fetchAllProposals();
    } catch (error) {
      console.error(error);
      window.alert(error.data.message);
    }
  };

  // Calls the `executeProposal` function in the contract, using
  // the passed proposal ID
  const executeProposal = async (proposalId) => {
    try {
      const signer = await getProviderOrSigner(true);
      const daoContract = getDaoContractInstance(signer);
      const txn = await daoContract.executeProposal(proposalId);
      setLoading(true);
      await txn.wait();
      setLoading(false);
      await fetchAllProposals();
    } catch (error) {
      console.error(error);
      window.alert(error.data.message);
    }
  };

  // Helper function to fetch a Provider/Signer instance from Metamask
  const getProviderOrSigner = async (needSigner = false) => {
    const provider = await web3ModalRef.current.connect();
    const web3Provider = new providers.Web3Provider(provider);

    const { chainId } = await web3Provider.getNetwork();
    if (chainId !== 4) {
      window.alert("Please switch to the Rinkeby network!");
      throw new Error("Please switch to the Rinkeby network");
    }

    if (needSigner) {
      const signer = web3Provider.getSigner();
      return signer;
    }
    return web3Provider;
  };

  // Helper function to return a DAO Contract instance
  // given a Provider/Signer
  const getDaoContractInstance = (providerOrSigner) => {
    return new Contract(
      CRYPTODEVS_DAO_CONTRACT_ADDRESS,
      CRYPTODEVS_DAO_ABI,
      providerOrSigner
    );
  };

  // Helper function to return a CryptoDevs NFT Contract instance
  // given a Provider/Signer
  const getCryptodevsNFTContractInstance = (providerOrSigner) => {
    return new Contract(
      CRYPTODEVS_NFT_CONTRACT_ADDRESS,
      CRYPTODEVS_NFT_ABI,
      providerOrSigner
    );
  };

  // piece of code that runs everytime the value of `walletConnected` changes
  // so when a wallet connects or disconnects
  // Prompts user to connect wallet if not connected
  // and then calls helper functions to fetch the
  // DAO Treasury Balance, User NFT Balance, and Number of Proposals in the DAO
  useEffect(() => {
    if (!walletConnected) {
      web3ModalRef.current = new Web3Modal({
        network: "rinkeby",
        providerOptions: {},
        disableInjectedProvider: false,
      });

      connectWallet().then(() => {
        getDAOTreasuryBalance();
        getUserNFTBalance();
        getNumProposalsInDAO();
      });
    }
  }, [walletConnected]);

  // Piece of code that runs everytime the value of `selectedTab` changes
  // Used to re-fetch all proposals in the DAO when user switches
  // to the 'View Proposals' tab
  useEffect(() => {
    if (selectedTab === "View Proposals") {
      fetchAllProposals();
    }
  }, [selectedTab]);

  // Render the contents of the appropriate tab based on `selectedTab`
  function renderTabs() {
    if (selectedTab === "Create Proposal") {
      return renderCreateProposalTab();
    } else if (selectedTab === "View Proposals") {
      return renderViewProposalsTab();
    }
    return null;
  }

  // Renders the 'Create Proposal' tab content
  function renderCreateProposalTab() {
    if (loading) {
      return (
        <div className={styles.description}>
          Loading... Waiting for transaction...
        </div>
      );
    } else if (nftBalance === 0) {
      return (
        <div className={styles.description}>
          You do not own any CryptoDevs NFTs. <br />
          <b>You cannot create or vote on proposals</b>
        </div>
      );
    } else {
      return (
        <div className={styles.container}>
          <label>Fake NFT Token ID to Purchase: </label>
          <input
            placeholder="0"
            type="number"
            onChange={(e) => setFakeNftTokenId(e.target.value)}
          />
          <button className={styles.button2} onClick={createProposal}>
            Create
          </button>
        </div>
      );
    }
  }

  // Renders the 'View Proposals' tab content
  function renderViewProposalsTab() {
    if (loading) {
      return (
        <div className={styles.description}>
          Loading... Waiting for transaction...
        </div>
      );
    } else if (proposals.length === 0) {
      return (
        <div className={styles.description}>
          No proposals have been created
        </div>
      );
    } else {
      return (
        <div>
          {proposals.map((p, index) => (
            <div key={index} className={styles.proposalCard}>
              <p>Proposal ID: {p.proposalId}</p>
              <p>Fake NFT to Purchase: {p.nftTokenId}</p>
              <p>Deadline: {p.deadline.toLocaleString()}</p>
              <p>Yay Votes: {p.yayVotes}</p>
              <p>Nay Votes: {p.nayVotes}</p>
              <p>Executed?: {p.executed.toString()}</p>
              {p.deadline.getTime() > Date.now() && !p.executed ? (
                <div className={styles.flex}>
                  <button
                    className={styles.button2}
                    onClick={() => voteOnProposal(p.proposalId, "YAY")}
                  >
                    Vote YAY
                  </button>
                  <button
                    className={styles.button2}
                    onClick={() => voteOnProposal(p.proposalId, "NAY")}
                  >
                    Vote NAY
                  </button>
                </div>
              ) : p.deadline.getTime() < Date.now() && !p.executed ? (
                <div className={styles.flex}>
                  <button
                    className={styles.button2}
                    onClick={() => executeProposal(p.proposalId)}
                  >
                    Execute Proposal{" "}
                    {p.yayVotes > p.nayVotes ? "(YAY)" : "(NAY)"}
                  </button>
                </div>
              ) : (
                <div className={styles.description}>Proposal Executed</div>
              )}
            </div>
          ))}
        </div>
      );
    }
  }

  return (
    <div>
      <Head>
        <title>CryptoDevs DAO</title>
        <meta name="description" content="CryptoDevs DAO" />
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <div className={styles.main}>
        <div>
          <h1 className={styles.title}>Welcome to Crypto Devs!</h1>
          <div className={styles.description}>Welcome to the DAO!</div>
          <div className={styles.description}>
            Your CryptoDevs NFT Balance: {nftBalance}
            <br />
            Treasury Balance: {formatEther(treasuryBalance)} ETH
            <br />
            Total Number of Proposals: {numProposals}
          </div>
          <div className={styles.flex}>
            <button
              className={styles.button}
              onClick={() => setSelectedTab("Create Proposal")}
            >
              Create Proposal
            </button>
            <button
              className={styles.button}
              onClick={() => setSelectedTab("View Proposals")}
            >
              View Proposals
            </button>
          </div>
          {renderTabs()}
        </div>
        <div>
          <img className={styles.image} src="/cryptodevs/0.svg" />
        </div>
      </div>

      <footer className={styles.footer}>
        Made with &#10084; by Crypto Devs
      </footer>
    </div>
  );
}
Vamos executá-lo! No seu terminal, no my-appdiretório, execute:

npm run dev
para ver seu site em ação. Deve se parecer com a captura de tela no início deste tutorial.

Parabéns! Seu site CryptoDevs DAO agora deve estar funcionando.

teste
Crie algumas propostas
Tente votar YAYem um e NAYno outro
Aguarde 5 minutos para que seus prazos passem
Execute ambos.
Veja o saldo do Tesouro DAO cair devido 0.1 ETHà proposta que passou ao comprar uma NFT na execução.
Enviar para o Github
Certifique-se de enviar todo esse código para o Github antes de prosseguir para a próxima etapa.

Implantação de site
De que serve um site se você não pode compartilhá-lo com outras pessoas? Vamos trabalhar na implantação do seu dApp no ​​mundo para que você possa compartilhá-lo com todos os seus amigos do LearnWeb3DAO.

Acesse Vercel Dashboard e faça login com sua conta do GitHub.
Clique no New Projectbotão e selecione seu DAO-Tutorialrepositório.
Ao configurar seu novo projeto, Vercel permitirá que você personalize seuRoot Directory
Como nosso aplicativo Next.js está em uma subpasta do repositório, precisamos modificá-lo.
Clique Editao lado de Root Directorye defina-o como my-app.
Selecione a estrutura comoNext.js
CliqueDeploy


Agora você pode ver seu site implantado acessando seu painel Vercel, selecionando seu projeto e copiando o domínio de lá!
PARABÉNS! Você está pronto!
Espero que você tenha gostado deste tutorial. Não se esqueça de compartilhar seu site DAO no #showcasecanal no Discord :D