// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title VoteChain
 * @dev A decentralized voting system smart contract
 * @author VoteChain Team
 */
contract VoteChain {
    
    // State variables
    address public admin;
    uint256 public totalVotes;
    uint256 public votingEndTime;
    bool public votingActive;
    
    // Structures
    struct Candidate {
        uint256 id;
        string name;
        string description;
        uint256 voteCount;
        bool exists;
    }
    
    struct Voter {
        bool hasVoted;
        bool isRegistered;
        uint256 votedCandidateId;
        uint256 timestamp;
    }
    
    // Mappings
    mapping(uint256 => Candidate) public candidates;
    mapping(address => Voter) public voters;
    uint256[] public candidateIds;
    
    // Events
    event CandidateAdded(uint256 indexed candidateId, string name);
    event VoteCast(address indexed voter, uint256 indexed candidateId, uint256 timestamp);
    event VoterRegistered(address indexed voter);
    event VotingStarted(uint256 endTime);
    event VotingEnded(uint256 totalVotes);
    
    // Modifiers
    modifier onlyAdmin() {
        require(msg.sender == admin, "Only admin can perform this action");
        _;
    }
    
    modifier votingIsActive() {
        require(votingActive && block.timestamp < votingEndTime, "Voting is not active");
        _;
    }
    
    modifier hasNotVoted() {
        require(!voters[msg.sender].hasVoted, "You have already voted");
        _;
    }
    
    modifier isRegisteredVoter() {
        require(voters[msg.sender].isRegistered, "You are not registered to vote");
        _;
    }
    
    // Constructor
    constructor() {
        admin = msg.sender;
        votingActive = false;
        totalVotes = 0;
    }
    
    /**
     * @dev Core Function 1: Register a voter
     * @param _voterAddress Address of the voter to register
     */
    function registerVoter(address _voterAddress) external onlyAdmin {
        require(!voters[_voterAddress].isRegistered, "Voter is already registered");
        require(_voterAddress != address(0), "Invalid voter address");
        
        voters[_voterAddress].isRegistered = true;
        voters[_voterAddress].hasVoted = false;
        
        emit VoterRegistered(_voterAddress);
    }
    
    /**
     * @dev Core Function 2: Cast a vote
     * @param _candidateId ID of the candidate to vote for
     */
    function castVote(uint256 _candidateId) external 
        votingIsActive 
        isRegisteredVoter 
        hasNotVoted 
    {
        require(candidates[_candidateId].exists, "Candidate does not exist");
        
        // Record the vote
        voters[msg.sender].hasVoted = true;
        voters[msg.sender].votedCandidateId = _candidateId;
        voters[msg.sender].timestamp = block.timestamp;
        
        // Update candidate vote count
        candidates[_candidateId].voteCount++;
        totalVotes++;
        
        emit VoteCast(msg.sender, _candidateId, block.timestamp);
    }
    
    /**
     * @dev Core Function 3: Get voting results
     * @return candidateIds Array of candidate IDs
     * @return names Array of candidate names  
     * @return voteCounts Array of vote counts
     * @return winner ID of the winning candidate
     */
    function getResults() external view returns (
        uint256[] memory,
        string[] memory names,
        uint256[] memory voteCounts,
        uint256 winner
    ) {
        require(!votingActive || block.timestamp >= votingEndTime, "Voting is still active");
        
        string[] memory candidateNames = new string[](candidateIds.length);
        uint256[] memory candidateVotes = new uint256[](candidateIds.length);
        uint256 winningVoteCount = 0;
        uint256 winningCandidateId = 0;
        
        for (uint256 i = 0; i < candidateIds.length; i++) {
            uint256 candidateId = candidateIds[i];
            candidateNames[i] = candidates[candidateId].name;
            candidateVotes[i] = candidates[candidateId].voteCount;
            
            if (candidates[candidateId].voteCount > winningVoteCount) {
                winningVoteCount = candidates[candidateId].voteCount;
                winningCandidateId = candidateId;
            }
        }
        
        return (candidateIds, candidateNames, candidateVotes, winningCandidateId);
    }
    
    // Additional utility functions
    
    /**
     * @dev Add a new candidate (Admin only)
     */
    function addCandidate(uint256 _candidateId, string memory _name, string memory _description) 
        external onlyAdmin 
    {
        require(!candidates[_candidateId].exists, "Candidate already exists");
        require(bytes(_name).length > 0, "Candidate name cannot be empty");
        
        candidates[_candidateId] = Candidate({
            id: _candidateId,
            name: _name,
            description: _description,
            voteCount: 0,
            exists: true
        });
        
        candidateIds.push(_candidateId);
        emit CandidateAdded(_candidateId, _name);
    }
    
    /**
     * @dev Start voting period
     */
    function startVoting(uint256 _durationInMinutes) external onlyAdmin {
        require(!votingActive, "Voting is already active");
        require(_durationInMinutes > 0, "Duration must be greater than 0");
        
        votingEndTime = block.timestamp + (_durationInMinutes * 1 minutes);
        votingActive = true;
        
        emit VotingStarted(votingEndTime);
    }
    
    /**
     * @dev End voting period (Admin only)
     */
    function endVoting() external onlyAdmin {
        require(votingActive, "Voting is not active");
        
        votingActive = false;
        emit VotingEnded(totalVotes);
    }
    
    /**
     * @dev Get candidate information
     */
    function getCandidate(uint256 _candidateId) external view returns (
        uint256 id,
        string memory name,
        string memory description,
        uint256 voteCount
    ) {
        require(candidates[_candidateId].exists, "Candidate does not exist");
        
        Candidate memory candidate = candidates[_candidateId];
        return (candidate.id, candidate.name, candidate.description, candidate.voteCount);
    }
    
    /**
     * @dev Check if voter has voted
     */
    function hasVoterVoted(address _voter) external view returns (bool) {
        return voters[_voter].hasVoted;
    }
    
    /**
     * @dev Get total number of candidates
     */
    function getCandidateCount() external view returns (uint256) {
        return candidateIds.length;
    }
}
