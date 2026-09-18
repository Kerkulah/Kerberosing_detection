# Analyst Runbook : Kerberoasting

**Alert:** Wazuh rule `100021` — multiple RC4 encrypted TGS requests from a
single account within 5 minutes.
**MITRE:** T1558.003
**Severity:** High

## 1. Triage

- [ ] Note the alerting `Account Name` (the requester) and the `Service
      Name`(s) requested (the target SPN accounts).

- [ ] Confirm how many distinct SPNs were requested and over what time
      window, a single request is low signal; a burst across several
      SPNs is the actual indicator.

- [ ] Check the `Ticket Encryption Type` confirm it's `0x17` (RC4) and
      not a false positive from a differently decoded field.

# 2. Determine legitimacy

- [ ] Is the requesting account a known service, monitoring tool, or
      backup solution account? Check against the allowlist (Nessus and
      any other known scanners should already be excluded).

- [ ] Is the requesting account a regular human user account? If so, why
      is it enumerating SPNs, this has no normal business justification
      for most end user accounts.

- [ ] Check the source IP/hostname the ticket requests originated from,
      does it match the account's usual workstation, or is it an
      unexpected host.

# 3. Scope

- [ ] Check whether the requesting account has authenticated successfully
      elsewhere recently (correlate with 4624/4625), establish how the
      account itself may have been compromised.

- [ ] Identify every SPN account targeted, these are your at risk
      credentials regardless of whether cracking succeeds.

- [ ] Check whether any of the targeted service accounts have since
      authenticated from an unusual source (early sign the attacker
      successfully cracked a ticket and is using the credential).

# 4. Contain

- [ ] If the requesting account is confirmed compromised or unauthorized:
      disable it, force a password reset, and review its recent activity.

- [ ] Isolate the source host if it's an internal machine that shouldn't
      be doing this (not applicable if this is confirmed authorized lab
      simulation traffic).

# 5. Remediate

- [ ] Rotate the password on every targeted SPN account, using a long
      random value, this is the actual fix regardless of whether a
      crack attempt succeeded, since the ticket itself proves the hash
      was exposed.

- [ ] Where possible, migrate SPN holding service accounts to the Group
      Managed Service Accounts, which use long, automatically
      rotated passwords that are impractical to crack.

- [ ] Where gMSA isn't feasible, enforce AES. only Kerberos encryption on
      the account to remove RC4 as an option.

# 6. Close-out

- [ ] Document requesting account, targeted SPNs, time window, and
      whether the activity was confirmed malicious, authorized testing,
      or a benign false positive.
      
- [ ] If false positive, evaluate whether the source should be added to
      the allowlist to reduce future noise.
