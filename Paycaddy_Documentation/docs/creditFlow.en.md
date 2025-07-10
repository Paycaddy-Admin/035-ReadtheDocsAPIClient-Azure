As an alternative to the debit and prepaid based systems, the credit core gives you the flexibility to offer credit lines to your users based on your own criteria. This feature allows you to configure and manage credit lines to individual user profiles.

With the credit core, users are assigned credit wallets linked to specific credit line settings. From there, they can begin making transactions using their available credit, just as they would with the balance of a prefunded wallet.

---
## Credit Products

It is the heart of our credit core as it is what governs how each credit line will behave over time. Think of it as the "rule-book" that defines the terms and conditions of credit usage for your users.

Each credit wallet issued to a user is tied to a credit product, which ensures consistent behavior across similar credit lines and allows you to manage risk, profitability and flexibility with precision and granularity.

They are provided by PayCaddy based on agreements/requests with the clients previously. You can have as much credit products as you consider necessary although every credit product request is reviewed internally to avoid conflicts or misunderstandings on how it scales over time.

It is build to be as granular as possible, you can control from basic settings as grace period, cut frequency and interest rate. Until the more specific ones such as base minimum payment rate, if it is revolving or not, the capital interest split, etc.

The full list of adjustable parameters can be found in the table below:

| Field                         | Description                                                                                                                                                                                                                                                                                                     | Example                                                     |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| **revolving**                 | Specifies whether the credit line is reinstated following the submission of a payment report. If set to **false**, the value reported is not restored.                                                                                                                                                          | `true` / `false`                                            |
| **compound**                  | Indicates whether the interest calculation at the statement closing should be based on the sum of interests owed and amount owed (**true**), or only on the amount owed (**false**).                                                                                                                            | `true` / `false`                                            |
| **cutFrequency**              | Specifies the duration of a credit line cycle, determining how frequently statements are generated.                                                                                                                                                                                                             | “daily”, “weekly”, “biweekly”, “month”, “year”              |
| **fixedInterestAmount**       | Fixed amount added to interest amount when calculating interest. Example formula when **compound** is `true`: `fixedInterestAmount + baseInterestRate * ([amount owed] + [interest owed]) + [amount owed]`.                                                                                                     | In cents: `1000` → USD 10                                   |
| **baseInterestRate**          | Percentage applied to amount owed to calculate interests at the end of each cycle.                                                                                                                                                                                                                              | Percentage: 2 → 2%; 5 → 5%; 100 → 100%; 150 → 150%          |
| **baseMinimumPaymentRate**    | Minimum percentage of the amount owed that must be covered by the total payments reported during a cycle to prevent the wallet from being considered delinquent.                                                                                                                                                | Percentage: 2 → 2%; 5 → 5%; 100 → 100 %(max.)               |
| **fixedMinimumPaymentAmount** | A fixed amount set as the minimum expected payment—whether on its own or combined with other payments—to keep a wallet from being considered delinquent. At the time of statement closing, this amount is added to the result of the base minimum payment rate to determine the total minimum payment required. | In cents: `1000` → USD 10                                   |
| **capitalInterestSplit**      | Percentage of the reported payment amount that must be applied to the interest owed, with the remaining portion applied to the amount owed.                                                                                                                                                                     | Percentage: 2 → 2%; 5 → 5%; 99 → 99 %(max.)                 |
| **gracePeriod**               | Number of days after the statement closing date within which the client must report a payment to avoid being considered delinquent. Must be greater than 0.                                                                                                                                                     | In days: 1 → 1 day; 2 → 2 days; 3 → 3 days                  |
| **variableSpecs**             | Indicates whether the client is allowed to change the associated Credit Policy after a credit wallet has been created, using the **[Change Interest Credit Capital](creditCore.en.md#change-interest-credit-capital-post)** operation.                                                                          | `true` / `false`                                            |
| **morosidadRate**             | Percentage of interest owed. Added to the interest calculation when a wallet is delinquent.                                                                                                                                                                                                                     | Percentage: 2 → 2%; 5 → 5%; 100 → 100%; 150 → 150%          |
| **baseInterestMorosidad**     | Fixed amount added to the interest calculation when a wallet is delinquent.                                                                                                                                                                                                                                     | In cents: `1000` → USD 10                                   |
| **interestAccrualMethod**     | Method utilized for Interest generation at the end of each cycle.                                                                                                                                                                                                                                               | "ADB" for Average-Daily-Balance or "BAC" for Balance-At-Cut |
| **dailyRateDivisor**          | Number of days that should be accounted as a year for the dailyRate calculation.                                                                                                                                                                                                                                | 360 or 365                                                  |


---
## Interest Accrual Methods

PayCaddy supports two interest accrual methods for credit lines associated with wallets: **Average Daily Balance (ADB)** and **Balance at Cut**. The accrual method is defined by the `creditProductCode` used during wallet creation via the `POST /wallet-credits` endpoint.

### 1. Average Daily Balance (ADB)

In the ADB method, interest is calculated based on the average of daily balances recorded throughout the billing cycle. The process is as follows:

- The annual effective interest rate (`annualInterestRate`) is divided by a configurable parameter called `rateDivisor`, which may be 360 or 365.
    
    ```
    dailyInterestRate = annualInterestRate / rateDivisor
    ```
    
- At the end of each day during the cycle, the wallet’s balance is recorded and accumulated into the `sumOfDailyBalance`.
    
- At the end of the cycle (`cutDate`), the interest generated is calculated as:
    
    ```
    interestGenerated = dailyInterestRate * sumOfDailyBalance
    ```
    
- After the grace period (defined by `gracePeriod` in the credit product), the interest is applied. It may be:
    
    - **Capitalized**: added to the `amountOwed` (principal), or
        
    - **Non-capitalized**: added to `interestOwed`, without affecting the principal.
        

> **Note:** Whether interest is capitalized is controlled by the `compound` parameter in the credit product.

### 2. Balance at Cut

With this method, interest is calculated based solely on the balance owed at the time of the cycle cut.

- On the cut date (`cutDate`), the current `amountOwed` is retrieved.
    
- The daily interest rate (same formula as in ADB) is applied directly to this balance:
    
    ```
    interestGenerated = baseInterestRate * amountOwed + fixedInterestAmount
    ```
    
- At the end of the grace period, the generated interest is applied:
    
    - Capitalized or not, depending on the credit product configuration.
        

> This method does **not** take into account the balance changes throughout the cycle—only the state at the cut date matters.

### Related Parameters

| Field                   | Description                                                                    |
| ----------------------- | ------------------------------------------------------------------------------ |
| `interestAccrualMethod` | Accrual method: `"average_daily_balance"` or `"balance_at_cut"`                |
| `rateDivisor`           | Defines whether the daily rate uses 360 or 365 days                            |
| `gracePeriodDays`       | Number of days between the cut date and when interest is applied               |
| `interestCompounds`     | `true` if interest compounds into the principal; `false` if tracked separately |


---
## Credit Notifications

Notifications from PayCaddy's credit core are sent via webhooks. These events allow clients to stay in sync with changes to wallet balances, interest, limits, and more. To receive these events, register a callback URL via the [`NotificationsEnlist`](./notificationsEnlist.en) endpoint.

!!!Warning
	We highly recommend to use a different **URL** for credit notifications 

### Notification Workflow

1. **Client** registers `creditNotificationsURL` once per environment.
    
2. **PayCaddy** validates the URL and starts sending webhook POST requests in near real-time.
    
3. Your system should **verify the signature**, parse the JSON, and update your internal records accordingly.
    

### Event Payload Format

Each webhook follows a standard structure:

```json
{
  "walletId": "string",
  "type": "Event Type String",
  "amount": 1234,
  "direction": "debit" | "credit",
  "affectsBalance": true,
  "walletDetails_after": {
    "principal_outstanding": 81500,
    "interest_accrued": 1200,
    "totalDebt": 82700,
    "limit": 12345,
    "amountOverdraft": 1234,
    "interestSpecs": {
      "interestRateNominal": 1234,
      "interestRateMoroso": 1234,
      "fixedInterestNominal": 1234,
      "fixedInterestMoroso": 1234
    },
    "minimumPaymentSpecs": {
      "minimumPaymentRate": 1234,
      "fixedMinimumPayment": 1234
    }
  }
}
```

### Supported Event Types

| `type`                      | Description                                                                          | Direction | Affects Balance | Notes                                                            |
| --------------------------- | ------------------------------------------------------------------------------------ | --------- | --------------- | ---------------------------------------------------------------- |
| `Interes Nominal Calculado` | Interest is calculated on cut date but not yet applied.                              | `debit`   | `false`         | Provisional—will be applied at end of grace period.              |
| `Interes Nominal Aplicado`  | Nominal interest is applied (compounding or not, per product setup).                 | `debit`   | `true`          | Adds to principal or interest depending on compounding settings. |
| `Interes Moroso Calculado`  | Penal interest is calculated for delinquent wallets.                                 | `debit`   | `false`         | No customer obligation until applied.                            |
| `Interes Moroso Aplicado`   | Penal interest is booked to the balance.                                             | `debit`   | `true`          | Adds to `interest_accrued`.                                      |
| `Reporte de Pago`           | A customer payment is reported and split between principal, interest, and overdraft. | `credit`  | `true`          | Updates all balances post-payment.                               |
| `Cambio de Limite`          | Credit limit is changed. Amount is the difference between new and old limit.         | `credit`  | `true`          | Only the `limit` field changes.                                  |

### Notes on `walletDetails_after`

The `walletDetails_after` object represents the full post-event state of the wallet. Fields like `principal_outstanding`, `interest_accrued`, and `totalDebt` should be used to refresh your internal state immediately.

Even if an event doesn't affect balances directly (e.g. interest calculated but not applied), this object reflects the most up-to-date ledger status.

---

## Wallet Creation

Once you have **(1) an approved credit product (`creditProductCode`)** and **(2) a callback URL for credit notifications**, you can open your first credit wallet with the **`Wallet Credit POST`** endpoint.

A credit wallet is a ledger that tracks borrowing against an assigned limit. It never carries a positive cash balance—only counters for **AA (available amount)**, **AO (outstanding principal)**, and **IO (outstanding interest)**.  

Send a `POST /v1/WalletCredits` with the following JSON body:

| Field               | Purpose                                                                                                     |
| ------------------- | ----------------------------------------------------------------------------------------------------------- |
| `userId`            | Links the wallet to a specific customer profile that already exists in your tenant.                         |
| `currency`          | ISO 4217 currency code for all amounts (e.g., `"USD"`).                                                     |
| `description`       | Free‑text label to help you identify the wallet in dashboards and reports.                                  |
| `limit`             | Credit‑line ceiling in **cents** (e.g., `100000` = USD 1 000.00).                                           |
| `firstCutDate`      | ISO 8601 timestamp that marks the start of the first billing cycle and anchors all future cut dates.        |
| `time`              | Number of days the credit line remains active; after expiry, new transactions are denied.                   |
| `creditProductCode` | Three‑digit code, issued by PayCaddy, that applies the rules defined in **Credit Products** to this wallet. |

> **Note**  
> The field `time` is optional. If `time` is omitted, the wallet remains active indefinitely (subject to your own program rules).

After the call returns **HTTP 201** with the wallet `id`, you may immediately link cards or virtual PANs to this wallet. All subsequent events—interest accruals, limit changes, payments, and so on—will be delivered to the callback URL you registered in **Credit Notifications**, keeping your internal ledgers and user interfaces in sync with PayCaddy’s credit core.

---
## Credit Usage

Once a credit wallet is linked to one or more cards, the cardholder can spend up to the wallet’s **limit** during each billing cycle. Every purchase reduces the **available amount (AA)**—no funds are ever drawn from a cash balance.

#### Day‑to‑Day Management

| Action                        | Effect on the wallet                                                                                                                                                                            | Typical endpoint                                               |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| **Report a Payment**          | Decreases **AO** (outstanding principal) and, if interest has been executed, **IO** (outstanding interest). The same amount is immediately restored to **AA**. Payments may be partial or full. | [ReportPayCredit](creditCore.en.md#report-pay-credit-post)     |
| **Report a Charge**           | Registers a closed‑loop purchase or any miscellaneous fee you want to post yourself. The reported amount is deducted from **AA**, just like an open‑loop card transaction.                      | [PayOutCredit](creditCore.en.md#pay-out-credit-post)           |
| **Increase the Wallet Limit** | Raises the credit line itself, giving the user more headroom for spending.                                                                                                                      | [ChangeWalletLimit](creditCore.en.md#change-wallet-limit-post) |

---

#### Cycle Close, Grace Period, and Default Status

1. **Cycle close** (`firstCutDate` + `cutFrequency`)
    
    - The credit core calculates **Interes nominal calculado** (regular interest) and sends the corresponding webhook.
        
2. **Grace period** (`gracePeriod` days after the cut date)
    
    - During this window the customer must submit one or more **ReportPayCredit** calls whose **combined amount** satisfies the **minimum payment rule** configured in the credit product: 
        
        `minimumPayment = baseMinimumPaymentRate × AO + fixedMinimumPaymentAmount`
    
3. **Missing the minimum**
    
    - If the required minimum is not met by grace‑period end, the wallet enters **default status**.
        
    - The system generates **Interes moroso calculado** (penal interest) and, after execution, **Interes moroso ejecutado**. These amounts are added to **IO** on top of any nominal interest already owed, so the debt grows faster the longer it remains unresolved.
        
4. **Returning to good standing**
    
    - The wallet exits default status once the sum of reported payments in the current cycle **covers the minimum payment** as defined above **plus** any **executed penal interest**.
        
    - A qualifying payment triggers an **Interes cancelado** webhook and, for revolving products, **Credito Reintegrado**, replenishing **AA** accordingly.
        

Because penal (“moroso”) interest can compound on top of regular (“nominal”) interest, remaining in default quickly becomes costly. Promptly reporting at least the calculated minimum payment keeps the wallet current and prevents additional charges.

---

## Credit Operations

### Administering Existing Credit Wallets

The credit core gives you fine‑grained control over each wallet. With a small set of endpoints you can:

- **Block or unblock** a wallet temporarily
    
- **Increase** a wallet’s credit limit
    
- **Dissolve** a wallet permanently
    
- **Override interest parameters** for an individual wallet (when permitted)
    

---

#### Block vs. Dissolve

|Action|Effect|Reversible?|Interest Accrual|Card Usage|
|---|---|---|---|---|
|**Block Wallet**|Sets the wallet status to _blocked_. All card authorisations linked to the wallet are declined. Balances remain unchanged.|**Yes** – call the _Unblock_ endpoint at any time.|**Continues** under the credit‑product rules.|**Stopped**|
|**Dissolve Wallet**|Closes the wallet permanently. No new transactions or interest will be generated. Any outstanding **AO** and **IO** must still be cleared via `ReportPayCredit`.|**No** – the operation is final.|**Ceases** after dissolution.|**Stopped**|

---

#### Increasing the Credit Limit

Use **`ChangeWalletLimit`** to raise the wallet’s limit. **Limits can only move upward**; reductions are disallowed to avoid stranding unsettled authorisations.

---

#### Wallet‑Level Interest Overrides

If the associated credit product has `variableSpecs = true`, you may fine‑tune certain interest settings for one wallet without affecting any others. Submit **`ChangeInterestCreditCapital`** with the fields you wish to override; the changes activate at the next cut date and are confirmed via the **Cambio Interest ejecutado** webhook.

> **Note**  
> Overrides are ignored when `variableSpecs` is `false`, ensuring that products intended to remain uniform cannot be altered at wallet level.