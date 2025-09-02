# Pinocchio Vault Rust

A secure Solana program implementation for managing SOL deposits and withdrawals using Program Derived Addresses (PDAs). Built with the Pinocchio framework for optimized no-std development.

## Overview

This vault program allows users to:
- **Deposit SOL** into their personal vault account
- **Withdraw all SOL** from their vault back to their wallet
- **Secure storage** using PDA-based account derivation

## Program Architecture

### Program ID
```
F1EK4ZQeKCJQCEMVXZBuRczNBXttZyh8jv13TPC9Rx7
```

### Account Structure

#### Vault Account
- **Derivation**: `[b"vault", owner_pubkey]`
- **Owner**: System Program
- **Purpose**: Stores user's deposited SOL lamports

### Instructions

#### 1. Deposit
**Discriminator**: `0`

Transfers SOL from the user's wallet to their vault account.

**Accounts**:
- `[signer, writable]` Owner - The user depositing SOL
- `[writable]` Vault - PDA account to receive the deposit
- `[]` System Program - Required for SOL transfers

**Instruction Data**:
```rust
struct DepositInstructionData {
    amount: u64, // Amount in lamports to deposit
}
```

**Validation**:
- Owner must be a signer
- Amount must be greater than 0
- Vault must be derived correctly from owner's pubkey
- Vault must be owned by System Program
- Vault must have 0 lamports (new account)

#### 2. Withdraw
**Discriminator**: `1`

Transfers all SOL from the user's vault back to their wallet.

**Accounts**:
- `[signer, writable]` Owner - The user withdrawing SOL
- `[writable]` Vault - PDA account to withdraw from
- `[]` System Program - Required for SOL transfers

**Validation**:
- Owner must be a signer
- Vault must be derived correctly from owner's pubkey
- Vault must be owned by System Program
- Vault must have lamports > 0

## Technical Implementation

### Dependencies
- **pinocchio**: `0.9.2` - Core Solana program framework
- **pinocchio-system**: `0.3.0` - System program interactions

### Security Features

1. **PDA Verification**: All vault accounts are verified against the correct derivation path
2. **Signer Validation**: Only the vault owner can deposit/withdraw
3. **Account Ownership**: Strict validation of account ownership
4. **Amount Validation**: Deposit amounts must be non-zero
5. **State Validation**: Appropriate lamport balance checks

### Error Handling

The program returns appropriate `ProgramError` variants:
- `NotEnoughAccountKeys`: Insufficient accounts provided
- `InvalidAccountOwner`: Account ownership validation failed
- `InvalidAccountData`: Account state validation failed
- `InvalidInstructionData`: Instruction data parsing failed

## Build & Deploy

### Prerequisites
- Rust toolchain with `wasm32-unknown-unknown` target
- Solana CLI tools
- Anchor (optional, for testing)

### Build
```bash
cargo build-sbf
```

### Deploy
```bash
solana program deploy target/deploy/pinocchio_vault_rust.so
```

### Test
```bash
cargo test
```

## Usage Examples

### TypeScript Client Example

```typescript
import { Connection, PublicKey, Transaction, SystemProgram } from '@solana/web3.js';

const PROGRAM_ID = new PublicKey('F1EK4ZQeKCJQCEMVXZBuRczNBXttZyh8jv13TPC9Rx7');

// Derive vault PDA
const [vaultPda] = PublicKey.findProgramAddressSync(
  [Buffer.from('vault'), owner.publicKey.toBuffer()],
  PROGRAM_ID
);

// Create deposit instruction
const depositIx = new TransactionInstruction({
  programId: PROGRAM_ID,
  keys: [
    { pubkey: owner.publicKey, isSigner: true, isWritable: true },
    { pubkey: vaultPda, isSigner: false, isWritable: true },
    { pubkey: SystemProgram.programId, isSigner: false, isWritable: false },
  ],
  data: Buffer.concat([
    Buffer.from([0]), // Deposit discriminator
    Buffer.from(new BN(1000000).toArray('le', 8)) // 0.001 SOL
  ])
});

// Create withdraw instruction
const withdrawIx = new TransactionInstruction({
  programId: PROGRAM_ID,
  keys: [
    { pubkey: owner.publicKey, isSigner: true, isWritable: true },
    { pubkey: vaultPda, isSigner: false, isWritable: true },
    { pubkey: SystemProgram.programId, isSigner: false, isWritable: false },
  ],
  data: Buffer.from([1]) // Withdraw discriminator
});
```

### Rust Client Example

```rust
use solana_sdk::{
    instruction::{AccountMeta, Instruction},
    pubkey::Pubkey,
    system_program,
};

// Derive vault PDA
let (vault_pda, _bump) = Pubkey::find_program_address(
    &[b"vault", owner_pubkey.as_ref()],
    &program_id
);

// Create deposit instruction
let deposit_ix = Instruction::new_with_bytes(
    program_id,
    &[&[0u8], &amount.to_le_bytes()].concat(), // [discriminator, amount]
    vec![
        AccountMeta::new(owner_pubkey, true),
        AccountMeta::new(vault_pda, false),
        AccountMeta::new_readonly(system_program::ID, false),
    ],
);

// Create withdraw instruction
let withdraw_ix = Instruction::new_with_bytes(
    program_id,
    &[1u8], // Withdraw discriminator
    vec![
        AccountMeta::new(owner_pubkey, true),
        AccountMeta::new(vault_pda, false),
        AccountMeta::new_readonly(system_program::ID, false),
    ],
);
```

## Project Structure

```
pinocchio_vault_rust/
├── Cargo.toml          # Package configuration
├── Cargo.lock          # Dependency lockfile
├── src/
│   └── lib.rs          # Main program implementation
├── target/             # Build artifacts (gitignored)
└── README.md          # This file
```

## License

This project is open source. Please ensure you understand the security implications before using in production.

## Contributing

Contributions are welcome! Please ensure all code follows the existing patterns and includes appropriate tests.

## Security Considerations

- This is a demonstration program - audit thoroughly before production use
- PDA derivation ensures account security but review all validation logic
- Consider implementing additional access controls for production deployments
- Test extensively on devnet before mainnet deployment