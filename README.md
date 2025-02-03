# SMTP-subaddressing
Sieve Email Filtering: Subaddress Extension (RFC 5233)

### Task description:

To configure Postfix to relay emails using subaddressing (e.g., `user+detail@example.com`) while stripping the `+detail` part for internal delivery but preserving it for external delivery, you can use the following approach:

### 1. **Configure Postfix to Strip Subaddressing for Internal Delivery**

You can use the `canonical` or `recipient_canonical` maps to rewrite the recipient address for internal delivery. Here's how:

#### a. **Create a Canonical Map**

Create a file `/etc/postfix/canonical` with the following content:

```
/^([^+]+)\+.*@example\.com$/\1@example.com/
```

This regex captures the part before the `+` and rewrites the address to remove the subaddressing part.

#### b. **Compile the Canonical Map**

Run the following command to compile the map:

```bash
postmap /etc/postfix/canonical
```

#### c. **Configure Postfix to Use the Canonical Map**

Edit your `/etc/postfix/main.cf` file and add the following lines:

```bash
recipient_canonical_maps = regexp:/etc/postfix/canonical
```

This tells Postfix to use the `canonical` map to rewrite recipient addresses.

### 2. **Preserve Subaddressing for External Delivery**

Postfix will not modify the sender address when relaying emails to external domains, so subaddressing will be preserved for outgoing emails by default.

### 3. **Relay Configuration**

Ensure that your Postfix is configured to relay emails correctly. This typically involves setting up `relayhost` or `transport_maps` if you're using a specific SMTP server for relaying.

For example, if you're using a relay host, you might have something like this in your `main.cf`:

```bash
relayhost = [smtp.example.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_security_level = encrypt
```

### 4. **Restart Postfix**

After making these changes, restart Postfix to apply the new configuration:

```bash
systemctl restart postfix
```

### 5. **Testing**

Send a test email to an address with subaddressing (e.g., `user+detail@example.com`) and verify that:

- The email is delivered internally to `user@example.com` (without the `+detail` part).
- The email is sent to external domains with the original subaddressing intact.

### Example Configuration Summary

- **`/etc/postfix/canonical`**:
  ```
  /^([^+]+)\+.*@example\.com$/\1@example.com/
  ```

- **`/etc/postfix/main.cf`**:
  ```bash
  recipient_canonical_maps = regexp:/etc/postfix/canonical
  relayhost = [smtp.example.com]:587
  smtp_sasl_auth_enable = yes
  smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
  smtp_tls_security_level = encrypt
  ```

This setup ensures that subaddressing is stripped for internal delivery but preserved for external delivery.
