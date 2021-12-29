# Alert Me

## Email alerts

### Set up credentials
Directions are based on the [Gmail API Quickstart instructions for Python](https://developers.google.com/gmail/api/quickstart/python).

1. [Create a project and enable an API](https://developers.google.com/workspace/guides/create-project).
2. Sign in with ASU credentials
3. If needed, return to the above link to enable the Gmail API.
4. Enter a project name (Default "Quickstart" is fine).
5. Choose "Desktop app" from the dropdown and hit "Create".
6. Click "DOWNLOAD CLIENT CONFIGURATION" to download JSON credential file.
7. Check JSON credential path below (e.g., `~/Downloads/credentials.json`) and change if necessary.
8. Move JSON credential file to a permanent home (e.g., `~/credentials.json`. Several examples are given below. Be sure to replace paths with the correction versions for you system(s).

```
# In R
file.rename(from='~/Downloads/credentials.json',to='~/credentials.json')
```

```
# In shell
mv ~/Downloads/credentials.json ~/credentials.json'
```

```
# Between systems (e.g., to Agave)
rsync --progress ~/Downloads/credentials.json <ASURITE>@agave.asu.edu:/home/<ASURITE>/
```

### Set up R package

Install [gmailr](https://gmailr.r-lib.org/).

```
# Install latest release from CRAN
install.packages('gmailr')

# Alternatively, install the development version from GitHub
# devtools::install_github('r-lib/gmailr')
```

The very first time the above is run on the system, you will need to sign in and authorize access. Once this is done, you should be able to load the credentials and send emails without needing to reauthorize, which is critical for programmatic use. Be sure to adjust the path to the credentials JSON file as needed.

```
library(gmailr)
gm_auth_configure(path = '/home/<ASURITE>/credentials.json')
```

## Push alerts 

### Set up credentials
Directions are based on the [pushoverr R package documentation](https://briandconnelly.github.io/pushoverr/).

### Set up credentials
1. Register for a free account on the [Pushover homepage](https://pushover.net).
2. Confirm account using the link set to your email address.
3. Download the Pushover app on your device. There are [iOS](https://apps.apple.com/us/app/pushover-notifications/id506088175), [Android](https://play.google.com/store/apps/details?id=net.superblock.pushover), and [Desktop](https://pushover.net/clients/desktop) versions.
4. Sign in to the app using your Pushover credentials.
5. Go to the [Pushover homepage](https://pushover.net) on your browser.
6. [Create a new application/API token](https://pushover.net/apps/build). You can name the app whatever you like (e.g., "RAlerts") 
7. Note the API token/key for your new app, as well as your user key.

### Set up R package

Install [pushoverr](https://briandconnelly.github.io/pushoverr/).

```
# Install latest release from CRAN
install.packages('pushoverr')

# Alternatively, install the development version from GitHub
# devtools::install_github('briandconnelly/pushoverr')
```

Send yourself a test message. You will need the API token/key as well as the user key set up earlier to configure the app. Be sure to change those values below.

```
library(pushoverr)

set_pushover_user(user='<YOUR USER KEY>')
set_pushover_app(token='<YOUR APP TOKEN>')

# Send yourself a test message
pushover('Hello world')
```

## Load functions

Now that the [email](Email-alerts) and [push](Push-alerts) alerts have been set up, The following functions can be loaded.

Load the `email.status` function. Be sure to replace <ASURITE> with your ASURITE for the arguments "to", "from", and "auth".

```
email.status = function(topic=NULL,subj=NULL,msg=NULL,to='<ASURITE>@asu.edu',from='<ASURITE>@asu.edu',auth='/home/<ASURITE>/credentials.json') {
	suppressMessages(require(gmailr))
	if (is.null(subj)) m.subject = paste0(paste(Sys.info()[c('user','nodename')],collapse='@'),': ',ifelse(is.null(topic),'Action',topic),' completed') else m.subject = subj
	if (is.null(msg)) m.html.body = paste0('<p>',ifelse(is.null(topic),'Action',topic),' completed at ',Sys.time(),'.</p>') else m.html.body = msg
	m.to = to
	m.from = from
	gm_auth_configure(path=auth)
	gm_mime() %>%
		gm_to(m.to) %>%
		gm_from(m.from) %>%
		gm_subject(m.subject) %>%
		gm_html_body(m.html.body) %>%
		gm_send_message() %>% suppressMessages()
}
```

Load the `push.status` function. Be sure to replace <YOUR USER KEY> and <YOUR APP TOKEN> with your user key and application token from the [Pushover homepage](https://pushover.net).

```
push.status = function(topic=NULL,msg=NULL,user='<YOUR USER KEY>',token='<YOUR APP TOKEN>') {
	suppressMessages(require(pushoverr))
	if (is.null(msg)) m.msg = paste0(paste(Sys.info()[c('user','nodename')],collapse='@'),': ',ifelse(is.null(topic),'Action',topic),' completed') else m.msg = msg
	pushover(
		message=m.msg,
		user=user,
		app=token
	)
}
```

Once the functions above have been loaded they can be run as follows.

```
# Send an email alerting me that my "Long task" has completed.
email.status('Long task')

# Send a push alert alerting me that my "Long task" has completed.
push.status('Long task')
```

The subject and body of emails as well as the contents of push alerts follow predefined syntaxes. Custom messages can be set following the examples below.

```
# Send an email with custom subject and message
email.status(subj='The long task',
	msg='It is finally over. Run back to your computer.')

# Send a push alert with custom content
push.status(msg='The long task is finally over.')
```
