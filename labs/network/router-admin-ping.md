# Router Admin - Ping Function Abuse & Command Injection

By: eufr4nc-SEC
Category: Network / Web Security
Target: Router Admin Panel - Ping / Diagnostics Tool

## Objective
To assess the security of the diagnostic ping function commonly exposed in router admin panels, and to determine whether user-supplied input is properly sanitized before being passed to the underlying operating system.

## Background
Most routers provide a ping tool in the admin panel at addresses like `192.168.1.1` or `router.admin`. It is intended for connectivity testing - an administrator enters an IP address or hostname, and the router returns the ping result.

The risk is that this tool often runs a system command in the background, such as:
`ping -c 4 <user_input>`

If that input is not sanitized, it becomes a direct path to command execution on the router itself.

## Methodology

**1. Reconnaissance**
Accessed the admin panel via the default gateway and identified the Diagnostics / Ping utility. Observed that the input field accepted both IPs and hostnames with no visible validation.

**2. Baseline Test**
Tested with a legitimate input:
`8.8.8.8`
Result: Normal ping output returned. This confirmed the function was live and executing system commands.

**3. Injection Test**
Tested for command injection using a separator:
`8.8.8.8; id`
`8.8.8.8 && cat /etc/passwd`

The response returned system-level output beyond the ping result, confirming that the input was being executed as part of a shell command, not treated as a single argument.

**4. Impact Assessment**
Successful injection would allow an authenticated attacker, or an attacker who obtains admin credentials, to execute arbitrary commands on the router, leading to full device compromise, traffic interception, and persistent access.

## Root Cause
The application failed to implement proper input validation and failed to separate user data from system commands. User input was concatenated directly into a shell command instead of being passed as a sanitized argument.

This is a classic Command Injection vulnerability, classified under CWE-78.
## Proof of Concept - Command Chaining

After confirming injection was possible, I tested command chaining to filter output:

Payload used:
`8.8.8.8; cat /etc/passwd | grep root`

Output returned:
`root:x:0:0:root:/root:/bin/bash`

This confirmed two things:
1. The ping function was executing chained shell commands.
2. I had read access to sensitive system files on the router.

The use of the pipe `|` allowed me to pass the output of `cat /etc/passwd` directly into `grep root`, filtering for only the privileged account. This demonstrates that an attacker is not limited to a single command - they can chain multiple commands together to enumerate the system efficiently.




## Remediation
- Enforce strict allow-list validation: only accept valid IPv4/IPv6 addresses and hostnames
- Avoid shell execution entirely. Use language-native ping libraries or APIs where possible
- If shell execution is unavoidable, use parameterized execution with no shell interpretation
- Change default admin credentials and restrict admin panel access to trusted LAN devices
- Log and rate-limit diagnostic tool usage
- Apply firmware updates from the vendor

## Key Takeaway
Diagnostic tools are often overlooked in security reviews because they appear harmless. In practice, they are one of the most privileged functions on a device.

A ping tool is not just a ping tool if it runs as root. It is a shell.

This lab reinforced a core principle: any function that touches the operating system must treat all user input as untrusted, regardless of how internal or administrative it appears to be.
