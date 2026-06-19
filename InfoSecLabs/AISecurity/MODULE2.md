# AI for Security

## Log Mining with LLMs

Because humans read slowly and get tired, hackers use "Low and Slow" attacks to blend in. By feeding these massive, messy logs into a Large Language Model (LLM), the AI can read millions of lines in seconds and highlight the one line that "doesn't look right," explaining it in plain English.

Traditional Security Information and Event Management (SIEM) tools rely on Static Correlation Rules.

Example SIEM Rule: "Alert if a user fails to login 5 times in 1 minute."

The problem? What if the attacker fails 4 times, waits 5 hours, logs in successfully from Russia, and then opens PowerShell? The static rule won't trigger.

LLM Anomaly Detection: An LLM doesn't look for static numbers. It looks for Semantic Deviations. You can feed an LLM the last 100 commands a user typed and ask, "Does this sequence of commands look normal for an HR employee?" The LLM understands that an HR employee running `whoami /priv` or `vssadmin delete shadows` is highly malicious, regardless of how slowly it was typed.

Integrating an LLM directly into a log pipeline requires specific prompting architectures to ensure deterministic outputs. Since models can hallucinate, SOC engineers use Few-Shot Prompting and Guided Output Parsing.

- JSON Extraction Prompt Architecture: To parse messy Windows Event Logs (Event ID 4688 - Process Creation) into usable Threat Intel, we use systemic framing.

```
// System Prompt
"You are a Senior DFIR Analyst. Parse the following raw Windows Event Log.
Extract ONLY the executable name, the parent executable, and a 1-sentence assessment of malicious intent. 
Output STRICTLY in JSON format matching this schema:
{ 'exe': string, 'parent': string, 'assessment': string, 'is_malicious': boolean }
Do not output markdown or conversational text."

// User Prompt
"Log: A new process has been created. Creator Subject: ... Process Information: New Process Name: C:WindowsTempsvch0st.exe. Parent Process Name: C:WindowsSystem32cmd.exe"
```

LLM Response:

```
{
  "exe": "svch0st.exe",
  "parent": "cmd.exe",
  "assessment": "Execution of a misspelled critical system process (svch0st vs svchost) from a temporary directory spawned by cmd.exe indicates defense evasion.",
  "is_malicious": true
}
```

This structured JSON can now be automatically forwarded back into the SOAR platform to trigger an automated endpoint isolation block.


## Automated Incident Response

In cybersecurity, we use SOAR (Security Orchestration, Automation, and Response). By adding AI to SOAR, the system doesn't just blindly pull wires; it reads emails, analyzes threats, and makes logical decisions on how to contain an attack before humans even finish pouring their coffee.

The goal of AI in incident response is to drastically reduce Mean Time to Respond (MTTR)

Take a reported Phishing Email case. Normally, an analyst spends 15 minutes checking the sender's domain, looking up links on VirusTotal, and inspecting headers. An AI-Augmented Playbook works like this:

- Ingestion: The email hits the phishing inbox.
- Extraction: An LLM extracts the true sender, all IP addresses, and any hidden URLs.
- Enrichment: The SOAR platform takes the LLM's extracted IPs and queries Threat Intelligence APIs.
- Triage Output: The LLM reads the API results and writes a final summary: "90% confident this is a credential harvesting attack. The domain was registered yesterday."

Advanced SOAR integrations use Autonomous AI Agents built on frameworks like LangChain or Microsoft AutoGen. These agents are given "Tools" (API access) and operate in a ReAct (Reasoning and Acting) loop.

The ReAct Loop for Endpoint Containment: When an EDR (Endpoint Detection and Response) alert fires for suspicious behavior, the Autonomous AI Agent wakes up.

- Thought: "I see an alert for suspicious lateral movement from Desktop-44. I need to check the active network connections on Desktop-44."
- Action: `run_tool("CrowdStrike_Get_Network_Connections", {host: "Desktop-44"})`
- Observation: Output reveals an active SMB connection to a Domain Controller on port 445 transferring large files.
- Thought: "Data exfiltration or ransomware lateral movement is likely occurring. I must isolate the host to prevent domain compromise."
- Action: `run_tool("CrowdStrike_Network_Contain", {host: "Desktop-44"})`

This entire process occurs in milliseconds. However, to prevent catastrophe, high-risk actions (like isolating a CEO's laptop or shutting down a database server) are gated behind "Human-in-the-Loop" (HITL) checkpoints. The AI prepares the isolation script, but pings the SOC Manager on Slack/Teams with a simple [Approve] or [Deny] button.


## Threat Hunting with Generative AI

Historically, Threat Hunting required experts who knew incredibly complex database languages. Today, with Generative AI, a security analyst can simply ask the AI in plain English: "Show me any computer in the company that downloaded a file from an unknown country and then immediately tried to access the password vault." The AI translates that question into the complex code needed to search the company's network.

Generative AI acts as a universal translator for SIEM Query Languages (like Splunk's SPL or Microsoft Sentinel's KQL).

Beyond tracking normal logs, AI is heavily utilized in Purple Teaming.

- Red Team: Attackers.
- Blue Team: Defenders.
- Purple Team: Working together to test defenses.

An AI can behave as a generic Red Team advisor. You provide the AI with a list of your servers, and it maps out simulated attack paths based on the latest threat intelligence (the MITRE ATT&CK Framework). It essentially writes the script for the drill, allowing the Blue Team to test if their alarms actually work.

For advanced engineers, leveraging LLMs via API for Dynamic Query Generation and Synthetic Log Generation is a game-changer.

1. KQL (Kusto Query Language) Generation Pipeline: Creating dynamic hunts in Azure Sentinel normally requires deep syntax knowledge. You can deploy an internal Streamlit app that connects to the OpenAI API with a heavily engineered system prompt.
```
user_input = "Find powershell executing encoded commands from the Temp folder"
system_prompt = """
You are a KQL Expert for Azure Sentinel.
Schema: SecurityEvent (EventID, ProcessName, CommandLine)
Rule: Always return valid KQL syntax formatted in markdown.
"""
# The LLM outputs the exact query for immediate execution:
# SecurityEvent
# | where EventID == 4688
# | where ProcessName =~ "powershell.exe"
# | where CommandLine contains "-e" or CommandLine contains "-encodedcommand"
# | where CommandLine contains @"Temp"
```

2. Synthetic Attack Emulation (Adversarial Data Generation): Machine Learning models need "bad data" to learn how to detect attacks. However, real ransomware logs are dangerous and rare. SOC teams use Generative AI to create Synthetic Threat Data.

By prompting an LLM with: "Generate 50 realistic Windows Event ID 4688 logs showing a Cobalt Strike beacon injecting into lsass.exe", the AI produces perfectly formatted, safe JSON logs. These synthetic logs are heavily injected into the SIEM development environment to train anomaly detection models and validate correlation rules without ever detonating real malware on the network.    