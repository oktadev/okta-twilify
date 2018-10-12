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

## Usage

`twilify` is a command line tool and should be used as such. Here is the help
information:

```console
$ twilify --help
Usage: twilify [options]

Options:
  -v, --version                          output the version number
  -i, --init                             Initialize the CLI
  -o, --okta-token <oktaToken>           Okta SSWS token
  -u, --okta-org-url <oktaOrgUrl>        Okta Org URL
  -s, --twilio-account-sid <accountSid>  Twilio Account SID
  -t, --twilio-auth-token <authToken>    Twilio Auth Token
  -p, --prefix <areaCode>                Your company's phone number prefix, e.g. 415
  -w, --webhook-url <webhookUrl>         Your Twilio Webhook URL, e.g. https://yourdomain.com/hooks/twilify
  -h, --help                             output usage information
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
