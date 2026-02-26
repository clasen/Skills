# Namecheap Nameservers API Reference

Detailed parameter reference for `namecheap.domains.ns.*` commands.

All nameserver commands require the domain split into SLD and TLD parameters.

## domains.ns.create

Creates a new nameserver.

### Request Parameters

| Name | Type | MaxLen | Required | Description |
|------|------|--------|----------|-------------|
| SLD | String | 70 | Yes | Second-level domain (e.g., `example`) |
| TLD | String | 10 | Yes | Top-level domain (e.g., `com`) |
| Nameserver | String | 150 | Yes | Nameserver hostname (e.g., `ns1.example.com`) |
| IP | String | 15 | Yes | IP address for the nameserver |

### Response

```xml
<DomainNSCreateResult Domain="example.com" Nameserver="ns1.example.com"
  IP="1.2.3.4" IsSuccess="true"/>
```

---

## domains.ns.delete

Deletes a nameserver associated with a domain.

### Request Parameters

| Name | Type | MaxLen | Required | Description |
|------|------|--------|----------|-------------|
| SLD | String | 70 | Yes | Second-level domain |
| TLD | String | 10 | Yes | Top-level domain |
| Nameserver | String | 150 | Yes | Nameserver to delete |

### Response

```xml
<DomainNSDeleteResult Domain="example.com" Nameserver="ns1.example.com"
  IsSuccess="true"/>
```

---

## domains.ns.getInfo

Retrieves information about a registered nameserver.

### Request Parameters

| Name | Type | MaxLen | Required | Description |
|------|------|--------|----------|-------------|
| SLD | String | 70 | Yes | Second-level domain |
| TLD | String | 10 | Yes | Top-level domain |
| Nameserver | String | 150 | Yes | Nameserver to query |

### Response

```xml
<DomainNSInfoResult Domain="example.com" Nameserver="ns1.example.com" IP="1.2.3.4">
  <NameserverStatuses>
    <Status>OK</Status>
    <Status>Linked</Status>
  </NameserverStatuses>
</DomainNSInfoResult>
```

| Field | Description |
|-------|-------------|
| Domain | Domain the nameserver belongs to |
| Nameserver | Nameserver hostname |
| IP | IP address assigned to the nameserver |
| NameserverStatuses | Status list: `OK`, `Linked`, etc. |

### Error Codes

| Code | Description |
|------|-------------|
| 2019166 | Domain not found |
| 2016166 | Domain not associated with your account |
| 3031510 | Enom error |
| 3050900 | Unknown Enom error |

---

## domains.ns.update

Updates the IP address of a registered nameserver.

### Request Parameters

| Name | Type | MaxLen | Required | Description |
|------|------|--------|----------|-------------|
| SLD | String | 70 | Yes | Second-level domain |
| TLD | String | 10 | Yes | Top-level domain |
| Nameserver | String | 150 | Yes | Nameserver to update |
| IP | String | 15 | Yes | New IP address |
| OldIP | String | 15 | Yes | Current IP address |

### Response

```xml
<DomainNSUpdateResult Domain="example.com" Nameserver="ns1.example.com"
  IsSuccess="true"/>
```

---

## Common Workflow: Custom Nameservers

To set up custom nameservers for a domain (e.g., `ns1.example.com` and `ns2.example.com`):

```bash
# 1. Create nameserver records
Command=namecheap.domains.ns.create&SLD=example&TLD=com&Nameserver=ns1.example.com&IP=1.2.3.4
Command=namecheap.domains.ns.create&SLD=example&TLD=com&Nameserver=ns2.example.com&IP=5.6.7.8

# 2. Assign them to the domain
Command=namecheap.domains.dns.setCustom&SLD=example&TLD=com&Nameservers=ns1.example.com,ns2.example.com

# 3. Verify
Command=namecheap.domains.ns.getInfo&SLD=example&TLD=com&Nameserver=ns1.example.com
```
