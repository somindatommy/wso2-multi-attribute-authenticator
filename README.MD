# Custom Basic Authenticator for multi-attribute authentication

This component was developed to allow users to use a user attribute (eg: email, phone number) as username when logging in
 with basic authentication instead of the immutable username which [WSO2 Identity Server](https://wso2.com/identity-and-access-management/) has by default.
 
 This component has been tested with WSO2 IS versions from 5.5.0 to 5.9.0.
 
 If you are working with WSO2 IS version 5.5.0, 5.6.0, 5.7.0 or 5.8.0, please find the documentation [here](README-5.8.0-and-old.MD). The following stpes are for 5.9.0.

### Steps to deploy
- Build the component by running "mvn clean install"
- Copy following jar file which can be found in target directory into <IS_HOME>/repository/components/dropins/
org.wso2.sample.authenticator.multi.attribute-1.0.0.jar
- Configure the customer authenticator by adding following in deployment.toml.
    ```
    [authentication.custom_authenticator.multi-attribute-authenticator]
    name = "MultiAttributeAuthenticator"
    enable = "true"
    
    [authentication.custom_authenticator.multi-attribute-authenticator.parameters]
    AuthMechanism = "basic"
    claimuri_1 = 'http://wso2.org/claims/emailaddress'
    claimregex_1 = '^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}$'
    claimuri_2 = 'http://wso2.org/claims/mobile'
    claimregex_2 = '^\+?(\d{3})*[0-9,\-]{8,}$'
    ```
    
- If you are going to use email address as as one of the authenticating attributes, add following in deployment.toml
    ```
    [tenant_mgt]
    enable_email_domain = "true"
    ```

- If you want to use this authenticator for all the service providers
  * update the following properties in the <IS_HOME>/repository/conf/identity/service-providers/default.xml file.
    ```
    <LocalAuthenticatorConfig>
        <Name>MultiAttributeAuthenticator</Name>
        <DisplayName>multi-attribute-authenticator</DisplayName>
    ```
  * Also  update the following variable in <IS_HOME>/repository/deployment/server/webapps/authenticationendpoint/login.jsp
    ```
    private static final String BASIC_AUTHENTICATOR = "MultiAttributeAuthenticator";
    ```
- To use this only for one/few service providers
  * Restart the Identity Server
  * Configure "attribute-based-authenticator" in authentication steps in Local and Outbound authentication config of the
 service providers instead of "basic".
  * Open <IS_HOME>/repository/deployment/server/webapps/authenticationendpoint/login.jsp
  * Add a new variable `MULTI_ATTRIBUTE_AUTHENTICATOR` near the `BASIC_AUTHENTICATOR` as below.
    ```
    private static final String BASIC_AUTHENTICATOR = "BasicAuthenticator";
    private static final String MULTI_ATTRIBUTE_AUTHENTICATOR = "MultiAttributeAuthenticator";
    ```
  * Update all the place which has `localAuthenticatorNames.contains(BASIC_AUTHENTICATOR)` as following.
    ```
    localAuthenticatorNames.contains(BASIC_AUTHENTICATOR) || localAuthenticatorNames.contains(MULTI_ATTRIBUTE_AUTHENTICATOR)
    ```
- To use this authenticator for WSO2 IS user dashboard, update the following properties in 
<IS_HOME>/repository/conf/identity/service-providers/sp_dashboard.xml and restart the Identity Server.
    ```
    <LocalAuthenticatorConfig>
        <Name>MultiAttributeAuthenticator</Name>
        <DisplayName>multi-attribute-authenticator</DisplayName>
    ```

### Explanation of the configuration parameters of the authenticator
- `AuthMechanism`: This is to tell identity server to consider this authenticator also using the "basic" auth mechanism.
 This is useful if you are trying to SSO with other service providers which are using default basic authenticator.
- `claimuri_1`: Claim URI which you will be using to validate user's identification (eg: http://wso2.org/claims/emailaddress, http://wso2.org/claims/mobile)
- `claimregex_1`: Regex pattern to check whether the user has entered a value for above claim
* We can have multiple claimuri, claimregex pairs like below. Then, it will check from the begining to the end and use the matching regex's claim URI to continue the authentication.
    ```
    claimuri_1 = 'http://wso2.org/claims/emailaddress'
    claimregex_1 = '^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}$'
    claimuri_2 = 'http://wso2.org/claims/mobile'
    claimregex_2 = '^\+?(\d{3})*[0-9,\-]{8,}$'
    claimuri_3 = 'http://wso2.org/claims/pin'
    claimregex_3 = '^[0-9]{4}$'
    ```
