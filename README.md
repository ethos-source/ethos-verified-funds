# ethos-verified-funds

## Abstract

Ethos-Verified-Funds is a smart contract that programatically and deterministically validates the origination of transactions that come from verified intitutions [(data/ethos-vsf-institutions.json)](https://raw.githubusercontent.com/ethos-source/ethos-verified-funds/master/data/ethos-vsf-institution.json). Ethos-Verified-Funds acts as a registry that, given a wallet address, can issue claims (tokens) validating the authenticity of the source of the funds associated to that address. The goal of this registry is to provide a central point of reference for on-chain claims related to the source of funds for given investements.

## Specification

The Ethos-Verified-Funds smart contract provides an interface for adding, getting, and removing claims. Claims are issued from an `issuer` to a `subject` with a `key`, which is of type `bytes32`. The claims data is then stored as type `bytes32`.

### setClaim
To be used by an `issuer` to set the claim `value` with the `key` about the `subject`.

function setClaim(address subject, bytes32 key, bytes32 value) public;

### setSelfClaim
To be used by an `issuer` to set a claim about themself.

function setSelfClaim(bytes32 key, bytes32 value) public;

### getClaim
Used by anyone to verify a specific claim.

function getClaim(address issuer, address subject, bytes32 key) public constant returns (bytes32);

### removeClaim
To be used by the `issuer` to remove a claim it has made, or by a `subject` to remove a claim made by someone about the `subject`.

function removeClaim(address issuer, address subject, bytes32 key) public;

## Implementation
```
contract Ethos-Verified-Funds {

    mapping(address => mapping(address => mapping(bytes32 => bytes32))) public registry;

    event ClaimSet(
        address indexed issuer,
        address indexed subject,
        bytes32 indexed key,
        bytes32 value,
        uint updatedAt);

    event ClaimRemoved(
        address indexed issuer,
        address indexed subject,
        bytes32 indexed key,
        uint removedAt);

    // create or update clams
    function setClaim(address subject, bytes32 key, bytes32 value) public {
        registry[msg.sender][subject][key] = value;
        ClaimSet(msg.sender, subject, key, value, now);
    }

    function setSelfClaim(bytes32 key, bytes32 value) public {
        setClaim(msg.sender, key, value);
    }

    function getClaim(address issuer, address subject, bytes32 key) public constant returns(bytes32) {
        return registry[issuer][subject][key];
    }

    function removeClaim(address issuer, address subject, bytes32 key) public {
        require(msg.sender == issuer || msg.sender == subject);
        delete registry[issuer][subject][key];
        ClaimRemoved(msg.sender, subject, key, now);
    }
}
```
