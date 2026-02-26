# Namecheap Domains API Reference

Detailed parameter reference for `namecheap.domains.*` commands.

## domains.getList

Returns a list of domains for the account.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| ListType | String | No | `ALL` (default), `EXPIRING`, or `EXPIRED` |
| SearchTerm | String | No | Keyword to search domain names |
| Page | Number | No | Page number (default: 1) |
| PageSize | Number | No | Results per page, max 100 (default: 20) |
| SortBy | String | No | Sort field: `Name`, `Name_DESC`, `ExpireDate`, `ExpireDate_DESC`, `CreateDate`, `CreateDate_DESC` |

### Response

Returns `Domain` elements with attributes: `ID`, `Name`, `User`, `Created`, `Expires`, `IsExpired`, `IsLocked`, `AutoRenew`, `WhoisGuard`, `IsPremium`, `IsOurDNS`.

---

## domains.getContacts

Gets contact information for a domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain name to query |

### Response

Returns `Registrant`, `Tech`, `Admin`, `AuxBilling` contact blocks with fields: `OrganizationName`, `JobTitle`, `FirstName`, `LastName`, `Address1`, `Address2`, `City`, `StateProvince`, `StateProvinceChoice`, `PostalCode`, `Country`, `Phone`, `PhoneExt`, `Fax`, `EmailAddress`.

---

## domains.create

Registers a new domain. Use HTTP POST for this command.

For IDN domains, send the domain in Punycode format and include the `IdnCode` parameter.

### Request Parameters

| Name | Type | MaxLen | Required | Description |
|------|------|--------|----------|-------------|
| DomainName | String | 70 | Yes | Domain name to register |
| Years | Number | 2 | Yes | Years to register (default: 2) |
| PromotionCode | String | 20 | No | Coupon code |
| Nameservers | String | — | No | Comma-separated custom nameservers |
| AddFreeWhoisguard | String | 10 | No | Add free domain privacy (default: `no`) |
| WGEnabled | String | 10 | No | Enable domain privacy (default: `no`) |
| IsPremiumDomain | Boolean | 10 | No | Whether domain is premium |
| PremiumPrice | Currency | 20 | No | Registration price for premium domain |
| EapFee | Currency | 20 | No | Early Access Program fee |
| IdnCode | String | 100 | No | Internationalized Domain Name code |

**Contact parameters** — required for four contact types: `Registrant`, `Tech`, `Admin`, `AuxBilling`. Replace `{Type}` with each prefix:

| Name | Type | MaxLen | Required | Description |
|------|------|--------|----------|-------------|
| {Type}OrganizationName | String | 255 | No | Organization |
| {Type}JobTitle | String | 255 | No | Job title |
| {Type}FirstName | String | 255 | Yes | First name |
| {Type}LastName | String | 255 | Yes | Last name |
| {Type}Address1 | String | 255 | Yes | Address line 1 |
| {Type}Address2 | String | 255 | No | Address line 2 |
| {Type}City | String | 50 | Yes | City |
| {Type}StateProvince | String | 50 | Yes | State/Province |
| {Type}StateProvinceChoice | String | 50 | No | State/Province choice |
| {Type}PostalCode | String | 50 | Yes | Postal code |
| {Type}Country | String | 50 | Yes | Country code |
| {Type}Phone | String | 50 | Yes | Phone: `+NNN.NNNNNNNNNN` |
| {Type}PhoneExt | String | 50 | No | Phone extension |
| {Type}Fax | String | 50 | No | Fax: `+NNN.NNNNNNNNNN` |
| {Type}EmailAddress | String | 255 | Yes | Email address |

**Billing contact** (all optional):

| Name | Type | MaxLen | Description |
|------|------|--------|-------------|
| BillingFirstName | String | 255 | First name |
| BillingLastName | String | 255 | Last name |
| BillingAddress1 | String | 255 | Address line 1 |
| BillingAddress2 | String | 255 | Address line 2 |
| BillingCity | String | 50 | City |
| BillingStateProvince | String | 50 | State/Province |
| BillingStateProvinceChoice | String | 50 | State/Province choice |
| BillingPostalCode | String | 50 | Postal code |
| BillingCountry | String | 50 | Country code |
| BillingPhone | String | 50 | Phone: `+NNN.NNNNNNNNNN` |
| BillingPhoneExt | String | 50 | Phone extension |
| BillingFax | String | 50 | Fax: `+NNN.NNNNNNNNNN` |
| BillingEmailAddress | String | 255 | Email |
| BillingTaxNumber | String | 25 | Tax number |

**Extended attributes** — required for certain TLDs: .us, .eu, .ca, .co.uk, .org.uk, .me.uk, .nu, .com.au, .net.au, .org.au, .es, .nom.es, .com.es, .org.es, .de, .fr.

### Response

```xml
<DomainCreateResult Domain="example.com" Registered="true"
  ChargedAmount="10.87" DomainID="12345" OrderID="6789"
  TransactionID="1011" WhoisguardEnable="true"
  NonRealTimeDomain="false"/>
```

| Attribute | Description |
|-----------|-------------|
| Domain | Domain name |
| Registered | `true` or `false` |
| ChargedAmount | Total charged |
| DomainID | Unique domain ID |
| OrderID | Unique order ID |
| TransactionID | Unique transaction ID |
| WhoisguardEnable | Whether privacy is enabled |
| NonRealTimeDomain | Whether registration is non-instant |

### Error Codes

| Code | Description |
|------|-------------|
| 2033409 | Auth error — order not found for username |
| 2033407, 2033270 | Cannot enable privacy when AddWhoisguard=NO |
| 2015182 | Invalid phone format (use `+NNN.NNNNNNNNNN`) |
| 2015267 | EUAgreeDelete must not be NO |
| 2011170 | Invalid PromotionCode |
| 2011280 | Invalid TLD |
| 2015167 | Invalid Years |
| 2030280 | TLD not supported in API |
| 2011168 | Invalid Nameservers |
| 2011322 | Invalid Extended Attributes |
| 2010323 | Missing required billing contact field |
| 2528166 | Order creation failed |
| 3019166, 4019166 | Domain not available |
| 3031166 | Provider error |
| 3028166 | Enom error |
| 3031900 | Unknown provider response |
| 4023271 | Error adding free PositiveSSL |
| 4023166 | Error adding domain |
| 5050900 | Unknown error adding domain |
| 4026312 | Refund error |
| 5026900 | Unknown refund error |
| 2515610 | Price mismatch |
| 2515623 | Premium/regular domain mismatch |
| 2005 | Invalid country name |

### IDN Language Codes

`afr`, `alb`, `ara`, `arg`, `arm`, `asm`, `ast`, `ave`, `awa`, `aze`, `bak`, `bal`, `ban`, `baq`, `bas`, `bel`, `ben`, `bho`, `bos`, `bul`, `bur`, `car`, `cat`, `che`, `chi`, `chv`, `cop`, `cos`, `cze`, `dan`, `div`, `doi`, `dut`, `eng`, `est`, `fao`, `fij`, `fin`, `fre`, `fry`, `geo`, `ger`, `gla`, `gle`, `gon`, `gre`, `guj`, `heb`, `hin`, `hun`, `inc`, `ind`, `inh`, `isl`, `ita`, `jav`, `jpn`, `kas`, `kaz`, `khm`, `kir`, `kor`, `kur`, `lao`, `lav`, `lit`, `ltz`, `mal`, `mkd`, `mlt`, `mol`, `mon`, `mri`, `msa`, `nep`, `nor`, `ori`, `oss`, `pan`, `per`, `pol`, `por`, `pus`, `raj`, `rum`, `rus`, `san`, `scr`, `sin`, `slo`, `slv`, `smo`, `snd`, `som`, `spa`, `srd`, `srp`, `swa`, `swe`, `syr`, `tam`, `tel`, `tgk`, `tha`, `tib`, `tur`, `ukr`, `urd`, `uzb`, `vie`, `wel`, `yid`

---

## domains.getTldList

Returns a list of available TLDs.

### Request Parameters

No additional parameters beyond global ones.

### Response

Returns `Tld` elements with attributes for each supported TLD including pricing and availability info.

---

## domains.setContacts

Sets contact information for a domain.

### Request Parameters

| Name | Type | MaxLen | Required | Description |
|------|------|--------|----------|-------------|
| DomainName | String | 70 | Yes | Domain name |

Plus the same contact parameters as `domains.create` for `Registrant`, `Tech`, `Admin`, and `AuxBilling` types.

---

## domains.check

Checks availability of one or more domains.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainList | String | Yes | Comma-separated list of domains to check |

### Response

```xml
<DomainCheckResult Domain="example.com" Available="true" ErrorNo="0"
  Description="" IsPremiumName="false" PremiumRegistrationPrice="0"
  PremiumRenewalPrice="0" PremiumRestorePrice="0"
  PremiumTransferPrice="0" IcannFee="0" EapFee="0.0"/>
```

---

## domains.reactivate

Reactivates an expired domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain to reactivate |
| PromotionCode | String | No | Coupon code |
| YearsToAdd | Number | No | Years to add |
| IsPremiumDomain | Boolean | No | Whether domain is premium |
| PremiumPrice | Currency | No | Reactivation price for premium domain |

---

## domains.renew

Renews an expiring domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain to renew |
| Years | Number | Yes | Years to renew (default: 2) |
| PromotionCode | String | No | Coupon code |
| IsPremiumDomain | Boolean | No | Whether domain is premium |
| PremiumPrice | Currency | No | Renewal price for premium domain |

### Response

Returns `DomainRenewResult` with attributes: `DomainName`, `DomainID`, `Renew` (true/false), `OrderID`, `TransactionID`, `ChargedAmount`.

---

## domains.getRegistrarLock

Gets the registrar lock status of a domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain to check |

### Response

Returns `DomainGetRegistrarLockResult` with `RegistrarLockStatus` attribute (`true`/`false`).

---

## domains.setRegistrarLock

Sets the registrar lock for a domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain name |
| LockAction | String | No | `LOCK` or `UNLOCK` |

---

## domains.getInfo

Returns detailed information about a domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain to query |

### Response

Returns `DomainGetInfoResult` with attributes: `Status`, `ID`, `DomainName`, `OwnerName`, `IsOwner`. Contains nested elements for `DomainDetails` (creation date, expiration date), `Whoisguard`, `DnsDetails`, `Modificationrights`.
