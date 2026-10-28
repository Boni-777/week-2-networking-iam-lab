# AWS Student Builders Group — Week 2 Lab
## Networking & IAM Fundamentals

**Name:** Bontu Bekele  
**Date completed:** October 28, 2026 
**Track:** Cloud & DevOps

> **Before submission:** The conceptual answers and subnet calculations below are completed. Run the three commands listed in the evidence section on your own Windows computer and replace the clearly marked result fields with what your computer actually shows. Do not invent results or publish your exact local IP address, MAC address, Wi-Fi name, passwords, AWS keys, tokens, or other secrets.

## Task 1 — Identify Your Network Information

**Operating system:** Microsoft Windows

Run `ipconfig` in Command Prompt or PowerShell and inspect the active Wi-Fi or Ethernet adapter. Do not paste the full output into this public repository.

- **Local IPv4 address:** Checked locally; not published here for privacy.
- **Default gateway/router:** Checked locally; not published here for privacy.
- **IPv4 classification:** [Replace with “private” or “public” based on the address you observed.]

Private IPv4 ranges are:
- `10.0.0.0` through `10.255.255.255`
- `172.16.0.0` through `172.31.255.255`
- `192.168.0.0` through `192.168.255.255`

An address in one of these ranges is private. `ipconfig` normally shows your computer's local address, not necessarily the public IP address used by your internet connection.

## Task 2 — Practice Subnetting

**Starting network:** `10.0.0.0/24`  
**Required number of subnets:** 4 equal-sized subnets

### Calculations

1. IPv4 addresses contain 32 bits.
2. The starting prefix is `/24`, so the number of host bits is `32 - 24 = 8`.
3. The starting network contains `2^8 = 256` total addresses.
4. To create four equal-sized subnets, borrow two bits because `2^2 = 4`.
5. The new prefix is `/24 + 2 = /26`.
6. Each `/26` subnet has `32 - 26 = 6` host bits and therefore `2^6 = 64` total addresses.
7. The subnet block size is 64, so the network addresses increment by 64 in the final octet: `.0`, `.64`, `.128`, and `.192`.
8. Under conventional IPv4 subnetting, the first address is the network address and the final address is the broadcast address, leaving `64 - 2 = 62` conventional usable host addresses.
9. Under standard AWS subnet rules, AWS reserves five IPv4 addresses in each subnet, leaving `64 - 5 = 59` AWS-usable IPv4 addresses.

| Subnet | CIDR | Network address | Full address range | Conventional usable host range | Broadcast address | AWS usable IPv4 addresses |
|---|---|---|---|---|---|---:|
| 1 | `10.0.0.0/26` | `10.0.0.0` | `10.0.0.0–10.0.0.63` | `10.0.0.1–10.0.0.62` | `10.0.0.63` | 59 (`10.0.0.4–10.0.0.62`) |
| 2 | `10.0.0.64/26` | `10.0.0.64` | `10.0.0.64–10.0.0.127` | `10.0.0.65–10.0.0.126` | `10.0.0.127` | 59 (`10.0.0.68–10.0.0.126`) |
| 3 | `10.0.0.128/26` | `10.0.0.128` | `10.0.0.128–10.0.0.191` | `10.0.0.129–10.0.0.190` | `10.0.0.191` | 59 (`10.0.0.132–10.0.0.190`) |
| 4 | `10.0.0.192/26` | `10.0.0.192` | `10.0.0.192–10.0.0.255` | `10.0.0.193–10.0.0.254` | `10.0.0.255` | 59 (`10.0.0.196–10.0.0.254`) |

**AWS reserved-address example:** In `10.0.0.0/26`, AWS reserves `10.0.0.0`, `10.0.0.1`, `10.0.0.2`, `10.0.0.3`, and `10.0.0.63`. Therefore, the AWS-usable range is `10.0.0.4–10.0.0.62`.

## Task 3 — Test DNS Resolution

**Command:** `nslookup example.com`

- **Domain tested:** `example.com`
- **Tool used:** `nslookup`
- **Did DNS resolution succeed?:** [Replace with the result from your computer.]
- **IP address(es) returned:** [Copy the address(es) shown by your command.]

**What DNS does:** The Domain Name System (DNS) looks up domain names and returns associated information, such as IP addresses, so a computer can find the relevant server.

## Task 4 — Inspect an HTTPS Website

Open `https://example.com` in a browser, then run:

```powershell
curl.exe -I https://example.com
```

Use `curl.exe` in Windows PowerShell to run the curl executable directly.

- **Did the website load in the browser?:** [Replace with Yes or No.]
- **HTTP status line returned:** [Copy the actual status line shown by your command.]
- **What HTTPS adds to HTTP:** HTTPS uses TLS to encrypt data in transit, help verify the server's identity using its certificate, and protect the connection against tampering.
- **Why DNS success does not guarantee a website will load:** DNS only helps resolve a domain name to an address. The server might be unreachable, the connection could time out, TLS validation could fail, or the web application could return an error.

Record a redirect or any other status exactly as returned. Do not assume the status is `200 OK`.

## Task 5 — Interpret HTTP Status Codes

| Status code | Meaning | Example situation |
|---|---|---|
| `200` | OK — the request succeeded. | A page or resource is returned successfully. |
| `301` | Moved Permanently — the resource has a permanent new URL. | A site redirects a resource to a new URL. |
| `403` | Forbidden — the server refuses access to the requested resource. | The requester does not have permission to access a resource. |
| `404` | Not Found — the requested resource was not found. | The URL points to a page or resource that does not exist. |
| `500` | Internal Server Error — the server encountered an unexpected problem. | An application error prevents the server from completing the request. |

### Difference between DNS failure, timeout, TLS error, and HTTP error

- **DNS failure:** The domain name could not be resolved to an address.
- **Connection timeout:** The destination did not respond within the allowed time. Investigate connectivity, routing, firewall/security rules, and server availability.
- **TLS certificate error:** The secure connection's certificate could not be trusted or validated, or another TLS check failed.
- **HTTP error response:** The client reached an HTTP server and received a response such as `403`, `404`, or `500`. The status code helps narrow down the issue.

## Task 6 — Write a Least-Privilege IAM Policy

The completed policy is in [`s3-read-policy.json`](s3-read-policy.json).

1. **Which field specifies the permitted action?** `Action`.
2. **Which field specifies the resources?** `Resource`.
3. **Does `s3:GetObject` allow listing all objects in a bucket?** No. Listing objects typically requires `s3:ListBucket`, which applies to the bucket resource itself.
4. **Why is a narrow permission safer than granting `s3:*`?** It follows least privilege by allowing only the required action and resource, reducing the potential impact of a mistake or compromised identity.
5. **Why should AWS access keys never be committed to GitHub?** Someone could copy and misuse the credentials to access resources or cause unexpected charges. Keep credentials out of repositories and revoke/rotate any exposed keys promptly.

**Safety:** This lab requires writing and reviewing the JSON policy only. Do not attach it to a live AWS identity unless you have explicit authorization.

## Task 7 — Troubleshooting Practice

1. **A domain name does not resolve to an IP address.** Investigate **DNS** first because the name-to-address lookup is failing. Check the spelling, DNS resolver, and DNS response.
2. **A domain resolves, but the browser connection times out.** Investigate **network connectivity** first: routing, firewall/security rules, port reachability, and whether the server is responding.
3. **The browser displays a TLS certificate warning.** Investigate **TLS**: certificate validity, hostname match, expiration, certificate chain, and device date/time.
4. **A website responds with HTTP `404`.** Investigate the **HTTP/application** layer: check the URL/path and whether the resource exists on the server.
5. **An EC2 application can reach Amazon S3 but receives `AccessDenied` when reading an object.** Investigate **IAM/S3 authorization**: the identity used by the application, its effective permissions, the object ARN, bucket policies or explicit denies, and relevant encryption-key permissions if applicable.

## Evidence

The lab allows screenshots to be optional if the README evidence is sufficient. If adding screenshots, place them in `evidence/` and make sure they do not expose private network details or credentials.

Suggested files:
- `evidence/dns-lookup.png` — screenshot of your `nslookup example.com` output.
- `evidence/https-response.png` — screenshot of your `curl.exe -I https://example.com` output.

Screenshots are not included in this package because they should show results from your own computer. Add them if you want to provide visual evidence.

## Final Checklist

- [ ] Enter your name and completion date.
- [ ] Confirm the local IPv4 classification from `ipconfig`; do not publish sensitive network details.
- [ ] Run `nslookup example.com` and record the actual result.
- [ ] Open `https://example.com`, run `curl.exe -I https://example.com`, and record the actual result.
- [ ] Review the subnet calculations and all task answers.
- [ ] Confirm `s3-read-policy.json` is valid JSON.
- [ ] Check that no passwords, AWS access keys, tokens, MAC addresses, or unnecessary private network details are included.
- [ ] Upload the files to GitHub.
- [ ] Submit the repository URL in the Week 2 section of the Cloud & DevOps track sheet.

## Windows Commands

Run these one at a time in Command Prompt or PowerShell:

```powershell
ipconfig
nslookup example.com
curl.exe -I https://example.com
```
