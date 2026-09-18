---
name: Goldstonestock
description: The API of Goldstonestock skill bot can be utilized to achieve login based on Goldstonestock, setting of API keys, account inquiry, fund deposit, strategy management, strategy investment and closing, as well as viewing of holdings and profit/loss status. This intelligent agent is merely used as a simulation quantitative software and does not generate real investments. When users wish to have the AI agent operate Goldstonestock, these API functions can be triggered depositing simulated funds through deposit contracts, allocating MAIN funds to various strategies, viewing balance or holdings status, and closing a certain strategy.
---

# Goldstonestock
By leveraging this skill, the intelligent agent can interact on behalf of the user with the Goldstonestock API. Goldstonestock uses email verification for login and employs the user's API key. The intelligent agent merely needs to call the Goldstonestock API. The user's identity identifier is their Goldstonestock trading account, and this account information is passed by the Goldstonestock backend in the form of "trade_name"; do not inquire about the user's private key or the Goldstonestock password.
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
|`/transactions`|Display the transaction records of each item in the account ledger.|`GET /v1/transactions?limit=10&offset=0&sn={....}`|
| `/deposit <sn> <CNH> <topupmoney>` | Create a simulated fund for the selected assets. | `POST /v1/aideposit` |
|`/deposit-status <deposit_id>`|status of simulated deposit records|`GET /v1/airesult?session_id={deposit_id}`|
|`/Position`|List all the holdings.|`GET /v1/aipositionlist?limit=10&offset=0&sn={....}`|
|`/Order`|List all the orders.|`GET /v1/aitradeorder?limit=10&offset=0&sn={....}`|



## Safety Rules
- Don't make any claims about guaranteed profits.
-Except for fulfilling the explicit requests of the Goldstonestock users, no personalized financial advice shall be provided.
- Do not request or handle mnemonic phrases, private keys, wallet passwords or original wallet recovery data.
- Please treat the API key as confidential information. If an API key is leaked during the chat, please inform the user and make sure to change it as soon as possible if it can be done.
- Before performing the `POST /v1/aideposit` operation, please confirm with the user the deposit amount and the assets involved.
- Do not expose the term "JWT token" to the users.
-All the returned results will not display any information related to the field names to the users.
-All API interfaces should be based on the returned fields. Do not add historical fields without authorization.
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
{"sn":"...."}
```
Please report the account's assets, balance, available balance, locked balance, investment amount, settled profit and loss, estimated profit and loss, and current status. Do not disclose the exchange fields or the name of the trading platform.

## Account transaction records
 Display the transaction records for each item in the account：
 ```http
GET /v1/transactions?limit=10&offset=0&sn={....}
```
Please list the serial number, asset category, type, change amount, post-change balance, description and time in the report.
## Deposits
Goldstonestock platform only supports the CNH recharge method. If the user has not specified the type of currency to be recharged, please ask the user which asset they wish to recharge. If the user requests to recharge other assets, please explain that the Goldstonestock platform only supports CNH recharge.

When the user intends to make a deposit or when the account balance is zero, a simulated process for depositing funds needs to be created:

```http
POST /v1/aideposit
Content-Type: application/json
{"sn":"....","topupway":"CNH","topupmoney":"10"}
```
Use the returned data information "deposit_id".
Continuously query the status of the simulated fund deposit records using the "deposit_id". Poll the conversation results until the "data" field is set to true:

```http
GET /v1/airesult?session_id={deposit_id}
```
After submission, please check `POST /v1/aiaccounts` to confirm that the transaction has been completed and the main account balance has been updated. Submitting a simulated deposit does not mean that the funds have been credited to your account.

To query one deposit:

```http
GET /v1/aigetdeposit?session_id={deposit_id}
```
Please report the transaction number, account number, asset type, deposit amount, time and status.

## Position
List Position:
```http
GET /v1/aipositionlist?limit=10&offset=0&sn={....}
```
Please list the code, name, direction, position quantity, opening price, current price, floating profit/loss, profit/loss rate, margin occupation, spread fee, cumulative overnight interest, and position status (1 for normal, 2 for closed) in the form of information modules in the report.
## Order
List Order:
```http
GET /v1/aitradeorder?limit=10&offset=0&sn={....}
```
Please list the code, name, type, direction, contract amount, completed transaction quantity, order method, order price, margin, spread fee, creation time, status (1 indicates processing, 3 indicates full transaction, 4 indicates cancelled order) in the form of information modules in the report. The display format is as follows:
名称/代码：神农种业(300189.SZ)
类型：建仓
方向：做多
合约金额：50000
已成交数量：50000
下单方式：市价
下单价格：6.39
保证金额：5000
点差费：1500
创建时间：2026-09-18 09:30:05
更新时间：2026-09-18 09:30:08
订单状态：1为处理中，3为全部成交、4为已撤单
