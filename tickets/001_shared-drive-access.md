# Ticket 001: User Can't Access a Shared Drive

## Symptom

User reports: "I can't open `\\\\SERVER\\Share` — getting Access Denied or path not found."

## Fast Triage Questions

Before touching anything, ask or check:

1. Is this affecting just one user, or everyone?
2. Are they on the same network or connected via VPN?
3. Can they reach the server at all?
4. Is this an authentication (permissions) problem or a network (connectivity) problem?

## Commands + Output Analysis

### 1\. Check network configuration

```
ipconfig /all
```

!\[ipconfig output](../../assets/ticket-001/01\_ipconfig.png)

**What to look for:** Confirm the machine has a valid IP, a default gateway, and DNS servers assigned. If any are missing or show 169.254.x.x (APIPA), the machine isn't getting a proper network config.

### 2\. Ping the server by name

```
ping ThomasDell-XPS15
```

!\[ping by name](../../assets/ticket-001/02\_ping\_name.png)

**What to look for:** Successful replies mean both name resolution and network connectivity are working. If this fails but ping by IP works, it's a DNS issue.

### 3\. Ping by IP address

```
ping 127.0.0.1
```

!\[ping by IP](../../assets/ticket-001/03\_ping\_ip.png)

**What to look for:** If this works but ping by name fails, the problem is name resolution (DNS), not the network itself.

### 4\. Test DNS resolution

```
nslookup ThomasDell-XPS15
```

!\[nslookup](../../assets/ticket-001/04\_nslookup.png)

**What to look for:** If nslookup returns the correct IP, DNS is fine. If it fails, try `ipconfig /flushdns` to clear stale cache and retest.

### 5\. Check existing drive mappings

```
net use
```

!\[net use](../../assets/ticket-001/05\_net\_use.png)

**What to look for:** Stale or disconnected sessions to the same server. These can block new connections, especially if they used different credentials.

### 6\. Attempt to connect to the share

```
net use \\\\ThomasDell-XPS15\\TestShare
```

!\[net use share](../../assets/ticket-001/06\_net\_use\_share.png)

**What to look for:** "Command completed successfully" means access works. "Access denied" means permissions. "Network path not found" means connectivity or name issue.

### 7\. Verify access

```
dir \\\\ThomasDell-XPS15\\TestShare
```

!\[dir verification](../../assets/ticket-001/07\_dir\_verify.png)

**What to look for:** You should see the files in the share listed. This confirms end-to-end access.

## Decision Tree

* **Ping by IP works, ping by name fails** → DNS/name resolution issue. Run `ipconfig /flushdns` and retest with `nslookup`.
* **Ping fails entirely** → Network issue. Check cable, Wi-Fi, VPN (Tailscale), or firewall.
* **Connection works but "Access Denied"** → Permissions issue. Check share permissions vs NTFS permissions, or user may have stale cached credentials.
* **Prompted for credentials repeatedly** → Cached credential conflict. Clear with `net use \\\\SERVER\\Share /delete` and reconnect.

## Fix Steps

Depending on the root cause:

**If stale session:**

```
net use \\\\SERVER\\Share /delete
net use \\\\SERVER\\Share /user:DOMAIN\\username
```

**If DNS issue:**

```
ipconfig /flushdns
nslookup SERVERNAME
```

**If VPN/Tailscale not connected:**

* Verify Tailscale is running and connected
* Confirm the server's Tailscale IP is reachable

## Verification

After applying the fix:

1. Run `dir \\\\SERVER\\Share` and confirm files are listed.
2. Have the user open the path in File Explorer.
3. Document what the root cause was and what fixed it.

