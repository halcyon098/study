---
title: "LocalPotato - When Swapping The Context Leads You To SYSTEM"
source: "https://www.localpotato.com/localpotato_html/LocalPotato.html"
author:
published:
created: 2025-08-13
description: "Here we are again with our new *potato flavor, the LocalPotato! This was a cool finding so we decided to create this dedicated website ;)"
tags:
  - "clippings"
---
LocalPotato - When Swapping The Context Leads You To SYSTEM

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

10 February 2023 - [Andrea Pierini](https://www.google.com/url?q=https://twitter.com/decoder_it&sa=D&source=editors&ust=1676048763728427&usg=AOvVaw0MZ0z96qXmnGA5QUAJ5z67) & [Antonio Cocomazzi](https://www.google.com/url?q=https://twitter.com/splinter_code&sa=D&source=editors&ust=1676048763728802&usg=AOvVaw2RwQHtBq8LPs4OTZhlc61e)  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f075e574b8b79453bce1d2df00b24b2a.png)

## Intro

Here we are again with our new \*potato flavor, the LocalPotato! This was a cool finding so we decided to create this dedicated website;)

The journey to discovering the LocalPotato began with a hint from our friend [Elad Shamir](https://www.google.com/url?q=https://twitter.com/elad_shamir&sa=D&source=editors&ust=1676048763729365&usg=AOvVaw014thg573Ri0_Ee9-7ge1l), who suggested examining the "Reserved" field in NTLM Challenge messages for potential exploitation opportunities.

After extensive research, it ended up with the “LocalPotato”, a not-so-common NTLM reflection attack in local authentication allowing for arbitrary file read/write. Combining this arbitrary file write primitive with code execution allowed us to achieve a full chain elevation of privilege from user to SYSTEM.

We reported our findings to the Microsoft Security Response Center (MSRC) on September 9, 2022, and it was resolved with the release of the January 2023 patch Tuesday and assigned the CVE number [CVE-2023-21746](https://www.google.com/url?q=https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-21746&sa=D&source=editors&ust=1676048763729734&usg=AOvVaw3Q51vsPLjpYYF2DwkZMj9Z).

  
Local NTLM Authentication

The NTLM authentication mechanism is part of the NTLMSSP (NTLM Security Support Provider), which is supported by the Windows security framework called SSPI (Security Support Provider Interface).  
SSPI provides a flexible API for handling authentication tokens and supports several underlying providers, including NTLMSSP, SPNEGO, Kerberos, etc…  
  
The NTLM authentication process involves the exchange of three types of messages (Type 1, Type 2, and Type 3) between the client and the server, processed by the NTLMSSP.  
The SSPI authentication handshake abstracts away the details of NTLM and allows for a mechanism-independent means of applying authentication, integrity, and confidentiality primitives.

[Local authentication](https://www.google.com/url?q=https://davenport.sourceforge.net/ntlm.html%23localAuthentication&sa=D&source=editors&ust=1676048763730126&usg=AOvVaw1M2aMmDv5RwKXqz49-W2Co) is a special case of NTLM authentication in which the client and server are on the same machine.  
The client acquires the credentials of the logged-in user and creates the Type 1 message, which contains the workstation and domain name of the client.  
The server examines the domain and workstation information and initiates local authentication if they match.  
The client then receives the Type 2 message from the server and checks the presence of the "Negotiate Local Call" flag to determine if the security context handle is valid.  
If it is, the default credentials are associated with the server context, and the resulting Type 3 message is empty.  
The server then verifies that the security context is bound to a user, and if so, authentication is complete.

In summary, during local authentication, the “Reserved” field which is usually set to zero for non-local authentication in the NTLM type 2 message, will reference the local server context handle that the client should associate to.

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b97dede1503cc2274835bff280475631.png)

In the above figure, we have highlighted the Reserved field containing the upper value of the context handle.

The Logic Bug

The NTLM authentication through SPPI is often misunderstood to involve direct mutual authentication between the client and server. However, in reality, the local authenticator (LSASS) is always involved, acting as the intermediary between the two.  
It is responsible for creating the messages, checking the identity permissions, and generating the proper tokens.

The objective of our research was to intercept a local NTLM authentication as a non-privileged local or domain user, and "swap" the contexts (NTLM "Reserved" field) with that of a privileged user (e.g. by coercing an authentication).

This would allow us to authenticate against a server service with these credentials, effectively exchanging the identity of our low-privileged user with a more privileged entity like SYSTEM. If successful, this would indicate that there are no checks in place to validate the Context exchanged between the two parties involved in the authentication.

The attack flow is as follows:

- Coerce the authentication of a privileged user against our server.
- Initiate an NTLM authentication of our client against a server service.
- Interception of the context "B" (Reserved bytes) of the NTLM Type 2 message coming from the server service where our unprivileged client is trying to authenticate.
- Retrieval of the context "A" (Reserved bytes) of the NTLM Type 2 message produced by our server when the privileged client tries to authenticate.
- Swap context A with B so that the privileged client will authenticate against the server service on behalf of the unprivileged client and vice versa.
- Retrieve both NTLM Type 3 empty response messages and forward them in the correct order to complete both authentication processes.
- As a result of the context swap, the Local Security Authority Subsystem (LSASS) will associate context B with the privileged identity and context A with the unprivileged identity. This results in the swap of contexts, allowing our malicious client to authenticate on behalf of the privileged user.

Below is a graphical representation of the attack flow:

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3bade8a3a6997683fb67209611c37cde.png)

To validate our assumptions about the context swap attack we did set up a custom scenario.  
In our experiment, we used [two socket servers](https://www.google.com/url?q=https://learn.microsoft.com/en-us/windows/win32/secauthn/using-sspi-with-a-windows-sockets-server?source%3Drecommendations&sa=D&source=editors&ust=1676048763731760&usg=AOvVaw3AYxCZG2PLsMBVZAPdXoUo) and [two socket clients](https://www.google.com/url?q=https://learn.microsoft.com/en-us/windows/win32/secauthn/using-sspi-with-a-windows-sockets-client&sa=D&source=editors&ust=1676048763732021&usg=AOvVaw04z_x6UR7E0C5mHgaWrqEK) to authenticate via NTLM with different users and exchange each other's "context”.  
Both parties were negotiating the NTLM authentication over a socket through SSPI.  
In particular the clients with two calls to [InitializeSecurityContext()](https://www.google.com/url?q=https://learn.microsoft.com/en-us/windows/win32/secauthn/initializesecuritycontext--ntlm&sa=D&source=editors&ust=1676048763732268&usg=AOvVaw246hQFMIAsGcYpX1V-5hGC) and the servers with two calls to [AcceptSecurityContext()](https://www.google.com/url?q=https://learn.microsoft.com/en-us/windows/win32/secauthn/acceptsecuritycontext--ntlm&sa=D&source=editors&ust=1676048763732479&usg=AOvVaw2ITyaVpka2g3nnKggmF0UX).

After some adjustments, we were successful in swapping identities and we were able to trick LSASS by associating the context with the “wrong” server.

To exploit this in a real-world scenario, we then had to find a useful trigger for coercing a privileged client and an appropriate server service.  
  
The Triggers for coercing a privileged client

Based on our previous research, we identified two key triggers for coercing a privileged client: the BITS service attempting to authenticate as the SYSTEM user via HTTP on port 5985 (WinRM), and authenticated RPC/DCOM privileged user calls.

[RogueWinRM](https://www.google.com/url?q=https://decoder.cloud/2019/12/06/we-thought-they-were-potatoes-but-they-were-beans/&sa=D&source=editors&ust=1676048763733050&usg=AOvVaw3GKWFUN2BWFQ4ZVInbhjjz) is a technique that takes advantage of the BITS service's attempt to authenticate as the SYSTEM user via HTTP on port 5985. Since this port is not enabled by default on Windows 10/11, it provides an opportunity to implement a custom HTTP server that can capture the authentication flow. This allows us to obtain SYSTEM-level authentication.

[RemotePotato0](https://www.google.com/url?q=https://www.sentinelone.com/labs/relaying-potatoes-another-unexpected-privilege-escalation-vulnerability-in-windows-rpc-protocol/&sa=D&source=editors&ust=1676048763733415&usg=AOvVaw36iJXimgvG5o2aTSJXIk7M) is a method for coercing privileged authentication on a target machine by taking advantage of standard COM marshaling. In our scenario, we discovered three interesting default CLSIDs that authenticate as SYSTEM:

1. CLSID: {90F18417-F0F1-484E-9D3C-59DCEEE5DBD8}  
	The ActiveX Installer Service "AxInstSv" is available only on Windows 10/11.
2. CLSID: {854A20FB-2D44-457D-992F-EF13785D2B51}  
	The Printer Extensions and Notifications Service "PrintNotify" is available on Windows 10/11 and Server 2016/2019/2022.
3. CLSID: {A9819296-E5B3-4E67-8226-5E72CE9E1FB7}  
	The Universal Print Management Service "McpManagementService" is available on Windows 11 and Server 2022.

By leveraging one of these triggers we could have the proper privileged identity to abuse.

Exploiting a server service

Initially, we tried to find a privileged candidate for our server service by examining the exposed RPC services, such as the Service Control Manager. However, we encountered a problem with local authentication to RPC services, as it is not possible to perform any reflection or relay attacks due to mitigations in the RPC runtime library (rpcrt4.dll).

As explained in this [blog post](https://www.google.com/url?q=https://googleprojectzero.blogspot.com/2021/10/windows-exploitation-tricks-relaying.html&sa=D&source=editors&ust=1676048763734374&usg=AOvVaw3Lh2nSUMSEb2u3ZIbke5E3) by [James Forshaw](https://www.google.com/url?q=https://twitter.com/tiraniddo&sa=D&source=editors&ust=1676048763734554&usg=AOvVaw1amS9oCjD4ceYhrT-wXmN9), Microsoft has added a mitigation in the RPC runtime to prevent authentication relay attacks from being successful.  
This is done in “ SSECURITY\_CONTEXT::ValidateUpgradeCriteria()” by checking if the authentication for an RPC connection was from the local system, and if so, setting a flag in the security context. The server will then reject the RPC call if this flag is set, before any code is called in the server. The only way to bypass this check is to either have authentication from a non-local system or have an authentication level of RPC\_C\_AUTHN\_LEVEL\_PKT\_INTEGRITY or higher, which requires knowledge of the session key for signing or encryption which of course mitigate effectively any relaying attempts.

Next, we turned our attention to the SMB server, with the goal of performing an arbitrary file write with elevated privileges.

The only requirement was that the SMB server should not require signing, which is the default for servers that are not Domain Controllers.

However, we found that the SMB protocol also has some mitigations in place to prevent cross-protocol reflection attacks.

This mitigation, also referred as [CVE-2016-3225](https://www.google.com/url?q=https://learn.microsoft.com/en-us/security-updates/securitybulletins/2016/ms16-075&sa=D&source=editors&ust=1676048763735142&usg=AOvVaw047l7X653YnMajMD4BCB7i), has been released to address the [WebDAV->SMB](https://www.google.com/url?q=https://bugs.chromium.org/p/project-zero/issues/detail?id%3D222&sa=D&source=editors&ust=1676048763735360&usg=AOvVaw0SBV2ytAPQ9bWN_IyIEkXq) relaying attack scenario.

Basically, it requires the use of the SPN "cifs/127.0.0.1" when initializing local authentication through InitializeSecurityContext() for connecting to the SMB server, even for authentication protocols other than Kerberos, such as NTLM.

The main idea behind this mitigation is to prevent relaying local authentication between two different protocols, which would result in an SPN mismatch in the authenticator and ultimately lead to an access denied error.  
According to [James Forshaw](https://www.google.com/url?q=https://twitter.com/tiraniddo&sa=D&source=editors&ust=1676048763735799&usg=AOvVaw1ffigqCh5I1sn5W-P-ftkj) a rticle " [Windows Exploitation Tricks: Relaying DCOM Authentication](https://www.google.com/url?q=https://googleprojectzero.blogspot.com/2021/10/windows-exploitation-tricks-relaying.html&sa=D&source=editors&ust=1676048763736059&usg=AOvVaw04RpUkQrNtRRNwHQ9Apr9l) ", it is possible to trick a privileged DCOM client into using an arbitrary Service Principal Name (SPN) to forge an arbitrary Kerberos ticket.  
While this applies for Kerberos, it turns out that it can also affect the SPN setting in an NTLM authentication.  
For this reason we chose to use the RPC/DCOM trigger for coercing a privileged client because we could return an arbitrary SPN in the binding strings of the Oxid resolver, thus bypassing the SMB anti-reflection mechanism.  
All we needed to do was to set an SPN of "cifs/127.0.0.1" in the originating privileged client, which was not a problem thanks to our trigger:

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a8514509cdcf3bf12ae2699afae3890f.png)

In the end, we were able to write an arbitrary file with SYSTEM privileges and arbitrary contents.  
The network capture of the SMB packets shows us successfully authenticating to the C$ share as the SYSTEM user and overwriting the file PrintConfig.dll:

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1a13e4066bc72c54d24c7aafde26ee65.png)

The POC

Creating a proof of concept for LocalPotato was a challenging task as it required writing SMB packets and sending them through the loopback interface for low-level NTLM authentication, accessing the local share, and finally writing a file.  
We relied on [Wireshark](https://www.google.com/url?q=https://www.wireshark.org/&sa=D&source=editors&ust=1676048763736930&usg=AOvVaw33Kv8GjfLOIXBR_8F_8mjP) captures and Microsoft's [MS-SMB2](https://www.google.com/url?q=https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/&sa=D&source=editors&ust=1676048763737189&usg=AOvVaw3nkNJwxjL1rUrgs2-mDSZ9) protocol specifications to complete the process. After multiple tests and code adjustments, we were finally successful.

  
To simplify the attack chain, we opted to eliminate the redirection to a non-Windows machine listening on port 135 and instead have the fake oxid resolver running on the Windows victim machine, so that the Potato trigger is local and the whole attack chain is fully local.

Just like we did in [JuicyPotatoNG](https://www.google.com/url?q=https://decoder.cloud/2022/09/21/giving-juicypotato-a-second-chance-juicypotatong/&sa=D&source=editors&ust=1676048763737563&usg=AOvVaw1Z2IMZYK9xfygBjYN-2Ew8), we leveraged the [SPPI hooks](https://www.google.com/url?q=https://learn.microsoft.com/en-us/windows/win32/api/sspi/nf-sspi-initsecurityinterfacew&sa=D&source=editors&ust=1676048763737779&usg=AOvVaw3BnSrwIDP1QT7rOz2VkYr6) to manipulate NTLM messages coming to our COM server from the privileged client, enabling the Context Swapping.

Converting an arbitrary file write into EoP is relatively straightforward.  
In our case, we utilized the McpManagementService CLSID on a Windows 2022 server, overwrote the printconfig.dll library, and instantiated the PrintNotify object.  
This forced the service to load our malicious PrintConfig.dll, granting us a SYSTEM shell:

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/46ca3fd94d767f65ba6977f7673edd5a.png)

There are various methods to weaponize an arbitrary file write into code execution as SYSTEM, such as using an [XPS Print Job](https://www.google.com/url?q=https://decoder.cloud/2019/11/13/from-arbitrary-file-overwrite-to-system/&sa=D&source=editors&ust=1676048763738319&usg=AOvVaw1FGLv3BqrjQQTiDwgz7Jae) or [NetMan DLL Hijacking](https://www.google.com/url?q=https://itm4n.github.io/windows-server-netman-dll-hijacking/&sa=D&source=editors&ust=1676048763738507&usg=AOvVaw0tvwt6sgWPlkZzCg_9NdgN). So you are free to combine the LocalPotato primitive with what you prefer;)

The LocalPotato POC is available at → [https://github.com/decoder-it/LocalPotato](https://www.google.com/url?q=https://github.com/decoder-it/LocalPotato&sa=D&source=editors&ust=1676048763738776&usg=AOvVaw3moHtb1ZZd2yakM0UT9NEz)

The Patch

The LocalPotato vulnerability was found in the NTLM authentication scheme. To locate the source of the vulnerability, we conducted a binary diff analysis of msv1\_0.dll, the security package loaded into LSASS to handle all NTLM-related operations:

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/af310507ab17d7fbc17543dffccf3c51.png)

The main focus of the analysis was the function SsprHandleChallengeMessage(), which handles NTLM challenges.

We observed the addition of a new check for the enabled feature “Feature\_MSRC74246\_Servicing\_NTLM\_ServiceBinding\_ContextSwapping” when authentication occurs:

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2200475467a1700a36adc1bc5986b5df.png)

The check introduced by Microsoft ensures that if the [ISC\_REQ\_UNVERIFIED\_TARGET\_NAME](https://www.google.com/url?q=https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831747\(v%3Dws.11\)&sa=D&source=editors&ust=1676048763739905&usg=AOvVaw15tbw_lxh654xFp5rNFi26) flag is set and an SPN is present, the SPN is set to NULL.

This change effectively addresses the vulnerability by disrupting this specific exploitation scenario.

The SMB anti-reflection mechanism checks for the presence of a specific SPN, such as "cifs/127.0.0.1", to determine whether to allow or deny access. With the patch in place, a NULL value will be found, thus denying the authentication.  
It's important to note that the ISC\_REQ\_UNVERIFIED\_TARGET\_NAME flag is passed and used by the DCOM privileged client, but prior to this patch, it was not taken into consideration for NTLM authentication.

Microsoft has released patches for supported versions of Windows, but don't worry if you have an older version. [0patch](https://www.google.com/url?q=https://blog.0patch.com/2023/02/micropatching-localpotato-ntlm.html&sa=D&source=editors&ust=1676048763740350&usg=AOvVaw1hW8m2ibqZlzLRv9seXRKr) provides fixes for LocalPotato for unsupported versions as well!  

Conclusion

In conclusion, the LocalPotato vulnerability highlights the weaknesses of the NTLM authentication scheme in local authentication.

Microsoft has resolved the issue with the release of the patch [CVE-2023-21746](https://www.google.com/url?q=https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-21746&sa=D&source=editors&ust=1676048763740757&usg=AOvVaw2hbzRBPp25HWmCmnbeTQR1), but this fix may just be a workaround as detecting forged context handles in the NTLM protocol may be difficult.

It is important to note that this type of attack is not specific to the SMB or RPC protocols, but rather a general weakness in the authentication flow.  
Other protocols that use NTLM as authentication method may still be vulnerable, provided exploitable services can be found.  

What’s next?

Well, to be honest, we ran out of ideas. But for sure, if we’ll find something new it will be the “Golden Potato”!  
  
Acknowledgments

Our thanks go to these two top security researchers:

- Elad Shamir ([@elad\_shamir](https://www.google.com/url?q=https://twitter.com/elad_shamir&sa=D&source=editors&ust=1676048763741332&usg=AOvVaw2lebkccYpY_QNr9vn7hClA)) who gave us the initial idea and with whom we constantly discussed and debated this topic
- James Forshaw ([@tiraniddo](https://www.google.com/url?q=https://twitter.com/tiraniddo&sa=D&source=editors&ust=1676048763741575&usg=AOvVaw1Q-EGE-l_Dh2uR5F5IcvPV)) who gave us useful hints when everything seemed to be lost