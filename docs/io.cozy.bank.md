[Table of contents](README.md#table-of-contents)

# Cozy Bank doctypes

Cozy can stores and manipulate bank related datas, distributed across several doctypes.


## `io.cozy.bank.settings`

This doctype store informations about Bank application settings. There is only one document, by default:

```
{
  notifications: {
    amountMax: {
      enable: false,
      value: 30
    }
  }
}
```


## `io.cozy.bank.accounts`

This doctypes stores informations about a Bank account:

- `label`: {string} - The bank account label (e.g. _John Doe Bank Account_)
- `institutionLabel`: {string} - The financial institution name the bank account belongs to
- `balance`: {number} - The current account balance
- `type`: {string} - The account type in the list `['none', 'bank', 'cash', 'asset', 'credit card', 'liability']`
- `number`: {string} - The bank account number in its institution
- `iban`: {string} - The bank account international identifier
- `serviceID`: {number} - In case of external service used to import transactions, this key can stores the service's account ID; can be `undefined` is the account isn't managed by any external service

### Example

```
{
  "_id": "2165d9a310deadbeeffc08d54c45102",
  "balance": 1337.73,
  "institutionLabel": "Société Générale (Particuliers)",
  "label": "Livret Dévelop. Durable (x1337)",
  "linxoId": "123456",
  "metadata": {
    "version": 1
  },
  "number": "03791 00048085818",
  "shortLabel": "Livret Dévelop. Durable",
  "type": "Savings"
}
```

## `io.cozy.bank.accounts.stats`

This doctype stores aggregated data about bank accounts:

- `periodStart`: {date} - The start of the reference period
- `periodEnd`: {date} - The end of the reference period
- `income`: {number} - The average income on the reference period
- `additionalIncome`: {number} - The average additional income on the reference period
- `mortgage`: {number} - The average mortgage expense on the reference period
- `loans`: {number} - The average total loans expense on the reference period
- `fixedCharges`: {number} - The average fixed charges on the reference period
- `currency`: {string} - A 3 uppercased chars defining the currecny used for the transaction as stated in [ISO4217](https://www.currency-iso.org/en/home/tables/table-a1.html)

### Relationships

- `account`: {object} - The associated account

### Example

```
{
  "_id": "f1aacd3254509687c07e48279b42cd50",
  "periodStart": "2019-03-01T00:00:00",
  "periodEnd": "2019-06-30T23:59:59",
  "income": 2345.67,
  "additionalIncome": 123.45,
  "mortgage": 567.89,
  "loans": 678.9,
  "fixedCharges": 234.56,
  "currency": "EUR",
  "relationships": {
    "account": {
      "data": {
        "_id": "f1aacd3254509687c07e48279b42f3de",
        "_type": "io.cozy.bank.accounts"
      }
    }
  }
}
```

## `io.cozy.bank.operations`

This doctype stors informations about a bank transaction:

- `label`: {string} - The label describing the transaction
- `type`: {string} - A type in the list : `['none', 'credit card', 'cash', 'check', 'transfer', 'internal transfer', 'debit card', 'deposit', 'financial fee', 'direct debit']`
- `date`: {date} - The date the transaction is emmited
- `dateOperation`: {date} - The date the transaction is registered in the account
- `dateImport`: {date} - The date the transaction is imported (can differ of the date of creation of the document as the import can be done by an external service)
- `amount`: {number} - The amount of the transaction
- `currency`: {string} - A 3 uppercased chars defining the currecny used for the transaction as stated in [ISO4217](https://www.currency-iso.org/en/home/tables/table-a1.html)
- `automaticCategoryId`: {string} - A category that apply to the transaction and is automatically calculated. This property should be hydrated by the connector. See the «&nbsp;categories&nbsp;» section below to know more about categories
- `manualCategoryId`: {string} - A category that apply to the transaction and is manually selected by the user
- `cozyCategoryId`: {string} - A category found by Cozy
- `cozyCategoryProba`: {number} - The probability of Cozy category
- `localCategoryId`: {string} - A category found by Cozy, based on the user recategorization habits
- `localCategoryProba`: {number} - The probability of local category
- `account`: {identifier} - The related account id the transaction belongs to
- `bills`: {array[<io.cozy.bills._id>]} - List of bills id.
- `reimbursements`: {array[<io.cozy.bills._id>]} - List of bills id.
- `creditOperations`: {array[<io.cozy.bank.operations._id>]} - List of credit operations id
- `debitOperations`: {array[<io.cozy.bank.operations._id>]} - List of debit operations id
- `parent`: {_id} - In case of a split transaction, the one refers the global transaction the split one belongs to


For the dates, any string or integer which can be interpreted by new Date(date) is possible but the
best the result of Date.toString() -> like 'Fri Mar 09 2018 19:04:40 GMT+0100 (CET)' which contains
the time zones.

### Categories

There are up to four categories that can be assigned to a `io.cozy.bank.operations` document. But there is only one that a connector have to set when retrieving the transactions from a bank: `automaticCategoryId`. This property have to be one of the categories ID listed [here](https://github.com/cozy/cozy-banks/blob/master/src/ducks/categories/tree.json). If you want to see more human labels, you can check the translations of each label in [english](https://github.com/cozy/cozy-banks/blob/master/src/locales/en.json) or [french](https://github.com/cozy/cozy-banks/blob/master/src/locales/fr.json).

Other category properties are to be set by other sources: the user or Cozy Banks services.

### Example

```
{
  "_id": "f0426fdeadbeef55755ee7f4d6555",
  "account": "15fb6402426bcdeadbeeffd5f4587bb",
  "amount": 10,
  "automaticCategoryId": "400110",
  "currency": "EUR",
  "date": "2017-09-22 00:00:00+01:00",
  "dateOperation": null,
  "label": "M  PIERRE RICHARD",
  "linxoId": "845811337",
  "cozyCategoryId": "400110",
  "cozyCategoryProba": 0.9323,
  "metadata": {
    "dateImport": "2018-03-09T09:23:40.075Z",
    "version": 1
  },
  "originalBankLabel": "VIR RECU    5383518660S DE: M  PIERRE RICHARD J MOTIF: Cascade REF: NOT PROVIDED",
  "type": "transfer"
}
```


## `io.cozy.bank.balancehistories`

This doctype stores a year of daily balances :

- `year`: {number} - The year of the balances
- `balances`: {object} - A day (YYYY-MM-DD format) / balance (number) map

### Relationships

- `account`: {object} - The associated account

### Example

```
{
  "_id": "f1aacd3254509687c07e48279b42cd50",
  "year": 2018,
  "balances": {
    "2018-01-01": 8000,
    "2018-01-02": 8000.42,
    "2018-01-03": 1337
  },
  "metadata": {
    "version": 1
  },
  "relationships": {
    "account": {
      "data": {
        "_id": "f1aacd3254509687c07e48279b42f3de",
        "_type": "io.cozy.bank.accounts"
      }
    }
  }
}
```
