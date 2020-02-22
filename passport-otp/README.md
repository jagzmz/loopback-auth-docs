# Passport-OTP Loopback

## Adding dependency

    npm i git+https://github.com/jagzmz/passport-otp.git

## Usage
Use this configuration in `authConfig.json` file.

    {
       "otp":{
          "authScheme":"otp",
          "provider":"passport-otp",
          "module":"passport-otp",
          "callbackHTTPMethod":"post",
          "resendAfter":0.5,
          "digits":6,
          "defaultOTPVia":"email",
          "redirectEnabled":false,
          "verificationRequired":false,
          "entryFlow":{
             "phoneVerificationRequired":false,
             "emailVerificationRequired":false
          }
       }
    }
## Config Description
Above are some config which can be used for this strategy, you do not need to use all of the configurations, these will be used by default, if you want to override them, you can do it by providing your own value for that config, and it will be overriden.
See [Overriding Default Configurations](#overriding-default-configuration)

|Config|Description  | Default
|--|--|--|
| resendAfter | Time to wait for a new OTP request (in minutes).|0.5|
|digits|No. of OTP digits to generate. |6
|defaultOTPVia| Default configuration to send OTP through.|email
|redirectEnabled| Passport strategy usually make a redirect on successful/unsuccessful login. | false
|verificationRequired| If verification required is set to true, requests like `updateEmail, updatePassword, updatePhone` will require OTP verification, otherwise updates will be made by user's Access Token.|false
|entryFlow| These have two keys viz. `phoneVerificationRequired, emailVerificationRequired`, if either of these are not set it won't let user to login, or won't provide Access Token while using this strategy.|false

## Overriding Default Configurations

You can override strategies default configuration according to your needs.
For this to be done, you need to just provide the key which needs to be overridden in your `authConfig.json`

**Example**: If you want a custom **resendAfter** interval for your OTP strategy, just provide 

    "otp":{
      // Resend Interval for OTP Request (in mins)
      resendAfter:1 	  
    }

Now **resendAfter** for OTP strategy will be set to **1 minute**.
