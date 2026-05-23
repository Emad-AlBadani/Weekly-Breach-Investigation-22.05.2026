# Weekly Breach Investigation 22.05.2026

<aside>
🛡️

**Weekly Breach Investigation**

- **Breach**: **TeamPCP Mini Shai-Hulud Supply Chain Campaign**
- **Date opened**: May **18, 2026**
- **Source**: The Hacker News · Bleeping Computer · Help Net Security · Infosecurity Magazine · Aikido Security
</aside>

## 1. Executive Summary

> **What happened?**
The attack began on May 11, 2026, when a threat group called TeamPCP compromised TanStack's entire router ecosystem, spreading a worm-like payload across 170 npm packages and two PyPI packages in a single coordinated campaign.
> 

> **Who was affected?**
The compromise rippled outward a stolen developer credential led to a poisoned version of the Nx Console VS Code extension being pushed to the Visual Studio Marketplace, where it sat silently harvesting credentials from any developer who opened a workspace.
> 

> **What was the impact?**
The malicious extension was used to steal secrets and developer credentials, which were then used to move through CI/CD pipelines and exfiltrate around 3,800 of GitHub's private code repositories. GitHub, OpenAI, Mistral AI, and Grafana Labs are all confirmed victims and the extension was only live for 18 minutes.
> 

## 2. Attack Timeline

<aside>
⌛

- **May 11, 2026**  TeamPCP compromises TanStack's entire router ecosystem, spreading a worm-like payload across 170 npm packages and 2 PyPI packages in a single coordinated campaign. This is the root of everything that follows.
</aside>

<aside>
⌛

- **May 18, 2026 - 12:30 PM UTC** A trojanized version of the Nx Console VS Code extension is pushed live on the Visual Studio Marketplace. The extension looks and behaves like the real Nx Console, but on startup it silently runs a single shell command that downloads and executes a hidden package from a planted commit on the official nrwl/nx GitHub repository. The credential stealer begins harvesting immediately 1Password vaults, GitHub tokens, AWS credentials, npm tokens, and Claude Code configurations.
</aside>

<aside>
⌛

- **May 18, 2026 - 12:48 PM UTC**  The community, including Aikido Intel, catches it quickly. The malicious version is pulled within 18 minutes on the VS Code Marketplace and 36 minutes on Open VSX. But the damage is done.
</aside>

<aside>
⌛

- **May 19, 2026** GitHub discloses it is investigating unauthorized access to internal repositories. TeamPCP claims to have extracted data from roughly 4,000 private repos.
</aside>

<aside>
⌛

- **May 21, 2026** GitHub CISO Alexis Wales officially confirms the breach was caused by the malicious Nx Console extension and promises a fuller report once the investigation is complete. The Nx team officially acknowledges that one of its developers was compromised by the TanStack supply chain attack, causing their GitHub credentials to be leaked this is what allowed the attacker to run workflows on the Nx repository as a contributor.
</aside>

## 3. MITRE ATT&CK Mapping

| **Tactic** | **Technique** | **ID** |
| --- | --- | --- |
| Initial Access | Supply Chain Compromise poisoned npm packages (TanStack) leading to a trojanized VS Code extension (Nx Console 18.95.0) | T1195.001 |
| Execution | On startup the extension silently runs a shell command that downloads and executes a hidden package from a planted commit on the official nrwl/nx GitHub repository Command and Scripting Interpreter. | T1059 |
| Impact | Credential Access + Exfiltration payload harvested cloud, CI/CD, and AI coding assistant credentials and established persistent access on macOS systems | T1552 / T1567 |

## 4. Detection Opportunities

<aside>
💡

- **Log source:** VS Code extension audit logs, endpoint EDR telemetry (unexpected shell command spawned by VS Code process), CI/CD pipeline access logs, npm and GitHub token usage logs, outbound network traffic from developer workstations.
</aside>

<aside>
💡

- **Detection rule:** Alert on VS Code spawning unexpected child processes or shell commands at workspace startup. Flag any outbound network connection from a VS Code extension process to an unknown external host. Monitor GitHub tokens being used from IP addresses that don't match the developer's normal location. Alert on new CI/CD pipeline runs triggered by contributor accounts that haven't been active recently. Flag bulk repository cloning or export activity from internal GitHub accounts.
</aside>

<aside>
💡

- **IOCs:** CVE-2026-48027 assigned to this vulnerability Nx Console version 18.95.0 is the malicious build. GitHub removed it from the Visual Studio Marketplace, isolated the compromised endpoint, and rotated critical internal secrets. Additional indicators: unexpected commits to the official nrwl/nx repository from contributor accounts, TeamPCP infrastructure domains, stolen token usage from anomalous IPs.
</aside>

## 5. Recommended Mitigations

> Audit every VS Code extension installed across developer workstations right now especially any that were updated in the May 11–18 window. If a developer installed Nx Console 18.95.0, treat their credentials as fully compromised and rotate everything: GitHub tokens, AWS keys, npm tokens, 1Password vault access, and any CI/CD secrets they had access to.
> 

> The upload of the malicious Nx Console version was performed "without manual approval" from other Nx administrators. This is the architectural failure that enabled the attack. Any publisher pipeline that allows a single compromised contributor to push a new release without multi-party approval is a risk. Enforce required reviewers on every release workflow.
> 

> Trust, not sophistication, is what makes attacks like Nx Console work. These are not sketchy packages from unknown publishers they have the install count, the verified publisher badge, and the marketplace legitimacy that signal safety. That signal is now the target. Train developers to treat even trusted, verified extensions as potential attack vectors and implement endpoint monitoring that watches VS Code process behavior, not just the extensions themselves.
> 

## 6. Analyst Notes

> What I learned
> 
> 
> <aside>
> 📋
> 
> - The attacker never needed to touch a single firewall or brute-force a single password. The attack never needed to breach a perimeter. It entered through packages and extensions that developers routinely install, then harvested the credentials those developers use to access everything else.
> </aside>
> 

> What surprised me
> 
> 
> <aside>
> 📋
> 
> - What is consistent across all of TeamPCP's attack waves is that technical sophistication is not the primary weapon trust is. A verified publisher badge, a high install count, and distribution through an official marketplace mean that developers install the extension without hesitation and that no one thinks to check. They only needed 18 minutes on the marketplace to hit GitHub, OpenAI, Mistral AI, and Grafana Labs simultaneously.
> </aside>
> 

> What I’d investigate further with internal access
> 
> 
> <aside>
> 📋
> 
> - With internal access, the first thing I'd want to see is the full list of developer machines that had Nx Console 18.95.0 installed even for a few seconds and cross-reference that against every privileged credential those developers held. Then I'd map every CI/CD pipeline those credentials could reach. The breach at GitHub was 3,800 repositories. The question is what was in those repositories and whether any of it enables a second wave attack downstream.
> </aside>
> 

```jsx
@Emad Al-Baadani
```