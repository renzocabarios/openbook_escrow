# openbook_escrow


``` rust
use anchor_lang::prelude::*;

// This is your program's public key and it will update
// automatically when you build the project.
declare_id!("Fqb6LEBCPntKGYjZ4S5EeCoByhs1Jy5g4nfQDWrcv8jx");
pub const BOOK_SEED: &str = "BOOK_SEED";
pub const ESCROW_SEED: &str = "ESCROW_SEED";

pub const AUTHOR_LENGTH: usize = 32;
pub const NAME_LENGTH: usize = 32;

#[program]
mod hello_anchor {
    use super::*;

    pub fn initialize_book(
        ctx: Context<InitializeBook>,
        name: String,
        author: String,
        cost_per_day: u64,
    ) -> Result<()> {
        let initialized_book = &mut ctx.accounts.book_account;

        let mut name_data = [0u8; NAME_LENGTH];
        name_data[..name.as_bytes().len()].copy_from_slice(name.as_bytes());
        initialized_book.name = name_data;

        let mut author_data = [0u8; AUTHOR_LENGTH];
        author_data[..author.as_bytes().len()].copy_from_slice(author.as_bytes());
        initialized_book.author = author_data;

        initialized_book.bump = ctx.bumps.book_account;
        initialized_book.author_length = author.as_bytes().len() as u8;
        initialized_book.name_length = name.as_bytes().len() as u8;
        initialized_book.book_authority = ctx.accounts.book_authority.key();
        initialized_book.cost_per_day = cost_per_day;
        initialized_book.available = true;

        Ok(())
    }

    pub fn initialize_escrow(ctx: Context<InitializeEscrow>) -> Result<()> {
        let book_account = &mut ctx.accounts.book_account;
        let escrow_account = &mut ctx.accounts.escrow_account;

        //         pub lendee: Pubkey,
        // pub book: Pubkey,
        // pub total: u64,
        // pub bump: u8,
        // let mut name_data = [0u8; NAME_LENGTH];
        // name_data[..name.as_bytes().len()].copy_from_slice(name.as_bytes());
        // initialized_book.name = name_data;

        // let mut author_data = [0u8; AUTHOR_LENGTH];
        // author_data[..author.as_bytes().len()].copy_from_slice(author.as_bytes());
        // initialized_book.author = author_data;

        escrow_account.lendee = book_account.book_authority.key();
        escrow_account.book = book_account.key();
        escrow_account.book = book_account.key();

        // TODO: the amount to pay should be days * book_account.cost_per_day
        escrow_account.bump = ctx.bumps.escrow_account;

        book_account.available = false;
        // escrow_account.author_length = author.as_bytes().len() as u8;
        // escrow_account.name_length = name.as_bytes().len() as u8;
        // escrow_account.cost = cost_per_day;
        // escrow_account.available = false;

        Ok(())
    }
}

#[derive(Accounts)]
pub struct InitializeEscrow<'info> {
    #[account(mut,    
        seeds = [
        book_account.name[..book_account.name_length as usize].as_ref(),
        BOOK_SEED.as_bytes(),
        book_account.book_authority.key().as_ref(),
        ],
        bump)]
    pub book_account: Account<'info, BookAccount>,
    #[account(
        init,
        payer = lendee,
        space = 8 + EscrowAccount::LEN,
        seeds = [
            ESCROW_SEED.as_bytes(),
            lendee.key().as_ref(),
            book_account.key().as_ref(),
            ],
        bump)]
    pub escrow_account: Account<'info, EscrowAccount>,
    #[account(mut)]
    pub lendee: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[account]
pub struct EscrowAccount {
    pub lendee: Pubkey,
    pub book: Pubkey,
    pub days: u64,
    pub bump: u8,
}

impl EscrowAccount {
    // Pubkey + [u8; TOPIC_LENGTH] + u8 + [u8; CONTENT_LENGTH] + u64 + u64 + u8
    pub const LEN: usize = 32 + AUTHOR_LENGTH + 1 + AUTHOR_LENGTH + 1 + 8 + 1;
}

#[derive(Accounts)]
#[instruction(name: String)]
pub struct InitializeBook<'info> {
    #[account(
        init,
        payer = book_authority,
        space = 8 + BookAccount::LEN,
        seeds = [
            name.as_bytes(),
            BOOK_SEED.as_bytes(),
            book_authority.key().as_ref()
            ],
        bump)]
    pub book_account: Account<'info, BookAccount>,
    #[account(mut)]
    pub book_authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[account]
pub struct BookAccount {
    pub book_authority: Pubkey,
    pub author: [u8; AUTHOR_LENGTH],
    pub author_length: u8,
    pub name: [u8; AUTHOR_LENGTH],
    pub name_length: u8,
    pub cost_per_day: u64,
    pub available: bool,
    pub bump: u8,
}
impl BookAccount {
    // Pubkey + [u8; TOPIC_LENGTH] + u8 + [u8; CONTENT_LENGTH] + u64 + u64 + u8
    pub const LEN: usize = 32 + AUTHOR_LENGTH + 1 + AUTHOR_LENGTH + 1 + 8 + 1;
}


```
