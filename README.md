# Migrating from ~~Apps for your Domain~~ ~~G Suite~~ Workspaces Legacy to personal GMail

For all steps I'll assume user@example.com is your Workspaces user and user@gmail.com is your GMail user.

## Google Voice

Do this first as Legacy Voice is going away at some point and I'm not sure it'll keep working when it does.

1. Log in to both accounts in a single browser session
1. Visit https://voice.google.com/ as user@gmail.com
1. Accept the terms of service but _DO NOT_ create a Google voice number
1. Visit https://www.google.com/voice/b/0#phones as user@example.com
1. Click 'Transfer' next to your number
1. You should be eligible to transfer to user@gmail.com assuming:
    1. You don't have a Voice number on user@gmail.com
    1. You have accepted the ToS as user@gmail.com

Notes:

* No content will transfer, only the number itself

## Allowing sending as user@example.com from user@gmail.com

Note: 2 Factor Auth must be enabled on user@gmail.com

1. Visit https://myaccount.google.com/apppasswords as user@gmail.com
1. Choose 'Mail' for app
1. Choose Other under device and enter "smtp for alternate froms"
1. Copy the password somewhere temporary
    1. You can reuse this for more accounts later
3. Visit https://mail.google.com/mail/#settings/accounts as user@gmail.com
4. Under "Send mail as" click "Add another email address"
5. Enter your user@example.com address details
6. Make sure "Treat as an alias" is checked, click "Next Step"
7. Enter `smtp.gmail.com` as the SMTP Server, port should be `587`
8. Username is your gmail account, user@gmail.com
9. Password is the app password we just copied
10. Make sure TLS is selected, click Add Account

A confirmation email will be sent to user@example.com.
Once confirmed you're able to choose that email address as a 'From' when sending.

You can reuse the same app password to set up as many email aliases as you need.
