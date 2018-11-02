---
layout: post
title: Writing to Google Sheets using Node JS
tags: ['node', 'google-sheets', 'javascript']
published: false
---

[Source Code](https://github.com/PeterPlucinski/google-sheets-and-node)

[Documentation](https://developers.google.com/sheets/api/quickstart/nodejs)

## Part 1 - Using Google's documentation and sample code to generate credentials

Initialize an empty project using `npm init`

1. Follow the first step in the Google documentation to enable the Google API. This will let you select or create a project for which to enable the Google Sheets API (It doesn't seem to matter which project you choose). You will need to download the `credentials.json` file to your local project root.

For reference, my `credentials.json` file looks like this (sensitive values have been changed):

```
{  
   "installed":{  
      "client_id":"112233445566-abcdefghijklmnopqrstuvwxyz123456.apps.googleusercontent.com",
      "project_id":"test-1234a",
      "auth_uri":"https://accounts.google.com/o/oauth2/auth",
      "token_uri":"https://www.googleapis.com/oauth2/v3/token",
      "auth_provider_x509_cert_url":"https://www.googleapis.com/oauth2/v1/certs",
      "client_secret":"abcdefghijklmnopqrstuvwx",
      "redirect_uris":[  
         "urn:ietf:wg:oauth:2.0:oob",
         "http://localhost"
      ]
   }
}
```

2. Following the guide, install the Google APIs client side library: `npm install googleapis@27 --save`

3. In step 3, there is some useful example code for working with the API. I've reproduced the code below. Note that this of course may change in the future. We'll be re-using some of this code and modifying it for our purposes. The example code does authentication and reads out some data stored in a sample spreadsheet.

```
const fs = require('fs');
const readline = require('readline');
const {google} = require('googleapis');

// If modifying these scopes, delete token.json.
const SCOPES = ['https://www.googleapis.com/auth/spreadsheets.readonly'];
const TOKEN_PATH = 'token.json';

// Load client secrets from a local file.
fs.readFile('credentials.json', (err, content) => {
  if (err) return console.log('Error loading client secret file:', err);
  // Authorize a client with credentials, then call the Google Sheets API.
  authorize(JSON.parse(content), listMajors);
});

/**
 * Create an OAuth2 client with the given credentials, and then execute the
 * given callback function.
 * @param {Object} credentials The authorization client credentials.
 * @param {function} callback The callback to call with the authorized client.
 */
function authorize(credentials, callback) {
  const {client_secret, client_id, redirect_uris} = credentials.installed;
  const oAuth2Client = new google.auth.OAuth2(
      client_id, client_secret, redirect_uris[0]);

  // Check if we have previously stored a token.
  fs.readFile(TOKEN_PATH, (err, token) => {
    if (err) return getNewToken(oAuth2Client, callback);
    oAuth2Client.setCredentials(JSON.parse(token));
    callback(oAuth2Client);
  });
}

/**
 * Get and store new token after prompting for user authorization, and then
 * execute the given callback with the authorized OAuth2 client.
 * @param {google.auth.OAuth2} oAuth2Client The OAuth2 client to get token for.
 * @param {getEventsCallback} callback The callback for the authorized client.
 */
function getNewToken(oAuth2Client, callback) {
  const authUrl = oAuth2Client.generateAuthUrl({
    access_type: 'offline',
    scope: SCOPES,
  });
  console.log('Authorize this app by visiting this url:', authUrl);
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  rl.question('Enter the code from that page here: ', (code) => {
    rl.close();
    oAuth2Client.getToken(code, (err, token) => {
      if (err) return console.error('Error while trying to retrieve access token', err);
      oAuth2Client.setCredentials(token);
      // Store the token to disk for later program executions
      fs.writeFile(TOKEN_PATH, JSON.stringify(token), (err) => {
        if (err) console.error(err);
        console.log('Token stored to', TOKEN_PATH);
      });
      callback(oAuth2Client);
    });
  });
}

/**
 * Prints the names and majors of students in a sample spreadsheet:
 * @see https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit
 * @param {google.auth.OAuth2} auth The authenticated Google OAuth client.
 */
function listMajors(auth) {
  const sheets = google.sheets({version: 'v4', auth});
  sheets.spreadsheets.values.get({
    spreadsheetId: '1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms',
    range: 'Class Data!A2:E',
  }, (err, res) => {
    if (err) return console.log('The API returned an error: ' + err);
    const rows = res.data.values;
    if (rows.length) {
      console.log('Name, Major:');
      // Print columns A and E, which correspond to indices 0 and 4.
      rows.map((row) => {
        console.log(`${row[0]}, ${row[4]}`);
      });
    } else {
      console.log('No data found.');
    }
  });
}
```

4. You can place the sample code provided in `index.js` and run it using `node .` - you will be provided with a URL to click on in the command line. Follow the URL and you will need to log in to your Google account and give permission for your script to interact with Google Sheets. You will receive a token which you will need to paste back into the command line. After this, the script will generate a `token.json` file in your project root.

My `token.json` file is reproduced below (again, sensitive values have been altered):

```
{  
   "access_token":"ab12.abcdefghijklmnopqrstuvwxyz12-_abcdefghijklmnopqrstuvwxyz123456789abcdefghijklmnopqrstuvwxyz123456789abcdefghijklmnopqrst_-123",
   "refresh_token":"1/abcdefghijklmno--1-abcde_abcdefghijklmnopq-abcdefghijklmnopq_123",
   "scope":"https://www.googleapis.com/auth/spreadsheets.readonly",
   "token_type":"Bearer",
   "expiry_date":1122334455667
}
```

## Part 2 - Creating a node server endpoint and saving data to Google Sheets

First, we'll need to include the node http component so we can create a node server.

```
const http = require('http');
```

We'll also modify the `SCOPES` config value so we can write to the spreadsheet. We'll also extract the spreadsheet ID into a config variable and create a `RANGE` variable which tells the API where to append values. Note, when modifying `SCOPES` you'll need to delete the exising `token.json` file and re-generate it. I'm also going to setup the server port as a config variable.

```
const SCOPES = ['https://www.googleapis.com/auth/spreadsheets'];
const TOKEN_PATH = 'token.json';
const SPREADSHEET_ID = '12gsgDLjMMjDIRlbcaKuUYY8rth_ugIsH4zI-Ns98PFc';
const RANGE = 'Sheet1';
const PORT = 3000;
```

Next, we're going to remove the listMajors() example function, which reads data from a sample spreadhseet, and create a new callback which will create the node server.

```
function createServerAndGoogleSheetsObj(oAuth2Client) {
    
    const sheets = google.sheets({ version: 'v4', auth: oAuth2Client });

    const server = http.createServer((request, response) => {

        if (request.method === 'POST') {
            
            // request object is a 'stream' so we must wait for it to finish
            let body = '';
            let bodyParsed = {};

            request.on('data', chunk => {
                body += chunk;
            });

            request.on('end', () => {
                bodyParsed = JSON.parse(body);
                saveDataAndSendResponse(bodyParsed.data, sheets, response);
            });

        } else {

            // normal GET response for testing the endpoint
            response.end('Request received');

        }

    });

    server.listen(PORT, (err) => {
        if (err) {
            return console.log('something bad happened', err)
        }
        console.log(`server is listening on ${PORT}`)
    });

}
```

We are still passing in the authentication object so we can create a Google Sheets object for working with the API. Once data is sent to the endpoint via a POST request, we'll call `saveDataAndSendResponse()` which will send the data to Google Sheets and return a relevant response.

Below is the final function of our script. Its fairly self explanatory.

```
function saveDataAndSendResponse(data, googleSheetsObj, response) {

    // data is an array of arrays
    // each inner array is a row
    // each array element (of an inner array) is a column
    let resource = {
        values: data,
    };

    googleSheetsObj.spreadsheets.values.append({
        spreadsheetId: SPREADSHEET_ID,
        range: RANGE,
        valueInputOption: 'RAW',
        resource,
    }, (err, result) => {
        if (err) {
            console.log(err);
            response.end('An error occurd while attempting to save data. See console output.');
        } else {
            const responseText = `${result.data.updates.updatedCells} cells appended.`;
            console.log(responseText);
            response.end(responseText);
        }
    });

}
```

The data argument will be a multidemeninal array, allowing you to send more than one row. Each inner array is a row in the spreadsheet.

Below is an example JSON request and resulting spreadsheet update.

```
{
	"data": [[1,2], [3,4]]
}
```

Full source code for the project can be found [here](https://github.com/PeterPlucinski/google-sheets-and-node).


