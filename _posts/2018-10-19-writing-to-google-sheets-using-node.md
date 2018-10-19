---
layout: post
title: Writing to Google Sheets using Node JS
tags: ['node', 'google-sheets', 'javascript']
published: false
---

[Source Code](https://github.com/PeterPlucinski/google-sheets-and-node)

[Documentation](https://developers.google.com/sheets/api/quickstart/nodejs)

## Part 1 - using Google's documentation and sample code, generating credentials.json and token.json

Initialize an empty project using `npm init`

1. Follow step in the Google documentation to enable the Google API. This will let you select or create an empty project which will contatin your Google Sheets spreadsheet. You will need to download the `credentials.json` file your local project root.

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

3. In step 3 of their guide, Google provides some useful exmaple code for working with their API. I've reproduced the code below. Note that this of course may change in the future. We'll be re-using some of this code and modyfying it for our purposes. The code does authentication and reads out some data stored in a sample spreadsheet.

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

4. You can place the sample code provided and run it using `node .` - when you do, you will be provided with a URL to click on in the command line. Follow the URL and you will need to give permission for your script to interact with your Google Sheets project via the API. You will receive a token which you will need to past into your command line. After this, the script will generate a `token.json` file in your project root.

My `token.json` file is below (again, sensitive values have been altered):

```
{  
   "access_token":"ab12.abcdefghijklmnopqrstuvwxyz12-_abcdefghijklmnopqrstuvwxyz123456789abcdefghijklmnopqrstuvwxyz123456789abcdefghijklmnopqrst_-123",
   "refresh_token":"1/abcdefghijklmno--1-abcde_abcdefghijklmnopq-abcdefghijklmnopq_123",
   "scope":"https://www.googleapis.com/auth/spreadsheets.readonly",
   "token_type":"Bearer",
   "expiry_date":1122334455667
}