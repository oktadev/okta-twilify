# okta-twilify

[Okta](https://developer.okta.com) is the biggest and best-est API service for handling user identity,
authentication, authorization, password reset, multi-factor authentication, and
just about involving user data.

Many people use Okta to store user accounts for their companies and business
projects.

[Twilio](https://www.twilio.com) is the best and most popular API service for
communications: sending and receiving SMS messages, faxes, phone calls, and all
sorts of other communications-related stuff.

If you happen to be using Okta to store employee accounts for your business,
`twilify` will take your business to a whole new level.

By downloading and running this tool in a cron-like fashion, you'll be able to
automatically assign each employee at your company a dedicated company phone
number.

These phone numbers can be in an area code (US only right now) that you specify,
and will forward phone calls, text messages, and faxes through to each
individual employee as if it was their own.

`twilify`  provides a simple, inexpensive (Twilio costs pennies), and automated way
to manage a business phone system without servers.

## Installation

You can install `twilify` via [npm](https://www.npmjs.com/), the Node package
manager. `twilify` runs on Node 10 and above. It may work on older releases but
no promises.

```bash
$ npm install okta-twilify
```

## Setup

Once you've got the program installed, it's time to get everything setup!

I'm going to assume that you are using Okta and already have some users, so I
won't walk through that. But there is one thing you do need to know.

Each Okta user you have should set their mobile phone number in their profile.
Every Okta user has a profile field named `mobilePhone` that should contain the
employee's personal cell phone (if they choose).

`twilify` will ONLY purchase company phone numbers for employees who have a cell
phone listed in this field. This prevents you from purchasing phone numbers
through Twilio (phone numbers are typically $1/mo/number) that won't be used.

Once you've made sure that at least some of your users have a mobile number set
in their Okta profiles, you need to setup a new Twilio account.

1. Create a new Twilio account here: https://www.twilio.com/try-twilio
2. Take note of your **Account SID** and **Auth Token** values. These are your
   Twilio API keys. `twilify` needs to know these values. They can be found on
   your [Twilio dashboard page](https://www.twilio.com/console).
3. Go to the [Twilio functions page](https://www.twilio.com/console/runtime/functions/manage)
   and click the big red plus button. This will create a new serverless function.
4. Name your first Function **SMS Forward**. This function will run every time a
   new text message is sent to one of your company numbers. It will forward the
   text message along to the employee who owns that phone number.

   ```javascript
   const okta = require("@okta/okta-sdk-nodejs");
   const MemoryStore = require("@okta/okta-sdk-nodejs/src/memory-store");

   exports.handler = function(context, event, callback) {
     const twilioClient = context.getTwilioClient();
     const oktaClient = new okta.Client({
       orgUrl: process.env.OKTA_ORG_URL,
       token: process.env.OKTA_TOKEN,
       requestExecutor: new okta.DefaultRequestExecutor(),
       cacheStore: new MemoryStore({ keyLimit: 100000, expirationPoll: null })
     });

     let user;

     oktaClient.listUsers({
       search: 'profile.primaryPhone eq "' + event.To + '"'
     }).each(u => {
       user = u;
     }).then(() => {
       twilioClient.messages.create({
         to: user.profile.mobilePhone,
         from: event.To,
         body: "From: " + event.From + "\n\n" + event.Body
       }, (err, message) => {
         callback();
       });
     });
   };
   ```
5. Click **Save** and persist your new function.
6. Now, return to the [Twilio functions
   page](https://www.twilio.com/console/runtime/functions/manage) and create a
   new function. This function should be named **Call Forward**, the **Path**
   value should be set to `/call-forward`, and paste the code below into the
   editor then click **Save**. This function will run when anyone calls one of
   your company phone numbers. It will forward the call to the employee who owns
   the particular number.

   ```javascript
   const okta = require("@okta/okta-sdk-nodejs");
   const MemoryStore = require("@okta/okta-sdk-nodejs/src/memory-store");

   exports.handler = function(context, event, callback) {
     const oktaClient = new okta.Client({
       orgUrl: process.env.OKTA_ORG_URL,
       token: process.env.OKTA_TOKEN,
       requestExecutor: new okta.DefaultRequestExecutor(),
       cacheStore: new MemoryStore({ keyLimit: 100000, expirationPoll: null })
     });

     let user;

     oktaClient.listUsers({
       search: 'profile.primaryPhone eq "' + event.To + '"'
     }).each(u => {
       user = u;
     }).then(() => {
       let twiml = new Twilio.twiml.VoiceResponse();

       twiml.say("Please wait. You are being connected to " + user.profile.firstName + ".");
       twiml.dial({
         callerId: event.From ? event.From : undefined,
         answerOnBridge: true
       }, user.profile.mobilePhone);
       twiml.say("Goodbye.");

       callback(null, twiml);
     });
   };
   ```
7. Now that your functions are created, you need to define some environment
   variables so your serverless functions can talk to Okta. Visit the
   [Twilio configuration page](https://www.twilio.com/console/runtime/functions/configure)
   and create two new keys: one named **OKTA_ORG_URL** and another one named
   **OKTA_TOKEN**. Set their values appropriately.
8. Now, click the big plus sign near the Dependencies section and add a new
   dependency: the Okta Node SDK. The name of the package name is
   `@okta/okta-sdk-nodejs` and the version should be `1.2.0`.
9. Finally, visit the [Functions
   page](https://www.twilio.com/console/runtime/functions/manage) again, click
   on one of the two newly created functions, and copy down the function URL. It
   should look something like this
   `https://toolbox-bobcat-2584.twil.io/call-forward`. Get rid of the path and
   just copy down the https URL, e.g. `https://toolbox-bobcat-2584.twil.io`. You
   will need this when configuring `twilify` for the first time.

## Usage

`twilify` is a command line tool and should be used as such. Here is the help
information:

```console
$ twilify --help
Usage: twilify [options]

Options:
  -v, --version                                           output the version number
  -i, --init                                              Initialize the CLI
  -o, --okta-token <oktaToken>                            Okta SSWS token
  -u, --okta-org-url <oktaOrgUrl>                         Okta Org URL
  -s, --twilio-account-sid <accountSid>                   Twilio Account SID
  -t, --twilio-auth-token <authToken>                     Twilio Auth Token
  -p, --prefix <areaCode>                                 Your company's phone number prefix, e.g. 415
  -f, --twilio-function-base-url <twilioFunctionBaseUrl>  Your Twilio Functions Base URL, e.g. https://toolbox-bobcat-xxxx.twil.io
  -h, --help                                              output usage information
```

I suggest running `twilify --init` to get started in interactive mode. By
running this command you'll be prompted for all the information the program
needs to run, and a new config file will be created for you at
`~/.config/twilify/config.json` which will hold your settings.

Once this config file has been created you can run `twilify` without any
additional parameters and it will retrieve the stored settings from the config
file to work.

## Releases

**0.0.1**: *Released 10-11-2018*

- First release ever!
