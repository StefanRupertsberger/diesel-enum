This crate allows the user to represent state in the database using Rust enums. This is achieved
through a proc macro. First the macro looks at your chosen `sql_type`, and then it devises a
corresponding Rust type. The mapping is as follows:

| SQL | Rust |
|--|--|
| `SmallInt` | `i16` |
| `Integer` | `i32` |
| `Int` | `i32` |
| `BigInt` | `i64` |
| `VarChar` | `String` |
| `Text` | `String` |

 The macro then generates three impls: a `FromSql` impl, an `ToSql` impl and a
`TryFrom` impl, which allow conversion between the Sql type an the enum (`FromSql` and `ToSql`),
and from the Rust type into the enum (`TryInto`).
//!
### Usage
```rust
#[macro_use] extern crate diesel;
use diesel_enum::DbEnum;
use diesel::sql_types::SmallInt;

#[derive(Debug)]
pub struct CustomError {
    msg: String,
    status: u16,
}

impl CustomError {
    fn not_found(msg: String) -> Self {
        Self {
            msg,
            status: 404,
        }
    }
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, AsExpression, FromSqlRow, DbEnum)]
#[sql_type = "SmallInt"]
#[error_fn = "CustomError::not_found"]
#[error_type = "CustomError"]
pub enum Status {
    /// Will be represented as 0.
    Ready,
    /// Will be represented as 1.
    Pending,
}
```
Alternatively you can use strings, with will be cast to lowercase. (e.g. `Status::Ready` will be
stored as `"ready"` in the database):
```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, AsExpression, FromSqlRow, DbEnum)]
#[sql_type = "VarChar"]
#[error_fn = "CustomError::not_found"]
#[error_type = "CustomError"]
pub enum Status {
    /// Will be represented as `"ready"`.
    Ready,
    /// Will be represented as `"pending"`.
    Pending,
}
```