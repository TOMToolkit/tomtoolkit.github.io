---
title: Setting up email notifications
---

Sometimes it is useful to have your TOM notify you when specific circumstances occur, 
for example when new data are obtained for a target of interest.  The TOM Toolkit provides
functions to enable you to notify your users by email.  

## [Email account set-up](#email_account_setup)

The TOM will need an email account from which to send emails.  This can be set to an existing
personal account, but for security's sake is better set to a separate account dedicated for 
that purpose.  

This is because most major providers of email services (e.g. Gmail) attempt to block robot emails 
under many circumstances as a security feature and to reduce spam.  Since your TOM is a robot sending 
useful emails, it is often necessary to change the configuration of the email account to allow these
emails through.  

You can find instructions for configuring a TOM Gmail account to allow notification emails [here](https://support.google.com/accounts/answer/6010255).

## [Set-up](#setup)

During the tom_setup procedure, you will be asked if you want to configure your TOM for 
email notifications.  If so, simply agree to this option.  

This will set-up the necessary email configuration in your mytom/mytom/settings.py file.  
Rather than include confidential information in this file, you will see that some of 
these parameters are from your environment variables:

    EMAIL_HOST = os.environ['EMAIL_HOST']
    EMAIL_HOST_USER = os.environ['EMAIL_HOST_USER']
    EMAIL_HOST_PASSWORD = os.environ['EMAIL_HOST_PASSWORD']

## [Setting environment variables](#setting_env_var)

The configuration of these variables depends on the environment you are running your TOM in.

### In a virtualenv:
If you are running the TOM in a virtualenv, you can set these parameters by editing the activate script 
for that environment, e.g.:

    .../tom_env/bin/activate

To set the parameters, add the following lines at the very end of this file:

    # Setting required environment variables:
    export EMAIL_HOST=YOUR_SETTING_HERE
    export EMAIL_PORT=587
    export EMAIL_HOST_USER=YOUR_TOM_EMAIL_ADDRESS
    export EMAIL_HOST_PASSWORD=YOUR_TOM_EMAIL_PASSWORD
    export EMAIL_USE_TLS=True

and the following lines at the end of the deactivate section:

    deactivate () {
    
    ...
    
    # unset variables
    unset EMAIL_HOST
    unset EMAIL_PORT
    unset EMAIL_HOST_USER
    unset EMAIL_HOST_PASSWORD
    unset EMAIL_USE_TLS
    }

### In a Docker container:

If your TOM is running in a Docker container, these variables can be set in a number of ways:

1) In the Dockerfile using the following syntax for each of the required parameters above:
    
    ARG email_host=YOUR_VALUE_HERE
    ARG email_host_user=YOUR_VALUE_HERE
    ARG email_host_password=YOUR_VALUE_HERE
    ENV EMAIL_HOST=$email_host
    ENV EMAIL_HOST=$email_host_user
    ENV EMAIL_HOST=$email_host_password

2) Using the `environment` parameter in a docker-compose file:

    environment:
      - EMAIL_HOST=YOUR_HOST
      - EMAIL_HOST_USER=YOUR_TOM_EMAIL_ADDRESS
      - EMAIL_HOST_PASSWORD=YOUR_TOM_EMAIL_PASSWORD

3) Using the `-e` flag as a commandline parameter when you run your container:

    docker run -it -e EMAIL_HOST=YOUR_HOST -e EMAIL_HOST_USER=YOUR_TOM_EMAIL_ADDRESS -e EMAIL_HOST_PASSWORD=YOUR_TOM_EMAIL_PASSWORD ...


## [Using email notifications](#using_email)

The tom_notifications module provides functions for sending emails in `email_tools.py`.  These can be imported into your apps or views.
