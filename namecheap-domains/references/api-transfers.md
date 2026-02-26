# Namecheap Domain Transfers API Reference

Detailed parameter reference for `namecheap.domains.transfer.*` commands.

## Supported TLDs for Transfer

Only these TLDs can be transferred via API: `.biz`, `.ca`, `.cc`, `.co`, `.co.uk`, `.com`, `.com.es`, `.com.pe`, `.es`, `.in`, `.info`, `.me`, `.me.uk`, `.mobi`, `.net`, `.net.pe`, `.nom.es`, `.org`, `.org.es`, `.org.pe`, `.org.uk`, `.pe`, `.tv`, `.us`

## domains.transfer.create

Transfers a domain to Namecheap.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain to transfer |
| Years | Number | Yes | Years to add upon transfer |
| EPPCode | String | Yes | Authorization/EPP code from current registrar |
| PromotionCode | String | No | Coupon code |
| AddFreeWhoisguard | String | No | Add free domain privacy (default: `no`) |
| WGEnabled | String | No | Enable domain privacy (default: `no`) |
| IsPremiumDomain | Boolean | No | Whether domain is premium |
| PremiumPrice | Currency | No | Transfer price for premium domain |

### Response

```xml
<DomainTransferCreateResult DomainName="example.com" Transfer="true"
  ChargedAmount="8.88" TransferID="12345" StatusID="1"
  OrderID="6789" TransactionID="1011"/>
```

| Attribute | Description |
|-----------|-------------|
| DomainName | Domain being transferred |
| Transfer | `true` if transfer initiated |
| ChargedAmount | Amount charged |
| TransferID | Unique transfer ID (use for status checks) |
| StatusID | Initial status ID |
| OrderID | Order ID |
| TransactionID | Transaction ID |

### Pre-Transfer Checklist

Before initiating a transfer:
1. Domain must be unlocked at the current registrar
2. Obtain the EPP/authorization code from the current registrar
3. Domain must not be within 60 days of registration or a previous transfer
4. Domain must not be expired (some registrars block transfer of expired domains)
5. WHOIS privacy/protection should be disabled at the current registrar
6. Admin email in WHOIS must be accessible (transfer approval email goes there)

---

## domains.transfer.getStatus

Gets the status of a particular transfer.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| TransferID | Number | Yes | Transfer ID from the `create` response |

### Response

Returns `TransferGetStatusResult` with attributes: `TransferID`, `StatusID`, `Status` (text description of current transfer state).

Common statuses:
- Submitted / Pending
- Cancelled
- Approved
- Completed
- Rejected

---

## domains.transfer.updateStatus

Updates the status of a transfer. Use this to re-submit a transfer after releasing the registry lock.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| TransferID | Number | Yes | Transfer ID |
| Resubmit | Boolean | Yes | Set to `true` to resubmit the transfer |

### Response

Returns `TransferUpdateStatusResult` with `TransferID` and `IsSuccess` attributes.

---

## domains.transfer.getList

Gets the list of domain transfers.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| ListType | String | No | Filter: `ALL` (default), `INPROGRESS`, `CANCELLED`, `COMPLETED` |
| SearchTerm | String | No | Keyword to search |
| Page | Number | No | Page number (default: 1) |
| PageSize | Number | No | Results per page, max 100 (default: 20) |
| SortBy | String | No | Sort: `TransferDate`, `TransferDate_DESC`, `DomainName`, `DomainName_DESC`, `StatusID`, `StatusID_DESC` |

### Response

Returns `TransferListResult` with `Transfer` elements containing: `ID`, `DomainName`, `User`, `TransferDate`, `OrderID`, `StatusID`, `Status`.

---

## Transfer Workflow

```bash
# 1. Initiate transfer
curl -s "https://api.namecheap.com/xml.response?\
ApiUser=${NAMECHEAP_API_USER}&ApiKey=${NAMECHEAP_API_KEY}&\
UserName=${NAMECHEAP_USERNAME}&ClientIp=${NAMECHEAP_CLIENT_IP}&\
Command=namecheap.domains.transfer.create&\
DomainName=example.com&Years=1&EPPCode=YOUR_AUTH_CODE"

# Save the TransferID from the response

# 2. Check status periodically
curl -s "https://api.namecheap.com/xml.response?\
ApiUser=${NAMECHEAP_API_USER}&ApiKey=${NAMECHEAP_API_KEY}&\
UserName=${NAMECHEAP_USERNAME}&ClientIp=${NAMECHEAP_CLIENT_IP}&\
Command=namecheap.domains.transfer.getStatus&TransferID=12345"

# 3. If transfer was rejected due to lock, resubmit after unlocking
curl -s "https://api.namecheap.com/xml.response?\
ApiUser=${NAMECHEAP_API_USER}&ApiKey=${NAMECHEAP_API_KEY}&\
UserName=${NAMECHEAP_USERNAME}&ClientIp=${NAMECHEAP_CLIENT_IP}&\
Command=namecheap.domains.transfer.updateStatus&TransferID=12345&Resubmit=true"

# 4. List all transfers
curl -s "https://api.namecheap.com/xml.response?\
ApiUser=${NAMECHEAP_API_USER}&ApiKey=${NAMECHEAP_API_KEY}&\
UserName=${NAMECHEAP_USERNAME}&ClientIp=${NAMECHEAP_CLIENT_IP}&\
Command=namecheap.domains.transfer.getList&ListType=INPROGRESS"
```
