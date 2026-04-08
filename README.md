# MemeWar
BaseMemeWar.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract BaseMemeWar {
    address public immutable owner;
    uint256 public nextMemeId;
    uint256 public constant VOTE_PRICE = 0.0003 ether;

    struct Meme {
        uint256 id;
        address creator;
        string description;
        uint256 votes;
        uint256 timestamp;
    }

    Meme[] public allMemes;
    mapping(uint256 => mapping(address => uint256)) public userVotes; // memeId => user => vote count
    mapping(uint256 => uint256) public totalVotesPerMeme;

    event MemeSubmitted(uint256 indexed memeId, address creator, string description);
    event Voted(uint256 indexed memeId, address voter, uint256 amount);
    event RewardsClaimed(address indexed claimer, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner");
        _;
    }

    // Meme
    function submitMeme(string memory _description) external {
        require(bytes(_description).length > 0 && bytes(_description).length <= 280, "Description 1-280 chars");

        allMemes.push(Meme({
            id: nextMemeId,
            creator: msg.sender,
            description: _description,
            votes: 0,
            timestamp: block.timestamp
        }));

        emit MemeSubmitted(nextMemeId, msg.sender, _description);
        nextMemeId++;
    }

    // Meme
    function vote(uint256 _memeId) external payable {
        require(_memeId < allMemes.length, "Meme does not exist");
        require(msg.value == VOTE_PRICE, "Vote costs exactly 0.0003 ETH");

        allMemes[_memeId].votes += 1;
        userVotes[_memeId][msg.sender] += 1;
        totalVotesPerMeme[_memeId] += 1;

        emit Voted(_memeId, msg.sender, msg.value);
    }

    // Memes
    function getLatestMemes(uint256 _count) external view returns (Meme[] memory) {
        uint256 start = allMemes.length > _count ? allMemes.length - _count : 0;
        Meme[] memory latest = new Meme[](_count);
        for (uint i = 0; i < _count && start + i < allMemes.length; i++) {
            latest[i] = allMemes[start + i];
        }
        return latest;
    }

    // Top Memes
    function getTopMemes(uint256 _count) external view returns (Meme[] memory) {
        Meme[] memory top = new Meme[](_count);
        // 
        uint256 start = allMemes.length > 50 ? allMemes.length - 50 : 0;
        for (uint i = 0; i < _count && start + i < allMemes.length; i++) {
            top[i] = allMemes[start + i];
        }
        return top;
    }

    // （Owner）
    function withdraw() external onlyOwner {
        (bool sent, ) = payable(owner).call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
    }

    function getTotalMemes() external view returns (uint256) {
        return allMemes.length;
    }

    function getContractInfo() external pure returns (string memory) {
        return "BaseMemeWar - Submit memes, vote with ETH, compete for the top spot!";
    }
}
