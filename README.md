
# Phrase meaning

* Making request on behalf of users = this is a process where you can get token key and secret of any users






# How do I use OAuth1.0 to get information?
* OAuth1.0 feels like a bit complicated than OAuth2.0. 
* I tried to manually perform OAuth 1.0a operation but it seems like a complex process and `creating a signature` was mess. The documentation and error handling wasn't giving me much direction. 
* A common error was
```javascript
{
    code: 215, 
    message: "Bad authentication data"
}
```
which does not direct me anywhere.
* So I choose a node package to handle the OAuth1.0. 





# NPM: node-oauth or oauth
* It can be found by searching google "oauth npm"
* Here is the direct link to that package: https://www.npmjs.com/package/oauth
* It has both OAuth1.0 and OAuth2.0.
* I'll note down the code that I've used to perform oAuth login..





# NPM: basic requirement of 'OAuth' library.
* consumer key - it refers to your app, it does not refer to user. 
* consumer secret key -  it refers to your app, it does not refer to user. 
* user token - with this token you are accessing a twitter user's information. It refers to user, not app. Whoever this token belongs to, OAuth1.0a will get that user's information like email, id, name and etc.
* user token secret 

<br>

* Here is the code explaining how to do it:
```javascript
// install oauth - npm install oauth
import { OAuth } from 'oauth';

async getTwitterUserEmail() {

    const oauth = new OAuth(
      'https://api.twitter.com/oauth/request_token',
      'https://api.twitter.com/oauth/access_token', // these links should be same as it is here, it won't need any changing.
      '08SLC0wePPPlNjqK6CTqvvMYa', // consumer key
      'SEDrfYG6o4dEpZ7VIoQ3BbpEYlNwSkJyJcFMx39A5SrEFRxP5a', // consumer secret key
      '1.0A',
      null,
      'HMAC-SHA1'
    );

    const twitterUserInfo = await new Promise((resolve, reject) => {
      oauth.get(
        'https://api.twitter.com/1.1/account/verify_credentials.json?include_email=true',
        '1809909964976373760-Nhk3fiGA30GhCYYzQFC09qRa3PzVEk', // User token
        'VEqvaHnncwcvJphd8WVdat4vVavwYVIn4FAKcssw84mvK', // User secret
        function (e, data, res) {
          if (e) {
            console.error(e);
            reject(e)
          }
          // console.log(data);
          const parsedData = JSON.parse(data);
          // console.log(parsedData.email);
          resolve(parsedData);
        }
      );
    })

    const email = twitterUserInfo.email;
    const name = twitterUserInfo.name;
```


<br>
<br>




### Fetching user token and token secret key
* I don't know the user token and user token secret key. 
* I'll have to fetch them. This is where the `real challenge` comes. How to get those user token and user token secret.



-------------------------------------------------------------
-----------------------------------------------------------
-------------------------------------------------------------

# Obtaining user token and user token secret 
* The whole process are seperated into 3 steps.
```
1. request token 
2. authorize request using url redirect(where a twitter user click "authorize")
3. finally get token and token secret.
```

## Step 1: request token
* Requirement: `OAuth 1.0a signature generator for node and the browser`.
* Install `OAuth 1.0a signature generator for node and the browser`
```
npm install oauth-signature 
```
* Understand which link to send `request` to and what should be the `query string`.
* send `POST` request to: `https://api.x.com/oauth/request_token?oauth_callback=https%3A%2F%2Fadmin.shopify.com%2Fstore%2Ftestinghoneybeeherb%2Fapps%2Fsnappy%2FsocialLoginAuth`
* A common `axios` request in javascript wlll look like this: 
```javascript

    const response = await axios.post(`https://api.x.com/oauth/request_token?oauth_callback=https%3A%2F%2Fadmin.shopify.com%2Fstore%2Ftestinghoneybeeherb%2Fapps%2Fsnappy%2FsocialLoginAuth`, null ,{
      headers: {
        'Content-Type': 'application/json',
        "Authorization": `OAuth oauth_consumer_key="08SLC0wePPPlNjqK6CTqvvMYa", oauth_callback="https%3A%2F%2Fadmin.shopify.com%2Fstore%2Ftestinghoneybeeherb%2Fapps%2Fsnappy%2FsocialLoginAuth" ,oauth_signature_method="HMAC-SHA1", oauth_timestamp="${this.timestamp}", oauth_nonce="${this.nonce}",oauth_version="1.0", oauth_signature="${signature}"`
      },
    })

    console.log(response.data);
```
> Don't put too much focus on the `request headers`. It will be explained or noted down later.

* `oauth_callback` value must need to be `percent encoded.`
```
https://api.x.com/oauth/request_token?oauth_callback=https%3A%2F%2Fadmin.shopify.com%2Fstore%2Ftestinghoneybeeherb%2Fapps%2Fsnappy%2FsocialLoginAuth
```

#### Generate signature
* nonce = its a random string. It can be any string. But it will have to be unique. 
* oauth_timestamp = its simply a current time of requesting. But it has to be in `specific format`,  and there is not much complication about it. I can ask chatgpt about it and it will give me proper information about it. 
* Here is a javascript code of generating signature.
```javascript
import oauthSignature from "oauth-signature";


const httpMethod = 'POST';
const baseUrl = 'https://api.x.com/oauth/request_token';
const timestamp = Math.floor(Date.now() / 1000)

generateNonce() { // Generate a random string
    const randomString = Math.random().toString(36).substring(2, 15);
    const timestamp = Date.now().toString(36);
    const uniqueString = randomString + timestamp;
    return uniqueString;
}

const params = {
    oauth_callback: 'https://admin.shopify.com/store/testinghoneybeeherb/apps/snappy/socialLoginAuth', // it can not be percent encoded here.
    oauth_consumer_key: '08SLC0wePPPlNjqK6CTqvvMYa',
    oauth_nonce: generateNonce(), 
    oauth_signature_method: 'HMAC-SHA1',
    oauth_timestamp: timestamp,
    oauth_version: '1.0'
};

const oauth_consumer_secret = 'SEDrfYG6o4dEpZ7VIoQ3BbpEYlNwSkJyJcFMx39A5SrEFRxP5a';

const signature  = oauthSignature.generate(httpMethod, baseUrl, params, oauth_consumer_secret);
console.log(signature);

```
> There goes the signature. Now make a `POST` request with in the following way:

```javascript 
const q = await axios.post(`https://api.x.com/oauth/request_token?oauth_callback=https%3A%2F%2Fadmin.shopify.com%2Fstore%2Ftestinghoneybeeherb%2Fapps%2Fsnappy%2FsocialLoginAuth`, null ,{
      headers: {
        'Content-Type': 'application/json',
        "Authorization": `OAuth oauth_consumer_key="08SLC0wePPPlNjqK6CTqvvMYa", oauth_callback="https%3A%2F%2Fadmin.shopify.com%2Fstore%2Ftestinghoneybeeherb%2Fapps%2Fsnappy%2FsocialLoginAuth" ,oauth_signature_method="HMAC-SHA1", oauth_timestamp="${timestamp}", oauth_nonce="${nonce}",oauth_version="1.0", oauth_signature="${signature}"`
      },

    })

console.log(q.data); 
// response: oauth_token=f3XucQAAAAAButyaAAABkXnXtME&oauth_token_secret=ORpa1cxwXQVuDwSTRI5bgBMiiPIfSJEb&oauth_callback_confirmed=true
```
* So, first process is done.


<br>

### Step 2: Perform webpage redirect with specific link
* Redirect or make request in web browser, and the link will ask user's permission just like oAuth2.0 and redirect again to the `callback url` I have provided in the `first step`
* Here is the link [request type = GET]: `https://api.x.com/oauth/authorize?oauth_token=Z6eEdO8MOmk394WozF5oKyuAv855l4Mlqo7hhlSLik`
* Don't have to include anything to `headers-authorization` or `form-body`. It will be empty.
* The `oauth_token` is the token got in the `first step`.
* The `GET` request should be done in browser, not in `axios`.
* This link will ask user's <b>authorization</b>. After authorization, the webpage will automatically redirect back to the `oauth_callback` including the `queryString`.
* The link will look like this
```
https://domain.com/callback?oauth_token=GRGP3QAAAAAButyaAAABkXlqD-k&oauth_verifier=QeNE8UA0pQuv7YvQOmpVmPtpo4QOjtEE
```


### Step 3: Getting user token and user token secret key
* make a `POST` request in this following like.
* Don't have to include anything to `headers` or `body`

```
https://api.x.com/oauth/access_token?oauth_token=9Npq8AAAAAAAx72QBRABZ4DAfY9&oauth_verifier=4868795
```
* This `oauth_token` is similar to step 2's `token`.
* The `oauth_verifier` is similar to step 2 `token`.
* When you'll make this request, the response will be something like:
```
oauth_token=1809909964976373760-Nhk3fiGA30GhCYYzQFC09qRa3PzVEk&oauth_token_secret=VEqvaHnncwcvJphd8WVdat4vVavwYVIn4FAKcssw84mvK&user_id=1809909964976373760&screen_name=mahinul9020
```
* `oauth_token` is the user token.
* `oauth_token_secret` is the user token secret.
* These two keys were needed in the `OAuth1.0` signature.
```javascript
// remember this code snippet?
const twitterUserInfo = await new Promise((resolve, reject) => {
      oauth.get(
        'https://api.twitter.com/1.1/account/verify_credentials.json?include_email=true',
        '1809909964976373760-Nhk3fiGA30GhCYYzQFC09qRa3PzVEk', // User token
        'VEqvaHnncwcvJphd8WVdat4vVavwYVIn4FAKcssw84mvK', // User secret - I did all those operation above to get this two code
        function (e, data, res) {
          if (e) {
            console.error(e);
            reject(e)
          }
          const parsedData = JSON.parse(data);
          resolve(parsedData);
        }
      );
    })
```





# Note 
* Creating <b>signature</b> felt like really a hassle for me. Everytime I followed the documentation, I wasn't able to create a signature with raw code. That's why I used a Node package named: `OAuth 1.0a signature generator for node and the browser`

# OAuth 1.0a signature generator for node and the browser
Link: https://www.npmjs.com/package/oauth-signature?activeTab=readme

> It made signature making less painful. Use this to make Oauth1.0 signature.


### 
