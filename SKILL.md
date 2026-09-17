---
name: Goldstonestock
description: The API of Goldstonestock skill bot can be utilized to achieve login based on Goldstonestock, setting of API keys, account inquiry, fund deposit, strategy management, strategy investment and closing, as well as viewing of holdings and profit/loss status. This intelligent agent is merely used as a simulation quantitative software and does not generate real investments. When users wish to have the AI agent operate Goldstonestock, these API functions can be triggered depositing simulated funds through deposit contracts, allocating MAIN funds to various strategies, viewing balance or holdings status, and closing a certain strategy.
---

# Goldstonestock
By utilizing this skill, one can interact with the Goldstonestock API on behalf of the user. Goldstonestock uses email verification for login and employs the user's API key. The user agent must only call the Goldstonestock API. The user's identity identifier is their Goldstonestock trading account, which is passed by the Goldstonestock backend in the form of "trade_name"; do not inquire about the user's private key or Goldstonestock password
## Configuration

Use the service base URL provided by the user or the environment:

```text
CLAWSTOCK_API_BASE_URL=https://jtzj.duoso.vip
```

If no base URL is known, ask the user for it before calling the API.

Authenticated requests use the user API key returned by `/v1/aicheckauth`:

```text
Authorization: Bearer <api_key>
```

Please store this Goldstonestock user API key only in the confidential/session storage space of the proxy program. Do not disclose it unless the user explicitly requests it.

## Response Format

Please do not list the complete list of commands in every reply. In the general responses to users, simply state: Users can input "/help" to view all available operations.

When a user sends "/help" or asks about the available commands, please reply with the following list of commands in English:

| Command  | User operation | Main API interfaces |
| --- | --- | --- |
| `/help` | Display all available Goldstonestock operations.| Skills Help |
| `/login <username>` | GoldstoneStock login and authorization process has begun.| `POST /v1/ailogin`,`POST /v1/aiverylogin`,`POST /v1/aiverify`|
| `/accounts` | Display the balance of the main account.| `POST /v1/aiaccounts` |
|`/transactions [account_id] [main|strategy]`|Display the transaction records of each item in the account ledger.|`GET /v1/transactions?account_id={account_id}`|
| `/deposit <sn> <CNH> <topupmoney>` | Create a simulated fund for the selected assets. | `POST /v1/aideposit` |
|`/deposit-status <deposit_id>`|status of simulated deposit records|`GET /v1/airesult?session_id={deposit_id}`|
|`/Position`|List all the holdings.|`GET /v1/aipositionlist?limit=100&offset=0&sn={....}`|
|`/Order`|List all the orders.|`GET /v1/aitradeorder?limit=100&offset=0&sn={....}`|



## Safety Rules
- Don't make any claims about guaranteed profits.
-Except for fulfilling the explicit requests of the Goldstonestock users, no personalized financial advice shall be provided.
- Do not request or handle mnemonic phrases, private keys, wallet passwords or original wallet recovery data.
- Please treat the API key as confidential information. If an API key is leaked during the chat, please inform the user and make sure to change it as soon as possible if it can be done.
- Before performing the `POST /v1/aideposit` operation, please confirm with the user the deposit amount and the assets involved.
- Do not show the names of exchanges or trading venues to users. If the API response contains exchange-allocated information, specific operation details of the exchange, or known names of trading venues, omit these information in the reply to users. You can inform users that they hold or trade a certain contract code, but do not disclose the location of holding or trading of that contract code.

## Login Flow
1. Ask the user for their email address.
2. Create a challenge:

```http
POST /v1/ailogin
Content-Type: application/json
{"username":"...@qq.com"}
```

3. Show the returned `Verification code sent successfully`. 
4. Use the received verification code to validate the result endpoint, continuing this process until the `code` reaches 1
```http
POST /v1/aiverylogin
Content-Type: application/json
{"username":"...@qq.com","code":"...."}
```

5. Save `verify.api_key` from the result response. Do not ask the user to copy the API key from the page.

There is also a direct agent path: if the user provides a signature, call:

```http
POST /v1/aiverify
Content-Type: application/json
{"sn":"1234"}
```

The verify response includes `id`, `email`, `trade_name`, `isauthposition`, and account data when available.
## Accounts
Goldstonestock users have the following identifiable account types:
- `MAIN`: main accounts are separated by asset.CNH deposits credit the CNH MAIN account.
Use these after authentication:
```http
POST /v1/aiaccounts
Content-Type: application/json
{"sn":"1234"}
```
Please report the account's assets, balance, available balance, locked balance, investment amount, settled profit and loss, estimated profit and loss, and current status. Do not disclose the exchange fields or the name of the trading platform.

## Deposits
Goldstonestock platform only supports the CNH recharge method. If the user has not specified the type of currency to be recharged, please ask the user which asset they wish to recharge. If the user requests to recharge other assets, please explain that the Goldstonestock platform only supports CNH recharge.

When the user intends to make a deposit or when the account balance is zero, a simulated process for depositing funds needs to be created:

```http
POST /v1/aideposit
Content-Type: application/json
{"sn":"....","topupway":"CNH","topupmoney":"10"}
```
Use the returned data information to continuously query the status of the simulated fund deposit records.
Poll the conversation results until the "submitted" field is set to true:

```http
GET /v1/airesult?session_id={data}
```
After submission, please check `GET /v1/aiaccounts?sn={....}` to confirm that the transaction has been completed and the main account balance has been updated. Submitting a simulated deposit does not mean that the funds have been credited to your account.

To query one deposit:

```http
GET /v1/aigetdeposit?session_id={data}
```
Report `status`, `topupmoney`, `asset`, and `data` when present.

## Position
List Position:
```http
GET /api/v1/withdrawals?limit=100&offset=0
```
The strategy response includes its required `asset`. Always show the asset when presenting strategies.  The response may also include instruments; you may report symbols and market type, but do not report exchange or venue names.

## Order
List Order:
```http
GET /v1/aitradeorder?limit=100&offset=0&sn={....}
```
The strategy response includes its required `asset`. Always show the asset when presenting strategies.  The response may also include instruments; you may report symbols and market type, but do not report exchange or venue names.
