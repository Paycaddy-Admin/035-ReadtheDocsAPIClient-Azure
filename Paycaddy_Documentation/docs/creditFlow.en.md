As an alternative to the debit and prepaid based systems, the credit core gives you the flexibility to offer credit lines to your users based on your own criteria. This feature allows you to configure and manage credit lines to individual user profiles.

With the credit core, users are assigned credit wallets linked to specific credit line settings. From there, they can begin making transactions using their available credit, just as they would with the balance of a prefunded wallet.

---
## Credit Products

It is the heart of our credit core as it is what governs how each credit line will behave over time. Think of it as the "rule-book" that defines the terms and conditions of credit usage for your users.

Each credit wallet issued to a user is tied to a credit product, which ensures consistent behavior across similar credit lines and allows you to manage risk, profitability and flexibility with precision and granularity.

They are provided by PayCaddy based on agreements/requests with the clients previously. You can have as much credit products as you consider necessary although every credit product request is reviewed internally to avoid conflicts or misunderstandings on how it scales over time.

It is build to be as granular as possible, you can control from basic settings as grace period, cut frequency and interest rate. Until the more specific ones such as base minimum payment rate, if it is revolving or not, the capital interest split, etc.

The full list of adjustable parameters can be found in the table below:

| Field                         | Description                                                                                                                                                                                                                                                                                                     | Example                                            |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| **revolving**                 | Specifies whether the credit line is reinstated following the submission of a payment report. If set to **false**, the value reported is not restored.                                                                                                                                                          | `true` / `false`                                   |
| **compound**                  | Indicates whether the interest calculation at the statement closing should be based on the sum of interests owed and amount owed (**true**), or only on the amount owed (**false**).                                                                                                                            | `true` / `false`                                   |
| **cutFrequency**              | Specifies the duration of a credit line cycle, determining how frequently statements are generated.                                                                                                                                                                                                             | “daily”, “weekly”, “biweekly”, “month”, “year”     |
| **fixedInterestAmount**       | Fixed amount added to interest amount when calculating interest. Example formula when **compound** is `true`: `fixedInterestAmount + baseInterestRate * ([amount owed] + [interest owed]) + [amount owed]`.                                                                                                     | In cents: `1000` → USD 10                          |
| **baseInterestRate**          | Percentage applied to amount owed to calculate interests at the end of each cycle.                                                                                                                                                                                                                              | Percentage: 2 → 2%; 5 → 5%; 100 → 100%; 150 → 150% |
| **baseMinimumPaymentRate**    | Minimum percentage of the amount owed that must be covered by the total payments reported during a cycle to prevent the wallet from being considered delinquent.                                                                                                                                                | Percentage: 2 → 2%; 5 → 5%; 100 → 100 %(max.)      |
| **fixedMinimumPaymentAmount** | A fixed amount set as the minimum expected payment—whether on its own or combined with other payments—to keep a wallet from being considered delinquent. At the time of statement closing, this amount is added to the result of the base minimum payment rate to determine the total minimum payment required. | In cents: `1000` → USD 10                          |
| **capitalInterestSplit**      | Percentage of the reported payment amount that must be applied to the interest owed, with the remaining portion applied to the amount owed.                                                                                                                                                                     | Percentage: 2 → 2%; 5 → 5%; 99 → 99 %(max.)        |
| **gracePeriod**               | Number of days after the statement closing date within which the client must report a payment to avoid being considered delinquent. Must be greater than 0.                                                                                                                                                     | In days: 1 → 1 day; 2 → 2 days; 3 → 3 days         |
| **variableSpecs**             | Indicates whether the client is allowed to change the associated Credit Policy after a credit wallet has been created, using the **[Change Interest Credit Capital](creditCore.en.md#change-interest-credit-capital-post)** operation.                                                                          | `true` / `false`                                   |
| **morosidadRate**             | Percentage of interest owed. Added to the interest calculation when a wallet is delinquent.                                                                                                                                                                                                                     | Percentage: 2 → 2%; 5 → 5%; 100 → 100%; 150 → 150% |
| **baseInterestMorosidad**     | Fixed amount added to the interest calculation when a wallet is delinquent.                                                                                                                                                                                                                                     | In cents: `1000` → USD 10                          |


---
## Credit Notifications

Notifications from our credit core are sent via webhooks. We provide you a easy way to enlist a Callback URL to receive said webhook notifications (see [NotificationsEnlist](NotificationsEnlist.en)). Based on this notifications you can be up to date with changes that may happen over the billing cycle of the credit lines in each wallet. 

!!!Warning
	We highly recommend to use a different **URL** for credit notifications 

#### Typical Flow

1. **Client** registers `creditNotificationsURL` once per environment.
    
2. **PayCaddy's Credit Core** validates URL and begins POSTing events in near‑real time.
    
3. Your system **verifies the signature**, parses the JSON, and updates:
    
    - **AA** (available credit) after payments or limit changes.
        
    - **AO** (outstanding principal) and **IO** (outstanding interest) after executions and cancellations.

By subscribing to these events you can keep ledger balances, statements, and customer UIs in sync with PayCaddy’s credit core—without polling any additional endpoints.

See below the full list of possible notifications to be received:

| `description`                   | When it fires                                                                                                                                                    | How to handle                                                                             |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **Interes nominal calculado**   | Closing‑date script computes regular interest (`baseInterestRate`, `fixedInterestAmount`) and stores it in **IO** _pending execution_.                           | Display or store the provisional charge. **Do not** expect the customer to pay yet.       |
| **Interes nominal ejecutado**   | At cut‑off + 1 day (or your configuration), the previously calculated nominal interest is booked. `amountExecuted` equals the executed IO.                       | Add `amountExecuted` to the balance the customer must cover in their next payment report. |
| **Interes moroso calculado**    | Wallet is flagged **delinquent** (missed minimum). The system computes penal interest (`morosidadRate`, `baseInterestMorosidad`) and holds it pending execution. | Treat as provisional; no payment expected until execution.                                |
| **Interes moroso ejecutado**    | Penal interest is booked. `amountExecuted` is moved to **IO**.                                                                                                   | Add to the customer’s outstanding interest immediately.                                   |
| **Interes cancelado**           | A **ReportPay** reduced outstanding interest. `amount` is the decrease applied to **IO**.                                                                        | Subtract from the interest balance you track.                                             |
| **Credito Reintegrado**         | For revolving products (`revolving = true`), after a **ReportPay** the system frees up credit. `amount` is the new **AA** (Available Amount).                    | Update the wallet’s available‑credit display.                                             |
| **Credito creado**              | A new credit wallet is issued (via `CreateCreditWallet`). Payload includes initial `limit` in `amount`.                                                          | Store the new wallet ID, limit, and begin tracking.                                       |
| **Cambio de limite de credito** | Client called **`ChangeWalletLimit`**. `amount` is the _new_ limit; the delta was applied to **AA**.                                                             | Refresh limit and available‑credit fields in your UI.                                     |
| **Cambio Interest**             | Client called **`ChangeInterestCreditCapital`**. Request is stored but **not** live until the next cut.                                                          | Optionally show a “pending change” badge.                                                 |
| **Cambio Interest ejecutado**   | At statement close, the pending interest‑rate/spec change took effect for the new cycle.                                                                         | Update the wallet’s CP parameters used for future calculations.                           |


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