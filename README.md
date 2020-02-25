

# Loopback Component Auth

  
  
  
  

# Installation

## Adding dependency


    npm i git+ssh://git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/loopback-component-auth

This component installs the following dependencies by default:

 - [loopback-component-passport](https://github.com/strongloop/loopback-component-passport)
 - [loopback-connector-sendgrid](https://github.com/Cellarise/loopback-connector-sendgrid)
 - [loopback-connector-twilio](https://github.com/dashby3000/loopback-connector-twilio)
 - [passport-facebook](https://github.com/jaredhanson/passport-facebook)
 - [passport-facebook-token](https://github.com/drudge/passport-facebook-token)
 - [passport-google-oauth](https://github.com/jaredhanson/passport-google-oauth)
 - [passport-google-token](https://github.com/michaelbats/passport-google-token)

## Create configurations

  

This component requires a **json** file (lets say `authConfig.json`) which contains configurations for the passport strategies, just like **providers.json** in loopback-component-passport. 
In the end, all the configuration will be merged in providers.json which will be used by this component.


Create a file name **authConfig.json** in your project root directory and paste the below sample configuration with required strategies along with their respective **clientId , clientSecret**.

**Note:** Otp strategy does not require **clientId , clientSecret** but **Email , Twilio** datasources instead. Check how configure Otp strategy here.

### Sample Configuration

    {
		  "local":{
		      "password":{
		         "minLen":4,
		         "digitRequired":true,
		         "lowerCase":true,
		         "upperCase":true,
		         "specialChar":true
		      },
		      "sensitiveData":[
		         "profilePic"
		      ]
		   },
          "passport": {
            "enabled": [],
            "serverURL": "http://localhost:PORT",
            "models": {
              "User": ""
            },
            "strategies": {
              "google": {
                "clientID": "",
                "clientSecret": ""
              },
              "google-token": {
                "clientID": "",
                "clientSecret": ""
              },
              "facebook": {
                "clientID": "",
                "clientSecret": ""
              },
              "facebook-token": {
                "clientID": "",
                "clientSecret": ""
              },
              "otp": {
                "provider": "otp",
                "digits":"6",
                "defaultOTPVia": "email",
                "entryFlow": {
                  "phoneVerificationRequired": false,
                  "emailVerificationRequired": true
                }
              }
            }
          }
        }


### Config Description

This component generates redirects and callback paths (if not given) for every strategy.

**Defaults Paths**
  

 - **basePath** - /auth
 - **successRedirect** - /auth-success
 - **failureRedirect** - /auth-failure

|Config|Default Value|Description
|--|--|--|
| serverURL | NA | Url on which loopback project is hosted on.|
| enabled | All strategies are enabled by default. | Enable specific strategies in passport.|
| authPath | / {**basePath**} / {**provider**} | API Path for users to make a social login.|
|successRedirect|/{**basePath**} / {**provider**} / {**successRedirect**} | Path to redirect on successful social login.|
|failureRedirect|/{**basePath**} / {**provider**} / {**failureRedirect**} | Path to redirect on un-successful social login.|
|models| `User` model defaults to loopback's inbuilt User model. | Custom `User, UserIdentity, UserCredential` models (if any) to be used by loopback-component-passport |
digits| 6|Number of OTP digits to be generated

**Note**:<br/>
 { **provider** } value is calculated based on the \`key\` of the respective strategy and splits the string if `-` is present in the string and replaces it with `/`. 
 
**Example**:<br/>
'google' - /auth/google<br/>
'google-token' - /auth/google/token  

To check the full configuration file check out [Default Configurations](#pre-defined-configuration)

### Overriding Default Configurations

You can override strategies default configuration according to your needs.
For this to be done, you need to just provide the key which needs to be overridden in your [authConfig.json](#create-configurations)

**Example**: If you want a custom **successRedirect** for your Google strategy, just provide 

    "google":{
      id...,
      secret...,
      successRedirect:"/custom-path" 	  
    }

Now **successRedirect** for Google strategy will be **`/{serverURL}/custom-path`**

	
## Preparing User model

This component comes with **predefined User Model** named `UserAuth` which inherits loopbacks User Schema along with its definition.

To use this model add the following to `model-config.json`

    "sources":{
        ...,
        "../node_modules/loopback-component-passport/lib/models"
    }

You have 2 options after this 
- Extend this model as a base for custom user model.
- Use this model as it is by providing datasource in `datasource.json` for this model named `UserAuth`.

**Note:**
**If you are using `UserAuth` as a base model, make sure to register the custom user model name inside `authConfig.json` in [models](#config-description) property.**


     
# Usage
 
## Add imports

    const LbAuth= require('loopback-component-auth');
## Initializing component

    const lbAuth=new LbAuth(app);
    const authConfig=require('/path/to/authConfig');
    //If you want to setup custom mail function
    lbAuth.setBeforeMessageFn(beforeMessageFn);
    lbAuth.setup("passport",authConfig);
Thats it.
See about [`beforeMessageFn`](#beforeMessageFn). 

# Inbuilt Configuration and Methods

## Methods

### User.signupAuth(data, req)
    POST /api/user/signupAuth
Allows user to signup

|Parameter| Type| Description | Required|
|--|--|--|--|
| data | object | User data which will contain email/password and other neccessary information for user.|email or phone
| req| object| httpRequest object| true

**Example:**

Signup using email/password.

    await User.signupAuth({
				email:"mithya@example.com",
				password:"otherthanradiant"
		})
<br/>
Signup using phone/password.

    await User.signupAuth({
				phone:{ 
						countryCode:"+91",
						phone:"8976542568"
					}
				password:"otherthanradiant"
		})
	 
countryCode will be set to `defaultCountryCode` if provided. See [Passport-OTP Loopback](/passport-otp/#sample-configuration)

**Note:**
If OTP strategy is enabled and `verificationRequired` flag is set. This method will send OTP to the `defaultOTPVia` which might be _email or phone_ else it will send otp to both if provided.
**If you want to modify the data which is being sent by OTP message, check out [Modifying verification email](#beforemessagefn).**

### User.updatePhone(countryCode, phone, req) 

    GET	/api/user/updatePhone
    AccessToken Required
Update user's phone.
OTP will be sent if strategy has `verificationRequired` is set to true. Otherwise phone will be updated using user's access token.

|Parameter| Type| Description | Required|
|--|--|--|--|
| countryCode | string | User's countryCode.|false
| phone | number | User's phone.|true
| req| object| httpRequest object| true

If verification was required OTP will be sent to given phone number,
`Twilio` model is needed to be setup along with Twilio datasource configuration in order for this to work.

### User.updatePhoneWithToken(countryCode, phone, token, req)

    GET	/api/user/updatePhoneWithToken
    AccessToken Required
Update phone using token which was sent from `User.updatePhone`.

|Parameter| Type| Description | Required|
|--|--|--|--|
| countryCode | string | User's countryCode.|false
| phone | number | User's phone.|true
| token | number | Token which was recieved with phone number|true
| req| object| httpRequest object| true

### User.updateEmail(email, req)

    GET	/api/user/updateEmail
    AccessToken Required
Update email from user's access token.
OTP will be sent on email if strategy has `verificationRequired` set to true.

|Parameter| Type| Description | Required|
|--|--|--|--|
| email | string | User's email.|true
| req| object| httpRequest object| true

### User.updateEmailWithToken(email, token, req)

    GET	/api/user/updateEmailWithToken
    AccessToken Required
Update email using token which was sent from `User.updateEmail`.

|Parameter| Type| Description | Required|
|--|--|--|--|
| email | string | User's email.|true
| token | number | Token which was recieved with email.|true
| req| object| httpRequest object| true

### User.updatePassword(oldPassword, newPassword ,defaultOTPVia , req)

    GET	/api/user/updatePassword
    AccessToken Required
Update password using user's access token.
OTP will be sent if strategy has `verificationRequired` set to true.

|Parameter| Type| Description | Required|
|--|--|--|--|
| oldPassword | string | User's old password |true if verification flag not set.
| newPassword | string | User's new password |true if verification flag not set.
| defaultOTPVia | string | OTP via email/phone |false
| req| object| httpRequest object| true

### User.updatePasswordWithToken(oldPassword,  newPassword, token, req)

    GET	/api/user/updatePasswordWithToken
    AccessToken Required
Update password with token which was sent from `User.updatePassword`.

|Parameter| Type| Description | Required|
|--|--|--|--|
| oldPassword | string | User's old password |false
| newPassword | string | User's new password |true
| token | number | Token recieved via email/phone.|true
| req| object| httpRequest object| true

### User.sendToken(data, req)

    GET	/api/user/sendToken
    AccessToken Required
Send token to email or phone.

|Parameter| Type| Description | Required|
|--|--|--|--|
| data | object | email or phone |true
|data.extras|  object| Extra fields to recieve inside customMailFunction|false
| req| object| httpRequest object| true

### User.verifyToken(data, req, extras)

    POST	/api/user/verifyToken
    AccessToken Required
Verify token for email/phone.

|Parameter| Type| Description | Required|
|--|--|--|--|
| data | object | email or phone |true
| req| object| httpRequest object| true
|extras| object||false


### User.generateCSV

    GET	/api/user/generateCSV
    Admin API
Generates an CSV containing data from all user's data after applying filter (if any).

|Parameter| Type| Description | Required|
|--|--|--|--|
| filter | object | Filter `{"where":{...}}` |false
| res| object| httpResponse object| true

### User.updateSensitiveData(data, token, req)

    POST	/api/user/updateSensitiveData
    Admin API
Updates user's sensitive data which are provided in `authConfig.json` .

|Parameter| Type| Description | Required|
|--|--|--|--|
| filter | object | Filter `{"where":{...}}` |false
| res| object| httpResponse object| true

### User.sendMessage(medium, mediumValue, data)
    Can be accessed via User Model.

This function can be used to send message to a specific medium email/phone. This method will uses [beforeMessageFn](#beforeMessageFn) as an mediator which enables programmer to set customized email/phone templates before message is being sent.

|Parameter| Type| Description | Required|
|--|--|--|--|
| medium | string | email or phone |true|
| mediumValue| string| email or phone value| true|
| data| object| May contain `requestType` and `extra data` which needs to be accessed in [beforeMessageFn's](#beforeMessageFn) data parameter. | true|

##### **sendMessage Example:** 

    await User.sendMessage({
	    'email',
	    'admin@mithya.com',
	    {
		    requestType:'emailNotification',
		    url:'https://you-have-won-lottery.com'
	    }
	 })

### User.setMessageMedium(medium, cb(mailOptions))

     Set custom provider for email/phone.
|Parameter| Type| Description | Required|
|--|--|--|--|
| medium | string | email or phone |true
| cb(mailOptions)| callback function| This function will be called before sending email/phone message | true

If you wish to setup a email/phone provider which does not have an loopback connector, you can use this function to override the default provider ( sendgrid ) and use your own provider such as NodeMailer or any other email provider or if you want to use an API to send an email.

**Example:**

    User.setMessageMedium('email',( mailOptions )=>{
		   //mailOptions contains generated template for email/phone
		   CustomEmailProvider cep= new CustomEmailProvider()
		   cep.send(mailOptions.email)
			   .then(response =>{
					   console.log('Success', response)
					   return response
				}
				.catch(error =>{
					   console.log('Error',error)
					   throw error
				}
	})
		   
### beforeMessageFn
    Should be passed inside setBeforeMessageFn
  
|Parameter| Type| Description | Required|
|--|--|--|--|
| formData | object | Will contain email and phone object which contains the respective templates  |true
| tokenData| object| Will contain generated OTP | true
|data| object | See below. | true
|User| object| User Model|true

**data** might contain fields according to the scenario
- **If an OTP is being sent**<br/>
`data` might contain,
`otpMedium` will be email/phone.
`user & accessToken` will be there only if request is sent by an existing user.
    `requestType` will be 'sendTokenRequest' if not provided in 'sendToken' function.

- **If [sendMessage](#usersendmessage-medium-mediumvalue-data) is invoked**<br/>
There won't be any `tokenData` available in this case.
`data` will contain field sent in [sendMessage's](#usersendmessage-medium-mediumvalue-data) data parameter.

**Example:**
Suppose a message is sent (see sendMessage [example](#sendmessage-example)) , following code explains how message template can be changed before sending an email.

    const beforeMessageFn= async (formData,  tokenData,  data,  User) => {
	    let requestType=data.requestType
	    switch( requestType ){
		    case 'emailNotification':{
			    let email = formData.email.to
			    let urlToRedirect = data.url
			    let notificationTemplate = require('./notificationTemplate').generate(urlToRedirect)
			    formData.email = {...formData.email, ...notificationTemplate}
			 }
        }
    }
			    
    
## Configuration
### Pre-defined configuration

    {
       "passport":{
          "basePath":"/auth",
          "successRedirect":"/auth-success",
          "failureRedirect":"/auth-failure",
          "failureFlash":true,
          "strategies":{
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
             },
             "google":{
                "provider":"google",
                "module":"passport-google-oauth",
                "strategy":"OAuth2Strategy",
                "scope":[
                   "email",
                   "profile"
                ],
                "failureFlash":true
             },
             "google-token":{
                "provider":"google",
                "module":"passport-google-token",
                "scope":[
                   "email",
                   "profile"
                ],
                "authInfo":true,
                "session":false,
                "json":true
             },
             "facebook":{
                "provider":"facebook",
                "module":"passport-facebook",
                "profileFields":[
                   "gender",
                   "link",
                   "locale",
                   "name",
                   "timezone",
                   "verified",
                   "email",
                   "updated_time"
                ],
                "scope":[
                   "email",
                   "public_profile"
                ]
             },
             "facebook-token":{
                "provider":"facebook",
                "module":"passport-facebook-token",
                "strategy":"FacebookTokenStrategy",
                "fbGraphVersion":"v3.3",
                "profileFields":[
                   "gender",
                   "link",
                   "locale",
                   "name",
                   "verified",
                   "email",
                   "updated_time"
                ],
                "scope":[
                   "email",
                   "public_profile"
                ],
                "authInfo":true,
                "session":false,
                "json":true
             }
          }
       }
    }

Strategy specific configuration are automatically merged, if those key/value pair settings are not given.

**Example:**
For OTP strategy if `resendAfter` key is not provided, default will be overrided which is 0.5 minutes. If you want to override this value, you can provide your own value in `authConfig.json` inside OTP strategy and that value will be used instead.


