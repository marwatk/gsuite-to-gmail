# Migrating from ~~Apps for your Domain~~ ~~G Suite~~ Workspaces Legacy to personal GMail

For all steps I'll assume user@example.com is your Workspaces user and user@gmail.com is your GMail user.

This is a WIP as I work out the kinks.

## Google Voice

Do this first as Legacy Voice is going away at some point (this month!) and I'm not sure transfer will still work when it does.

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
  * There doesn't appear to be a way to import anything to Google Voice, so preserve your `Takeout/Google Voice` folder if it's important to you

## Start a Google Takeout

This takes a while (in my case almost 24 hours), so best to start it now while we do other steps.

1. Visit [https://takeout.google.com](https://takeout.google.com)
1. Make sure the things you want are selected
1. Choose either .zip or tar.gz, I chose tar.gz because I thought I could `curl | tar`, but you can't, so it really doesn't matter
1. Wait for it to finish (you'll get an email to that account)

## Allow sending as user@example.com from user@gmail.com

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

## Mail fowarding user@example.com -> user@gmail.com

### CloudFlare

I use CloudFlare for their awesome Argo Tunnel allowing me to publish web servers without exposing ports anywhere. Their email forwarding is in beta and it takes about 2 weeks to get in once requested. Once accepted, here are the steps to forward to your user@gmail.com account

1. Log in to CloudFlare and choose your domain
1. Click 'Email (Beta)'
1. Add either individual aliases or a set up a catch-all
1. Verify your target address by clicking the link in their confirmation email

### Namecheap

If using namecheap as your registrar and DNS they have free mail redirection.

1. Click 'Manage' on the domain in your domain list
1. Click the 'Advanced DNS' tab on the top right
1. Scroll down to 'Mail Settings'
1. Switch from whatever settings you were on (e.g. 'Gmail') to 'Email forwarding'
1. Click back to the 'Domain' tab on the top
1. Scroll down to the 'Redirect Email' settings and fill out as needed

### Other Services

I don't use any other services but would welcome instructions for them.

## Moving email messages from user@example.com to user@gmail.com

There are several methods to do this, including [setting up the old account in the new account as POP source](https://www.39digits.com/migrate-g-suite-account-to-a-personal-google-account#gmail), but that takes a while and doesn't create a local backup. 

Instead I used [Got Your Back](https://github.com/GAM-team/got-your-back) for backing up and restoring email. You could follow the instructions there, or you can use the [script/docker container I wrote](https://github.com/marwatk/gyb-docker). You should read it over to make sure it's not doing anything nefarious (it's pretty short).

For this step you're going to require a Unix shell with Docker. I tried to make it Mac compatible, but don't have a Mac to test with so ymmv.

You'll still need to trust GYB and while it's longer than my script it's pretty auditable as well.

These will be given access to your Google account, you probably shouldn't just run them blindly.

**Note:** It's easiest to paste the URLs GYB generates into an incognito window that only has one Google account signed in since links you paste don't let you choose which account to use and will load into the first one.

**Important Note:** You should never have to enter your actual Google password anywhere but Google on the web during this process. And definitely never on the shell. If you do something has gone horribly wrong and do not continue. GYB uses OAuth and never needs your actual password. Its access can be revoked in [your Google Account permissions](https://myaccount.google.com/permissions) (and we will do so at the end).

```bash
# Download the script
curl -s --fail -L https://raw.githubusercontent.com/marwatk/gyb-docker/main/gyb > gyb

# Audit the script (it's pretty simple, don't just run stuff you find on the internet)
cat gyb

chmod a+x gyb

# Replace this with an actual empty folder
export BACKUPS_DIR=/path/to/backups

# Replace this with your apps admin account (can be the same account)
export APPS_ADMIN_EMAIL=admin@example.com

# Set up account credentials, you'll need to do this once per Google Apps Domain
./gyb --email "${APPS_ADMIN_EMAIL}" --action create-project
# Follow the prompts

# Authorize service account for the domain (again, once per domain)
./gyb --email "${APPS_ADMIN_EMAIL}" --action check-service-account
# Follow the prompts

# Replace this with your actual email
export APPS_EMAIL=user@example.com

# Do the backup, repeat this step for as many domain accounts you want to back up
./gyb --email "${APPS_EMAIL}" --service-account

# Now set up credentials and back up your user@gmail.com
# Edit this value
export GMAIL=user@gmail.com

# Set up account credentials
./gyb --email "${GMAIL}" --action create-project
# Follow the prompts
# Make sure to choose read/write access since we'll be restoring to this account

# Do the backup
./gyb --email "${GMAIL}"

# Repeat the previous two steps for as many target gmail
# accounts as you have

# Now we restore the user@example.com mail to user@gmail.com (don't mess with --local-folder arg, it's universal)
# You can repeat this for as many GMAIL/APPS_EMAIL pairings as you have as long as you have done the above steps for them first
./gyb --email "${GMAIL}" --action restore --local-folder "/backups/GYB-GMail-Backup-${APPS_EMAIL}"
```

After that's complete visit [your Google Account permissions](https://myaccount.google.com/permissions) on each account and revoke the access you just granted in case either my script or GYB decided to hijack your OAuth credentials. Or leave it and continue to use GYB (which is awsome) to back up your email.

Also visit [Domain delegation](https://admin.google.com/ac/owl/domainwidedelegation) for each domain account and remove the the GYB access there.

## Extract your Takeouts

For the rest of the migration you'll need your Google Takeout data. I couldn't find a way to easily automate this, so for me that meant manually downloading 26 50GB files.

Download each account to an account-specific folder. Then extract them. You can do this manually or:


```bash
cd <the folder>

# If you downloaded tarballs
find . -maxdepth 1 -type f -name "*.tgz" -exec tar -xzvf {} \;

# If you downloaded zips
find . -maxdepth 1 -type f -maxdepth 1 -name "*.zip" -exec unzip {} \;
```

## Calendar

This is pretty simple, in your takeout extract there's a `Takeout/Calendar` folder and inside there are `.ics` files for each calendar in your account. Just visit the settings on the target account's calendar and then import the files where you'd like them in your calendar.

There are missing items like guests and guest status, but I don't think there's a way to import those easily.

## Contacts

Similar to Calendar, there's a `Takeout/Contacts` folder. Inside there's an `All Contacts/All Contacts.vcf`. Importing that seemed to include all the other groups, but ymmv. Import as needed.

## Photos

I had read horror stories of Takeout mangling photos and separating photos and metadata, but for me at least that turned out to not be the case. The photos I spot checked all had proper metadata matching the original uploads.

I'm going to be paying for Google One for my family at 2TB, so I didn't worry about reuploading these images as they were the storage-saver quality. I'm going to upload my originals instead to my personal account.

That said, there are several ways to get photos saved to a different account. The easiest way is to set up partner sharing with the new account making sure to set "Save Photos to Your Account" to "All Photos" on the new account. Rumor has it this can take several days. Performing a Takeout of the target account and comparing the folders is probably wise.

## Drive

There are several complicated ways to do this, each with pros and cons:

* [Downloading via desktop sync and upload to alternate account](https://www.39digits.com/migrate-g-suite-account-to-a-personal-google-account#google-drive)
  * Loses sharing settings
* [Using G Suite Business](https://www.tabgeeks.com/tabgeeks-blog/how-to-transfer-ownership-between-domains-in-google-drive)
  * Requires temporarily upgrading to a higher tier

I went with the former. Or if you don't care about sharing or preserving the Google Doc-iness you can just upload the files from your `Takeout/Drive` folder.

## Others

For other items I decided to leave them in their old accounts since I'll be persisting them. If/when I look to migrate them I'll update this document. Also open to additions.

## Downgrading your Legacy Account to Cloud Identity Free

I wanted my Google Accounts to stick around for my purchase, YouTube videos, etc so deleting my Workspaces account wasn't an option. I also wasn't sure where I had used "Login with Google", so want that to stick around. So instead we convert it to an account similar to one for using Google Cloud.

**DO NOT PERFORM THESE STEPS BEFORE MOVING YOUR VOICE NUMBER, YOU WILL LOSE IT**

If you 'upgrade' to paid it no longer includes Google Voice and you will immediatly lose access to Voice. Great ugprade, Google. We're not actually going to pay for our subscription, just use it long enough to add Cloud Identity Free.



