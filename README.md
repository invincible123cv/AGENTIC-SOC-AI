<h1>Installing and importing Google GenAI library</h1>
<p>Download the Google Generative AI SDK in order to use Gemini models in the code</p>
<br>

```scss
pip install google-genai
```
<br>
<p>Import genai SDK from google library to our code</p>
<br>

```scss
from google import genai
```
<br>

<h1>Getting out Google API key</h1>
<p>First create an account on [Google AI Studio](https://aistudio.google.com). Then go to the API keys section, then create your own free API key to use in this project
</p>
<br>

```python
gemini_client = genai.Client(api_key="YOUR API KEY")
```
<br>
<p>Initialize a variale for user input</p>
<br>

```python
prompt = input("Your question: ")
```
<br>
<p>And send request to Gemini via our API</p>
<br>
<p>Output the response</p>
<br>

```python
print(response.text)
```
<br>
<p>So the overall process is</p>
<br>

```python
User input → Gemini API → Model processes → Returns response object → Extract text → Print
```
<br>
<p>Complete code section:</p>
<br>

```python
from google import genai

gemini_client = genai.Client(api_key='YOUR_API_KEY')

prompt = input("Enter your question: ")

response = gemini_client.models.generate_content(
    model='gemini-2.5-flash',
    contents={'text' : prompt}
)

print(response.text)
```

<br>
<p>Further more if we want to configure the agent's behavior and identity we can add</p>
<br>

```python
from google import genai

gemini_client = genai.Client(api_key='YOUR_API_KEY')

system_identity = input("Identify the system's identity: ")
prompt = input("Enter your question: ")


response = gemini_client.models.generate_content(
    model='gemini-2.5-flash',
    contents={'text' : prompt},
    config= types.GenerateContentConfig(system_instruction=system_identity)
)

print(response.text)
```

<br>

<h1>Forcing JSON output with Google Gemini API</h1>
<p>In order to configure the response to be returned in JSON format, just simply add the following in the config in the response variable</p>
<br>

```python
config= types.GenerateContentConfig(
        system_instruction=system_identity,
        response_mime_type="application/json"
        )
```

<br>
<p>However, the response may vary from each time, which results in different JSON structure. In order to predefine the JSON schema or the output's structure, content and constraints, we can add:</p>
<br>

```python
response = gemini_client.models.generate_content(
    model='gemini-3-flash-preview',
    contents={'text': prompt},
    config=types.GenerateContentConfig(
        system_instruction=system_identity,
        response_mime_type="application/json",
        response_schema={
            "type": "object",
            "properties": {
                "event_id": {"type": "string"},
                "times": {"type": "string"},
                "source": {
                    "type": "object",
                    "properties": {
                        "ip": {"type": "string"},
                        "hostname": {"type": "string"},
                        "user": {"type": "string"},
                        "location": {"type": "string"}
                    }
                },
                "event": {
                    "type": "object",
                    "properties": {
                        "type": {"type": "string"},
                        "description": {"type": "string"},
                        "log_summary": {"type": "string"}
                    }
                },
                "analysis": {
                    "type": "object",
                    "properties": {
                        "threat_level": {"type": "string"},
                        "confidence_score": {"type": "number"},
                        "attack_type": {"type": "string"},
                        "indicators": {
                            "type": "array",
                            "items": {"type": "string"}
                        },
                        "reasoning": {"type": "string"}
                    }
                },
                "response": {
                    "type": "object",
                    "properties": {
                        "recommended_actions": {
                            "type": "array",
                            "items": {"type": "string"}
                        },
                        "automated_response": {"type": "string"},
                        "priority": {"type": "string"}
                    }
                }
            }
        }
    )
)
```
<br>
<p>Mental note:</p>
<br>

```scss
Schema
 └── type: object
     └── properties
         ├── event_id  ← property (string)
         ├── timestamp ← property (string)
         └── source    ← property (object)
                └── properties
                    ├── ip
                    └── hostname
```

<br>
<p>However, the output is not exactly easy to read, as there is no ident or space between each object and property.</p>
<br>

```scss
"log_summary":"User clicked on link in email 'Action Required: Benefit Update'. Process 'cmd.exe' spawned by 'outlook.exe' then executed obfuscated PowerShell."},"analysis":{"threat_level":"CRITICAL","confidence_score":0.95,"attack_type":"Phishing / Remote Access Trojan (RAT)","indicators":["http://it-support-portal.net/login","C:\\Users\\alice.smith\\AppData\\Local\\Temp\\srv.exe","185.122.3.4"],"reasoning":"Observed email delivery followed by user click and abnormal process tree behavior consistent with initial access via phishintion.","log_summary":"User clicked on link in email 'Action Required: Benefit Update'. Process 'cmd.exe' spawned by 'outlook.exe' then executed obfuscated PowerShell."},"analysis":{"threat_level":"CRITICAL","confidence_score":0.95,"attack_type":"Phishing / Remote Access Trojan (RAT)","indicators":["http://it-support-portal.net/login","C:\\Users\\alice.smith\\AppData\\Local\\Temp\\srv.exe","185.122.3.4"],"reasoning":"Observed email delivery followed by user click and abnormal process tree behavior consistent with initial access via phishing."},"response":{"recommended_actions":["Isolate the workstation from the network immediately","Reset Alice Smith's AD and M365 credentials","Perform memory forensics on WRK-FIN-09","Scan the mail server for other recipients of the phishing email"],"automated_response":"Workstation network port disabled via NAC","priority":"HIGH"}}  

# It's basically a string currently
```

<br>
<p>So we have to use the json library to format it correctly.</p>
<br>

```python
import json #at the start

data = json.loads(response.text)
print(json.dumps(data, indent = 2))
```
<br>

<h1>Tools</h1>
<p>Opposed to <b>reponse_schema</b> in the config where it controls the data format, <b>tools</b> define what possible actions the AI can suggest based on the input.</p>
<br>
<p>So imagine we configure the system instruction as </p>
<br>

```scss
You are a SOC analyst.

Analyze logs and detect threats.

If malicious:
- use block_ip for malicious IPs
- use disable_user for compromised accounts
- use isolate_host for infected machines

Always call a tool when threat is HIGH or CRITICAL.
```

<br>
<p>And input this prompt as a simulated incident</p>
<br>

```scss
Multiple failed login attempts detected for user admin from IP 185.122.3.4 within 2 minutes. Total attempts: 40.
```
<br>
<p>Then based on the tools we previously configured and stored, it should pick one out and suggest it.
</p>
<br>

```python
from google import genai
from google.genai import types
import json

gemini_client = genai.Client(api_key='YOUR_API_KEY')

system_identity = input("Identify the system's identity: ")
prompt = input("Enter your question: ")

additional_tools = [
    types.Tool(
        function_declarations=[
            {
                "name": "block_ip",
                "description": "Block a malicious IP address",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "ip": {"type": "string"}
                    },
                    "required": ["ip"]
                }
            },
            {
                "name": "disable_user",
                "description": "Disable a compromised user account",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "username": {"type": "string"}
                    },
                    "required": ["username"]
                }
            },
            {
                "name": "isolate_host",
                "description": "Isolate a compromised machine from the network",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "hostname": {"type": "string"}
                    },
                    "required": ["hostname"]
                }
            }
        ]
    )
]

response = gemini_client.models.generate_content(
    model='gemini-3-flash-preview',
    contents={'text': prompt},
    config=types.GenerateContentConfig(
        system_instruction=system_identity,
        tools = additional_tools
        )
)

if response.function_calls:
    for function_call in response.function_calls:
        print(f"-Tools: {function_call.name}")
        arguments = function_call.args
        for arg_name, arg_value in arguments.items():
            print(f"  - {arg_name}: {arg_value}")
else:
    print("No suggestions")
```
<br>
<p>This is just an example of tool in genai, in the future this would be used to decide which log analytic table to look into</p>
<br>

<h1>Querying Log Analytics Workspace with Python</h1>
<h2>Create a honeyspot VM</h2>
<p>First we need to create a Resource Group in Azure as a folder to store all the VM, networks, etc.</p>
<br>
<img width="806" height="489" alt="image" src="https://github.com/user-attachments/assets/568e1315-d8b7-4cb8-b31a-0a4370556e3f" />
<br>
<p>Then create a virtual network</p>
<br>
<img width="805" height="410" alt="image" src="https://github.com/user-attachments/assets/b086eb88-a01a-4a3d-b01a-92c892ba767f" />
<br>
<p>And an intentionally vulnerable Windows VM exposed to the public by configuring all ports on the VM to be reachable.</p>
<br>
<img width="801" height="325" alt="image" src="https://github.com/user-attachments/assets/e763214b-64d0-4207-a064-264d61e74595" />
<br>
<h2>Create Log Analytics Workspace and Configure DCR (Data Collection Rule)</h2>
<br>
<p>First, create a Log Analytics Workspace, configuring it to be included the previously setup Resource Group</p>
<br>
<img width="807" height="426" alt="image" src="https://github.com/user-attachments/assets/28175510-c202-4f46-835f-a691b0e684f2" />
<br>
<p>Then, we need to create a DCR to collect resources with types as Windows logs and set the destination to our created LAW</p>
<br>
<img width="808" height="415" alt="image" src="https://github.com/user-attachments/assets/c30006e8-4a96-4046-adac-0c97181c9539" />
<br>
<p>Then in the Resource section below, connect the Windows VM from which we want to collect the logs, the AMA (Azure Monitoring Agent) would we automatically created within the Windows VM allowing us to collect logs from</p>
<br>
<img width="801" height="328" alt="image" src="https://github.com/user-attachments/assets/612a2040-908f-4f80-a50d-b172d6bb4723" />
<br>
<h2>Querying Logs with Python</h2>
<p>In order to authenticate with Azure log analytics workspace and query form it, we need to install some modules.</p>
<br>

```python
pip install azure-identity
pip install azure-monitor-query
```

<br>
<p>Import some necessary modules:</p>
<br>

```python
from datetime import timedelta
from azure.identity import DefaultAzureCredential
from azure.monitor.query import LogsQueryClient
import pandas as pd
```
<br>
<p>Then install Azure CLI on your host machine</p>
<br>

```python
winget install --exact --id Microsoft.AzureCLI
```
<br>
<p>Query the logs from python</p>
<br>

```python
from datetime import timedelta
from azure.identity import DefaultAzureCredential
from azure.monitor.query import LogsQueryClient
import pandas as pd

  

LOG_ANALYTICS_WORKSPACE_ID = ""

log_analytics_client = LogsQueryClient(credential=DefaultAzureCredential())

hours_ago = 24

# KQL Query

kql = f'''
Event |
where EventLog == "Security"
| where EventID  == "4625"
| project TimeGenerated, Computer, EventID
'''

  
# Execute the query
response = log_analytics_client.query_workspace(
    workspace_id = LOG_ANALYTICS_WORKSPACE_ID,
    query = kql,
    timespan=timedelta(hours=hours_ago)
)


print(respnse.text)
```

<br>
<p>The output would include all the columns and rows in this table
</p>
<br>
<img width="805" height="295" alt="image" src="https://github.com/user-attachments/assets/a9175eda-c8c0-418c-885f-495a44f92117" />
<br>
<p>However, the output is barely structured  like this</p>
<br>

```scss
['TenantId', 'SourceSystem', 'TimeGenerated', 'Source', 'EventLog', 'Computer', 'EventCategory', 'EventLevel', 'EventLevelName', 'UserName', 'Message', 'ParameterXml', 'EventData', 'EventID', 'RenderedDescription', 'MG', 'ManagementGroupName', 'AzureDeploymentID', 'Role', 'Type', '_ResourceId']
[['741f8549-4056-4b06-8f4e-a81695530772', 'OpsManager', datetime.datetime(2026, 4, 20, 13, 20, 0, 792780, tzinfo=<iso
```
<br>
<h2>Format the query output format with Panda</h2>

```python
df = pd.DataFrame(rows, columns=columns)

df["TimeGenerated"] = pd.to_datetime(df["TimeGenerated"].dt.strftime("%Y-%m-%d %H:%M:%S")) # format date time (previous:2026-04-20 13:20:00.792780+00:00 )

print(df)
```
<br>

```scss
        TimeGenerated   Computer  EventID
0 2026-04-20 13:20:00  Windows10     4625
1 2026-04-20 13:20:09  Windows10     4625
```
<br>
<h1>Querying from Log Analytics with Functions</h1>

```python
from datetime import timedelta
from azure.identity import DefaultAzureCredential
from azure.monitor.query import LogsQueryClient
import pandas as pd

FIELDS = {
    "SecurityEvent" : "TimeGenerated,Computer,IpAddress,Process",
    "DeviceLogonEvents" : "TimeGenerated, AccountName, DeviceName,IpAddress"
}

TABLE_NAME = "SecurityEvent"

LOG_ANALYTICS_WORKSPACE_ID = ""



log_analytics_client = LogsQueryClient(credential=DefaultAzureCredential())

hours_ago = 24


def query_log_analytics(client,workspace_id,table,fields,timespan):


    # KQL Query
    kql = f'''
    {table} | 
    project {fields}
    '''

    # Execute the query
    response = client.query_workspace(
        workspace_id = workspace_id,
        query = kql,
        timespan=timedelta(hours=timespan)
    ) 

    #Extract the table

    table = response.tables[0]
    rows = table.rows
    columns = table.columns

    df = pd.DataFrame(rows, columns=columns)
    df["TimeGenerated"] =            pd.to_datetime(df["TimeGenerated"].dt.strftime("%Y-%m-%d %H:%M:%S"))
    records = df.to_csv(index=False)

    print(records)

query_log_analytics(log_analytics_client,LOG_ANALYTICS_WORKSPACE_ID,TABLE_NAME,FIELDS[TABLE_NAME],hours_ago)
```

<br>
<h1>Gemini, Token Count, Cost and Model Selection</h1>
<p>When we use Gemini, all text (your inputs and the model's response) is broken down into tokens.</p>
- A token is usually about 4 characters of English text on average
- Every sent request (your prompt) and every response received from Gemini cost tokens
- Tokens are the currency of interaction with Gemini: prompt tokens, response tokens
<br>
<p>Here is a comparison between Gemini different models: https://ai.google.dev/gemini-api/docs/models#gemini-3</p>
<br>
<h2>Basic Setup</h2>
<p>In order to calculate the the total tokens we just need to import the <b>genai</b> module from google library</p>

```python
from google import genai
```

<br>
<p>And optionally install another module named `colorama` to color some of our outputs</p>
<br>

```python
from colorama import init, Fore, Style
```
<br>

```scss
init   → enable colors
Fore   → text color
Style  → formatting + reset
```
<br>

<h2>Complete code</h2>

```python
from google import genai
from log_analytics import records
from colorama import init, Fore, Style

# CONFIG

init(autoreset=True)

API_KEY = ''
MAX_OUTPUT_TOKEN = 5000
DEFAULT_MODEL = "gemini-3-flash-preview"
current_model = DEFAULT_MODEL

models = {
    "gemini-3.1-pro-preview" : {"max_window_tokens" : 1048576, "max_outout_tokens" : 65536, "cost_per_mil" : 2},
    "gemini-3-flash-preview" : {"max_window_tokens" : 1048576, "max_outout_tokens" : 65536, "cost_per_mil" : 0.5},
    "gemini-2.5-flash" : {"max_window_tokens" : 1048576, "max_outout_tokens" : 65536, "cost_per_mil" : 0.3},
    "gemini-2.5-pro" : {"max_window_tokens" : 1048576, "max_outout_tokens" : 65536, "cost_per_mil" :1.25}
}

# CLIENT

client = genai.Client(api_key=API_KEY)

# CALCULATE COST FUNCTION

prompt =  f"You are a threat hunter. Any anomalies or potential breaches?\n{records}"

def calculate_chat_cost(tokens,cost_per_mil):
    return round((tokens/1000000)*cost_per_mil,6)


# ESTIMATED TOTAL TOKENS AND COST
estimated_input_tokens = len(prompt)//4
estimated_output_tokens = MAX_OUTPUT_TOKEN
estimated_total_tokens = estimated_input_tokens + estimated_output_tokens 
estimated_total_cost = calculate_chat_cost(estimated_total_tokens, models[DEFAULT_MODEL]["cost_per_mil"])
choice = input(
    f"Chat will cost approximately ${estimated_total_cost:.4f}\n"
    "Proceed? (y/n): "
).strip().lower()
if choice != "y":
    exit(0)

# ACTUAL TOKEN USAGE
response = client.models.generate_content(
    model = DEFAULT_MODEL,
    contents = {"text": prompt},
    config = {
        "max_output_tokens" : MAX_OUTPUT_TOKEN
    }
)

usage = response.usage_metadata
total_tokens = usage.total_token_count
total_cost = calculate_chat_cost(total_tokens, models[DEFAULT_MODEL]["cost_per_mil"])

tokens_status = (Fore.GREEN + "UNDER" + Style.RESET_ALL) if total_tokens <= estimated_total_tokens else (Fore.RED + "OVER" + Style.RESET_ALL) 
cost_status = (Fore.GREEN + "UNDER" + Style.RESET_ALL) if total_cost <= estimated_total_cost else (Fore.RED + "OVER" + Style.RESET_ALL)


print(f"Estimated tokens: {estimated_total_tokens}")
print(f"Total tokens: {total_tokens} [{tokens_status}]")
print(f"Total cost: ${estimated_total_cost}")
print(f"Total cost: ${total_cost} [{cost_status}]")
print("=========Answer=========")
print(response.text)
```
