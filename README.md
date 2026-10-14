# MCPs Explained

## Facts

MCP is just a glorified HTTP server that AI connects to it using HTTP protocls. Only difference is the json it sends, it must have a specific structure, that's it.

### MCP Protocol

- All request from AI using MCP protocl have specific structure:
- Token Efficiency: It avoids cluttering the AI's context with complete API technical documentation, dynamically loading only the necessary tool descriptions and endpoints when needed ❓ HOW
- Standardization: It provides a universal framework for any AI model to interact with various tools without needing custom, ad-hoc integrations for each one.

### MCP Server
- It's a **Gateway**: Translate abstract AI model's intentions into specific and logic queries.
- The server hold the tokens and credentials to query the final services

## Comparison with other protocols

- https://en.wikipedia.org/wiki/HTTP#Request_methods 
- https://en.wikipedia.org/wiki/List_of_FTP_commands
- https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol


| Protocol | Primary Purpose | Connection Nature | Key Commands / Methods | Protocol Handler (The Software) | Content / Storage Layer (The Warehouse) |
| --- | --- | --- | --- | --- | --- |
| HTTP | Transferring and managing web resources and API data | Stateless (each request is independent) | GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS | Web server software (e.g., Nginx, Apache, IIS) parsing headers and requests | Static files (HTML/CSS), dynamic server engines (PHP/Node), or application databases (JSON APIs) |
| FTP | Managing and transferring files and directories on a server | Stateful (maintains an active session/connection) | USER, PASS, RETR (download), STOR (upload), DELE | FTP daemon software (e.g., vsftpd, FileZilla Server) handling commands | Operating system filesystem (folders and files within a restricted root/chroot directory) |
| SMTP | Delivering and routing email messages between servers | Stateful (operates via a strict step-by-step pipeline) | EHLO/HELO, MAIL FROM, RCPT TO, DATA, QUIT | Mail Transfer Agent or MTA (e.g., Postfix, Exim, Sendmail) managing the state machine | Temporary mail queue/spool (flat files on disk) and final mailbox storage (Maildir / Mbox) |
| MCP | Standardizing how AI models securely interact with external tools, files, and services | Stateful Session (local or via stream between AI client and server) | JSON-RPC methods (initialize, tools/list, tools/call, plus tool-specific arguments like email IDs) | MCP Server process (parses JSON-RPC, isolates credentials/tokens, and acts as a security sandbox/gateway) | External APIs (e.g., Gmail API), third-party services, or local filesystems abstracted into tools |


## How does the JSON-RPC looks like?

Example:

...

Let's mimic an AI and communicate manually with a MCP server to do stuff

...