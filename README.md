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

```scss
gemini_client = genai.Client(api_key="YOUR API KEY")
```
<br>
<p>Initialize a variale for user input</p>
<br>

```scss
prompt = input("Your question: ")
```
<br>
<p>And send request to Gemini via our API</p>
<br>
<p>Output the response</p>
<br>

```scss
print(response.text)
```
<br>
<p>So the overall process is</p>
<br>

```scss
User input → Gemini API → Model processes → Returns response object → Extract text → Print
```
<br>
<p>Complete code section:</p>
<br>

```scss
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

```scss
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

```scss
config= types.GenerateContentConfig(
        system_instruction=system_identity,
        response_mime_type="application/json"
        )
```

<br>
<p>However, the response may vary from each time, which results in different JSON structure. In order to predefine the JSON schema or the output's structure, content and constraints, we can add:</p>
