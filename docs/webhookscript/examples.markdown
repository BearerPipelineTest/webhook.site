---
title: Examples
nav_order: 500
parent: WebhookScript
---

# WebhookScript Examples

!!! info 
    Do you have a nice example to share with other users? Or looking for even more examples? Take a look at the WebhookScript example repository, and make a pull request if you want to contribute: [https://github.com/fredsted/webhookscripts](https://github.com/fredsted/webhookscripts)
    
## Convert a date

In this example, if we assume the variable `$mydate$` is set to `2021-07-26T16:23:50+03:00`, the variable will be overwritten to `2021.07.26 16:23` for actions running after the WebhookScript action.

```javascript
input = var('$mydate$')

output = date_format(input, 'YYYY.MM.DD HH:mm')

set('$mydate$', output)
```

For more information about available date format characters, see here: https://docs.webhook.site/webhookscript/date-format.html

## Parse and loop through JSON

```javascript
// Define input as a JSON string
json = '{
    "items": [
        {
            "first_name": "Jack",
            "last_name": "Daniels",
            "phone": "+1 100-555-999",
            "group_ids": [346, 46456, 23423]
        },
        {
            "first_name": "Jim",
            "last_name": "Beam",
            "phone": "+1 123-555-788",
            "group_ids": [3456, 43546, 234234, 456456]
        }
    ]
}'

// Decode to array
data = json_decode(json)

// Define an array of valid groups
valid_groups = [43546]

// Loop over items
for (item in data['items']) {
    has_valid_group = false

    for (group_id in item['group_ids']) {
        if (valid_groups.contains(group_id)) {
            has_valid_group = true
        }
    }
    
    if (has_valid_group) {
        dump(item['first_name'] + ' did not have a valid group.') 
    } else {
        dump(item['first_name'] + ' has a valid group.') 
    }
}
```

Returns the following output:

```
"Jack has a valid group."
"Jim did not have a valid group."
```

## Loop through and compare items

In this example, we loop through a series of items and pick the item that's contained in a string. 

```javascript
location = 'test ABC example';

compares = [
  '123': 'token1',
  'ABC': 'token2',
  'DEF': 'token3',
]

token = ''; // Default value

for (compare in array_keys(compares)) {
    if (location.contains(compare)) {
        token = compares[compare]
    }
}

dump(token) // token2
```

## Submit request with escaped JSON

If you're building a JSON object, we recommend doing it in WebhookScript instead of typing JSON in the Send Request action type (If you do anyway, we recommend using the `.json` Variable Modifier, [More info here](/custom-actions/variables.html#variable-modifiers)). 

In this example, one of the JSON values contain HTML generated using the string_format function.

```javascript
html_template = '<b><u>New {} lead</u></b><br>
<br>
Location: {}<br>
Message from customer:<br>
<div style="background:#CCC">{}</div>'

html_message = string_format(
    html_template,
    var('lead_type'),
    var('location'),
    var('message')
)

payload = json_encode([
    'lead': [
        'firstname': var('firstname'),
        'lastname': var('lastname'),
        'html': html_message
    ]
])


request(
  'https://example.com/leads',
   payload,
   'POST',
   ['Content-Type: application/json']
)
```

## Validate request

In this example, we use a common method of verifying webhooks by taking a hash of its contents concatenated to a secret. It demonstrates the way WebhookScript can get various information about the request by using the `get_variable()` function, as well as string concatenation, [hashing](/webhookscript/functions/string.html#hashstringnumber-value-string-algo-string), if statements and returning responses with content, status codes and headers using `respond()`, which halts execution.

```javascript
verification_secret = "JHRlc3RTY3JpcHRTZWNyZXQ"
verification_challenge = var("request.header.x-request-verification")
verification_result = hash(var("request.content") + verification_secret, "sha256")

if (!verification_challenge or verification_challenge != verification_result) {
    respond("Invalid request", 500)
}

respond("Successful request", 200)
```

## Send a x-www-form-urlencoded request

```javascript
content = query([
    'country': 'Curaçao', 
    'population': 158665
])
headers = ['Content-Type: application/x-www-form-urlencoded'];
response = request('https://example.com', content, 'POST', headers);
```

## Transform and resend

In the following, an incoming request is JSON decoded to an array, transformed and sent to "Web Service 1". Then the output is saved and passed on to "Web Service 2" in XML format. Basic error handling and validation is demonstrated.

```javascript
// Configuration, fetched from the users' Global Variables in Control Panel
ws1_api_key = var('WS1_KEY')
ws2_user_token = var('WS1_USER_TOKEN')

// Function for error handling which stops processing further actions/code and returns an error message
function error (message) {
    echo('Error: {}'.format(message))
    respond(json_encode(['error': message]), 500)
}

// Parse original request
orig_req = json_decode(var('request.content'))

// If the JSON was invalid
if (!orig_req) {
  error('Invalid request')
}

// Send request to Web Service 1, using format() for string placeholders
// with JSON decoded values from the incoming request body
ws1_url = 'https://ws1.example.com/3.0/lists/{}/interest-categories/{}/interests'.format(
  orig_req['listId'],
  orig_req['groupId']
)

ws1_content = [
  'first_name': orig_req['firstName'],
  'last_name': orig_req['lastName']
]
ws1_response = request(
    ws1_url,
    json_encode(ws1_content),
    'POST',
    ['Authorization: Basic ' + ws1_api_key]
)

// Don't go further if the Web Service 1 step didn't succeed
if (ws1_response['status'] != 200) {
  echo(ws1_response['content']); // Log content to output
  error('Invalid response from WS1')
}

// Get a value from the Web Service 1 request
ws1_response_id = json_decode(ws1_response)['id']

// Pass response on to Web Service 2 in XML format, using a multi-line string and format()
ws2_content = '
  <qdbapi>
    <usertoken>{}</usertoken>
    <listid>{}</listid>
    <field fid="7">{}</field>
  </qdbapi>'.format(ws2_user_token, orig_req['listId'], ws1_group_id)
  
ws2_response = request(
  'https://ws2.example.com/db/zzzzzz',
  ws2_content,
  'POST', 
  [
    "Action: API_EditRecord", 
    "Content-Type: application/xml"
  ]
)

if (ws2_response['status'] != 200) {
  echo(ws2_response['content']); // Log content to debug log
  error('Invalid response from WS2')
}

// Output the WS2 response content to debug output
echo(ws2_response['content'])
respond('OK', 200)
```

## Telegram bot

The Messaging service Telegram allows bots using their API. The general principle is this:

1. [Create a Bot using the /newbot command](https://core.telegram.org/bots#6-botfather) sent to the *BotFather* Telegram User
2. Using the bot token sent from *BotFather*, use the Telegram API to [create a Webhook subscription](https://core.telegram.org/bots/api#setwebhook) (using your Webhook.site URL)
3. Add some logic using WebhookScript!

Note: Everywhere you see `TELEGRAM_TOKEN`, replace it with the token you got from BotFather!

### Subscribe to Webhook

To create the Webhook subscription, change the token and the Webhook.site URL to your own and go to the following URL in your browser:

https://api.telegram.org/bot**TELEGRAM_TOKEN**/setWebhook?url=**https://webhook.site/a1351781**

You should get a response similar to this:

```json
{
  "ok": true,
  "result": true,
  "description": "Webhook was set"
}
```

### First incoming Webhook

When you add your bot to your Telegram contacts list, Telegram automatically sends a `/start` command to the bot, which triggers a Webhook similar to this:

```json
{
  "update_id": 176446573,
  "message": {
    "message_id": 1,
    "from": {
      "id": 2346545645,
      "is_bot": false,
      "first_name": "Simon",
      "language_code": "en"
    },
    "chat": {
      "id": 34534673234,
      "first_name": "Simon",
      "type": "private"
    },
    "date": 1581706369,
    "text": "/start",
    "entities": [
      {
        "offset": 0,
        "length": 6,
        "type": "bot_command"
      }
    ]
  }
}
```

You should be able to see this in the Webhook.site requests list.

From this, we have all the parts needed to build a script that answers to commands:

```javascript
// Telegram API token
token = 'TELEGRAM_TOKEN';

content = json_decode(var('$request.content$'));
msg = content['message']['text'];
response = "Couldn't come up with anything witty.";

if (msg == "How's it going?") {
    response = 'Pretty good.'
}

if (msg == r"You're (.*)") {
    match = regex_extract_first(r"You're (.*)", msg)
    response = 'No, YOU are {}'.format(match);
}

if (msg == "/start") {
    response = "Hi! I'm WebhookBot."
}

url = 'https://api.telegram.org/bot{}/sendMessage'.format(token)

json = [
    'chat_id': content['message']['chat']['id'],
    'text': response
]

request(url, json_encode(json), 'POST');
```

Things to note:

* The API token is added to the script, but could also have been saved in Global Variables in Control Panel and fetched out with the `var()` function.
* The third if-statement uses regex matching to provide a dynamic response. Someone typing "You're a bot" would receive "No, YOU are a bot"
* Finally, we JSON encode a WebhookScript array and send it using the `request()` function.

Simply copy this script into a WebhookScript Custom Action (remember to change the token!), and click Save Action.

!["WebhookScript" Telegram Bot screenshot](/images/example-telegram-bot-script.png)

Then, you can interact with the bot using the Telegram app:

!["WebhookScript" Telegram Bot screenshot](/images/example-telegram-bot.png)

And that's it! Congratulations on your bot. It's not very smart, but from here, the possibilities are endless!

## Building HTML content

The following script builds a piece of HTML content using the string_format function, based on previously defined variables, and shows how to use a function to return different content based on input.

After this, it sends a JSON request (by converting an array to JSON via the json_encode function) containing the HTML using basic Bearer authentication.

```javascript
function alert_class() {
    if (status == 'Operational') { return 'success'; }
    if (status == 'Degraded Performance') { return 'warning'; }
    if (string_contains(status, 'Disruption')) { return 'danger'; }
}

template = '
  <div class="alert alert-{}">
    <h2 class="alert-title">{} - {}</h2>
    <p>
      {}<br />
      State: {}<br />
      Component affected: {}
    </p>
    <p>{}</p>
  </div>';

postbody = string_format(
  template,
  alert_class(),
  message,
  state,
  component,
  to_date('now').date_format('LLLL')
)
  
json = [
  'alert': [
    'title': '{} - {}'.format(component, state),
    'body': postbody,
    'draft': false,
    'types': ['alert']
  ],
]

request(
  'http://example.com/alerts',
  json_encode(json),
  'POST',
  [
    'Content-Type: application/json',
    'Authorization: Basic {}'.format(token)
  ]
)
```

## Uploading and parsing CSV file

With this script, a file upload form is displayed when visiting the URL. After submitting the form, the CSV file is processed and validated (in this example, there must be more than 2 rows). If it can't be validated, an error message is shown. Finally, the user is shown an "Upload successful" message if the CSV file is valid.

```javascript
url = var('request.url')
set_header('content-type', 'text/html');

// Display file upload form and exit if HTTP method is not POST
if (var('request.method') != 'POST') {
    respond('
        <html>
            <head><title>Upload CSV</title></head>
            <body>
                <h1>Upload CSV</h1>
                <form action="{}" method="POST" enctype="multipart/form-data">
                    <input type="file" name="file"/>
                    <button type="submit">Upload</button>
                </form>
            </body>
        </html>
    '.format(url))
}

// Use a comma as delimiter and treat first row (0) as header row
array = csv_to_array(var('request.file.file.content'), ',', 0)

// If CSV can't be parsed, or there's less than 2 rows, fail
if (!array or array.length() < 2) {
    respond('
        <h1>Could not parse CSV</h1>
        <a href="{}">Upload again</a>
    '.format(url));
}

// Display the parsed CSV in JSON format 
respond('
    <h1>Upload successful</h1>
    <pre>{}</pre>
    <p>
        <a href="{}">Upload again</a>
    </p>
'.format(json_encode(array), url))
```
