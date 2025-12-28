# Keymaster

A simple allowlist/whitelist manager smart contract for Stacks blockchain built with Clarity. Control access, manage permissions, and enable exclusive minting on-chain.

## What It Does

Keymaster allows you to:
- Add addresses to allowlist
- Remove addresses from allowlist
- Check if address is allowed
- Mint tokens/NFTs only if whitelisted
- Batch add multiple addresses
- Track allowlist size and members

Perfect for:
- NFT presale/whitelist mints
- Exclusive access control
- Private beta access
- Learning access control patterns
- Gated content systems
- Permission management

## Features

- **Admin Controls**: Owner manages the allowlist
- **Batch Operations**: Add multiple addresses at once
- **Mint Integration**: Only whitelisted can mint
- **Status Checking**: Verify access before operations
- **Public Verification**: Anyone can check allowlist status
- **Flexible Management**: Add/remove addresses anytime
- **Transparent List**: All addresses visible on-chain

## Prerequisites

- [Clarinet](https://github.com/hirosystems/clarinet) installed
- Basic understanding of Stacks blockchain
- A Stacks wallet for testnet deployment

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/keymaster.git
cd keymaster

# Check Clarinet installation
clarinet --version
```

## Project Structure

```
keymaster/
├── contracts/
│   └── keymaster.clar       # Main allowlist contract
├── tests/
│   └── keymaster_test.ts    # Contract tests
├── Clarinet.toml            # Project configuration
└── README.md
```

## Usage

### Deploy Locally

```bash
# Start Clarinet console
clarinet console

# Add address to allowlist (admin only)
(contract-call? .keymaster add-to-allowlist 
  'ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC
)

# Check if address is allowed
(contract-call? .keymaster is-allowed 
  'ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC
)

# Mint (only if whitelisted)
(contract-call? .keymaster mint)

# Remove from allowlist (admin only)
(contract-call? .keymaster remove-from-allowlist 
  'ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC
)
```

### Contract Functions

**add-to-allowlist (address)**
```clarity
(contract-call? .keymaster add-to-allowlist 
  'ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC
)
```
Add an address to the allowlist (admin only)

**add-batch-to-allowlist (addresses)**
```clarity
(contract-call? .keymaster add-batch-to-allowlist 
  (list 
    'ST1ADDR1
    'ST1ADDR2
    'ST1ADDR3
  )
)
```
Add multiple addresses at once (admin only)

**remove-from-allowlist (address)**
```clarity
(contract-call? .keymaster remove-from-allowlist 
  'ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC
)
```
Remove an address from allowlist (admin only)

**is-allowed (address)**
```clarity
(contract-call? .keymaster is-allowed 
  'ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC
)
```
Check if an address is on the allowlist

**mint**
```clarity
(contract-call? .keymaster mint)
```
Mint token/NFT (only if caller is whitelisted)

**get-allowlist-size**
```clarity
(contract-call? .keymaster get-allowlist-size)
```
Returns total number of addresses on allowlist

**get-allowlist**
```clarity
(contract-call? .keymaster get-allowlist)
```
Returns list of all allowed addresses

**transfer-admin (new-admin)**
```clarity
(contract-call? .keymaster transfer-admin 
  'ST1NEW_ADMIN
)
```
Transfer admin rights to new address (current admin only)

**get-admin**
```clarity
(contract-call? .keymaster get-admin)
```
Returns current admin address

## How It Works

### Adding to Allowlist
1. Admin calls `add-to-allowlist` with address
2. Address marked as allowed in mapping
3. Allowlist counter incremented
4. Address can now access restricted functions

### Checking Access
1. User attempts restricted action (like minting)
2. Contract checks if address is on allowlist
3. If yes: Action proceeds
4. If no: Transaction fails with error

### Minting (Example Integration)
1. User calls `mint` function
2. Contract verifies caller is on allowlist
3. If allowed: Token/NFT minted to caller
4. If not: Error returned
5. Can track minted status per address

### Admin Management
- Only admin can modify allowlist
- Admin can add/remove addresses
- Admin rights can be transferred
- All operations require admin authorization

## Data Structure

### Allowlist Entry
```clarity
{
  address: principal,
  added-at: uint,
  added-by: principal,
  status: bool
}
```

### User Mint Status
```clarity
{
  has-minted: bool,
  mint-count: uint,
  first-mint-at: uint
}
```

### Storage Pattern
```clarity
;; Map to track allowed addresses
(define-map allowlist principal bool)

;; Map to track mint status
(define-map mint-status principal {
  has-minted: bool,
  mint-count: uint
})

;; Admin address
(define-data-var admin principal tx-sender)

;; Allowlist size counter
(define-data-var allowlist-size uint u0)

;; Total minted counter
(define-data-var total-minted uint u0)
```

## Testing

```bash
# Run all tests
npm run test

# Check contract syntax
clarinet check

# Run specific test
npm run test -- keymaster
```

## Learning Goals

Building this contract teaches you:
- ✅ Access control patterns
- ✅ Admin/owner permissions
- ✅ Allowlist management
- ✅ Minting logic integration
- ✅ Batch operations
- ✅ Authorization checks

## Example Use Cases

**NFT Presale:**
```clarity
;; Admin adds whitelist for presale
(contract-call? .keymaster add-batch-to-allowlist 
  (list 
    'ST1EARLY_SUPPORTER1
    'ST1EARLY_SUPPORTER2
    'ST1EARLY_SUPPORTER3
  )
)

;; Whitelisted users can mint early
(contract-call? .keymaster mint)  ;; Only works if whitelisted
```

**Private Beta Access:**
```clarity
;; Add beta testers
(contract-call? .keymaster add-to-allowlist 'ST1TESTER1)
(contract-call? .keymaster add-to-allowlist 'ST1TESTER2)

;; Check access before allowing features
(contract-call? .keymaster is-allowed tx-sender)
```

**Gated Community:**
```clarity
;; Add verified members
(contract-call? .keymaster add-to-allowlist 'ST1MEMBER)

;; Members can access exclusive content
;; Non-members are blocked
```

**Exclusive Token Sale:**
```clarity
;; Whitelist approved buyers
(contract-call? .keymaster add-batch-to-allowlist approved-buyers)

;; Only whitelisted can participate in sale
(contract-call? .keymaster mint)
```

## Access Control Flow

### Complete Lifecycle:
```
1. ADMIN SETUP → Deploy contract, become admin
   ↓
2. ADD TO ALLOWLIST → Admin adds approved addresses
   ↓
3. VERIFICATION → Users check if they're allowed
   ↓
4. ACCESS GRANTED → Whitelisted users can mint/access
   ↓
5. MINT/ACTION → Restricted function executes
   ↓
6. MANAGEMENT → Admin can add/remove as needed
```

## Access Patterns

### Allowlist Verification:
```clarity
;; Before attempting restricted action
(let ((allowed (unwrap! 
  (contract-call? .keymaster is-allowed tx-sender)
  ERR_NOT_ALLOWED
)))
  (asserts! allowed ERR_NOT_ALLOWED)
  ;; Proceed with action
)
```

### Admin Check:
```clarity
;; Verify caller is admin
(asserts! (is-eq tx-sender (var-get admin)) ERR_NOT_ADMIN)
```

### Mint Guard:
```clarity
;; Only whitelisted can mint
(define-public (mint)
  (let ((allowed (default-to false (map-get? allowlist tx-sender))))
    (asserts! allowed ERR_NOT_WHITELISTED)
    ;; Mint logic here
    (ok true)
  )
)
```

## Common Patterns

### Setup Allowlist
```clarity
;; Deploy contract (you become admin)

;; Add initial addresses
(contract-call? .keymaster add-to-allowlist 'ST1USER1)
(contract-call? .keymaster add-to-allowlist 'ST1USER2)

;; Or add in batch
(contract-call? .keymaster add-batch-to-allowlist 
  (list 'ST1USER1 'ST1USER2 'ST1USER3 'ST1USER4 'ST1USER5)
)
```

### Check Before Action
```clarity
;; User checks if they're whitelisted
(contract-call? .keymaster is-allowed tx-sender)

;; If true, proceed to mint
(contract-call? .keymaster mint)
```

### Manage List During Campaign
```clarity
;; Add new approved users
(contract-call? .keymaster add-to-allowlist 'ST1NEWUSER)

;; Remove users if needed (wrong address, etc.)
(contract-call? .keymaster remove-from-allowlist 'ST1OLDUSER)

;; Check current list size
(contract-call? .keymaster get-allowlist-size)
```

### Transfer Control
```clarity
;; Transfer admin to multisig or new admin
(contract-call? .keymaster transfer-admin 'ST1NEW_ADMIN)

;; New admin can now manage allowlist
```

## Allowlist Examples

### Small Exclusive Drop (10 people):
```clarity
Allowlist Size: 10
Total Supply: 10
Status: Highly exclusive
Access: Private list
```

### Community Presale (100 people):
```clarity
Allowlist Size: 100
Total Supply: 1000
Status: Early access
Access: Community members
```

### Large Whitelist (1000+ people):
```clarity
Allowlist Size: 1000+
Total Supply: 10000
Status: Presale phase
Access: Verified users
```

## Integration Patterns

### With NFT Contract:
```clarity
;; NFT contract checks keymaster
(define-public (mint-nft)
  (let ((allowed (contract-call? .keymaster is-allowed tx-sender)))
    (asserts! (is-eq allowed true) ERR_NOT_WHITELISTED)
    ;; Mint NFT logic
  )
)
```

### With Token Sale:
```clarity
;; Token sale verifies whitelist
(define-public (buy-tokens (amount uint))
  (let ((allowed (contract-call? .keymaster is-allowed tx-sender)))
    (asserts! (is-eq allowed true) ERR_NOT_WHITELISTED)
    ;; Token purchase logic
  )
)
```

### With Content Access:
```clarity
;; Gated content checks access
(define-read-only (get-premium-content)
  (let ((allowed (contract-call? .keymaster is-allowed tx-sender)))
    (asserts! (is-eq allowed true) ERR_NO_ACCESS)
    ;; Return premium content
  )
)
```

## Deployment

### Testnet
```bash
clarinet deployments generate --testnet --low-cost
clarinet deployments apply -p deployments/default.testnet-plan.yaml
```

### Mainnet
```bash
clarinet deployments generate --mainnet
clarinet deployments apply -p deployments/default.mainnet-plan.yaml
```

## Roadmap

- [ ] Write the core contract
- [ ] Add comprehensive tests
- [ ] Deploy to testnet
- [ ] Add time-based whitelist (expires after date)
- [ ] Implement tiered access levels
- [ ] Add mint limits per address
- [ ] Support multiple admins/roles
- [ ] Create allowlist snapshots
- [ ] Add merkle tree verification (gas efficient)

## Advanced Features (Future)

**Tiered Access:**
- Gold tier: Mint 5 NFTs
- Silver tier: Mint 3 NFTs
- Bronze tier: Mint 1 NFT

**Time-Based Access:**
- Early access: Block 0-100
- General access: Block 100-200
- Public sale: Block 200+

**Mint Limits:**
- Max mints per address
- Global supply cap
- Phase-based limits

**Multiple Admins:**
- Role-based access
- Multi-sig admin
- Delegated management

**Merkle Proofs:**
- Gas-efficient verification
- Off-chain allowlist
- On-chain proof verification

## Security Features

- ✅ Only admin can modify allowlist
- ✅ Admin rights transferable (for multisig)
- ✅ Cannot add same address twice
- ✅ Transparent allowlist (anyone can verify)
- ✅ Immutable mint records
- ✅ Access control on sensitive functions

## Best Practices

**Managing Allowlist:**
- Double-check addresses before adding
- Test with small batch first
- Keep backup of allowlist
- Announce allowlist additions publicly
- Be transparent about criteria

**As Admin:**
- Use multisig for admin if possible
- Test thoroughly before mainnet
- Have clear rules for allowlist
- Communicate with community
- Be fair and consistent

**For Users:**
- Verify you're on allowlist before mint
- Check allowlist status early
- Understand mint rules
- Don't share allowlist access
- Report suspicious activity

## Important Notes

⚠️ **Admin Responsibilities:**
- Admin has full control over allowlist
- Wrong addresses can be added/removed
- Admin keys must be secured
- Consider multisig for mainnet

💡 **Gas Optimization:**
- Use batch operations when possible
- Large allowlists cost more gas
- Consider merkle trees for 1000+ addresses
- Read operations are free

🎯 **Allowlist Strategy:**
- Clear criteria for inclusion
- Fair selection process
- Public announcement
- Verification period
- Timely additions

## Limitations

**Current Constraints:**
- Admin-only management (not democratic)
- All addresses equal weight
- No automatic expiration
- List stored on-chain (gas costs)

**Design Choices:**
- Simple binary allow/deny
- Admin control for flexibility
- Transparent for fairness
- Batch ops for efficiency

## Allowlist Sizes

### Small (1-50):
- Highly exclusive
- Personal connections
- Early believers
- Close community

### Medium (51-500):
- Community presale
- Active members
- Verified users
- Discord whitelist

### Large (501-5000):
- Public presale
- Multiple phases
- Broad community
- Consider merkle trees

### Massive (5000+):
- Public sale
- Multiple tiers
- Must use merkle proofs
- High gas costs otherwise

## Common Errors

```clarity
ERR_NOT_ADMIN (u100)
- You're not the admin
- Only admin can manage allowlist

ERR_NOT_WHITELISTED (u101)
- Address not on allowlist
- Cannot perform restricted action

ERR_ALREADY_MINTED (u102)
- Already minted maximum amount
- Check mint limits

ERR_MINT_CLOSED (u103)
- Minting period ended
- Too late to mint
```

## Use Case Comparisons

| Use Case | List Size | Access Type | Best Practice |
|----------|-----------|-------------|---------------|
| NFT Drop | 100-1000 | Presale | Batch add, time-based |
| Beta Access | 10-100 | Private | Careful selection |
| Token Sale | 500-5000 | Presale | Tiered access |
| DAO Members | 50-500 | Exclusive | Verified only |
| Content Gate | Any | Subscription | Regular updates |

## Contributing

This is a learning project! Feel free to:
- Open issues for questions
- Submit PRs for improvements
- Fork and experiment
- Build access control systems

## License

MIT License - do whatever you want with it

## Resources

- [Clarity Language Reference](https://docs.stacks.co/clarity)
- [Clarinet Documentation](https://github.com/hirosystems/clarinet)
- [Stacks Blockchain](https://www.stacks.co/)
- [Access Control Patterns](https://book.clarity-lang.org/)
- [Merkle Trees Explained](https://en.wikipedia.org/wiki/Merkle_tree)

---

Built while learning Clarity 🔑

## Philosophy

"Access control is not about keeping people out - it's about letting the right people in."

Guard the gate. Control access. Be the Keymaster. 🗝️

---

**Your Contract:**
- Admin: ???
- Allowlist Size: ???
- Total Minted: ???
- Access Status: ???

**Who holds the keys?** 🚪
