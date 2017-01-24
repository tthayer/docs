# Title: IBM Maximo (ICD) rest login when the MAXAUTH header is disabled
## Summary: Steps involved in accessing the API using session tokens
## Author: tony.thayer@gmail.com
## Date: 20170124

IBM's documentation has several examples showing REST API authentication.

  * passing username and password in the URL:
    * This is terrible for any reason you can think of. 
    * For testing with a preprod system it is barely acceptable.
    * Your password will be retained in the server request logs in plaintext even if you're using SSL.

  * passing a base64-encoded string comprised of 'username:password' in a maxauth HTTP header
    * This is a little safer than using the URL method since it's part of your encapsulated request.
    * It only works if Maximo is configured to allow this authentication method, which it isn't at my workplace.

  * User session
    * This is documented on page 19 here:
      https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/02db2a84-fc66-4667-b760-54e495526ec1/page/87348f89-b8b4-4c4a-94bd-ecbe1e4e8857/attachment/dd399e0a-957c-4736-9be3-9d64635a5407/media/Maximo%20JSON%20API_Query.pdf
    * This method provides a REST endpoint for login but it has the same limitations of the two methods listed above.

  * Simulate a login from a browser
    * This method allows use of the REST API while also adhering to the same authentication configuration end users are stuck with.
    * The steps are basically as follows:
      * Authenticate your request by accessing https://yourmaximo/maximo/j_security_check
        * POST request
        * header set to "Content-Type: application/x-www-form-urlencoded"
        * Payload data comprising of the following:

          ```python
          payload = {
            'allowinsubframe': 'null',
            'mobile': 'false',
            'login': 'jsp',
            'loginstamp': str(time.time()),
            'localStorage': 'true',
            'j_username': _username,
            'j_password': _password,
          }
          ```

        * The payload needs to be urlencoded in your request:

          ```python
          session = requests.Session()
          session.post(url=maximo_login_endpoint, data=urllib.urlencode(payload), headers=headers, verify=False)
          ```

        * Inspecting your request, you should see two cookies. One has your JSESSIONID and the other is your LtpaToken response.
        * Using the python requests library, you can create a session and reuse it for subsequent requests.

          ```python
          r = session.get('https://yourmaximo/maximo/oslc/os/mxwo?lean=1&searchAttributes=wonum&oslc.searchTerms=%22IW123456%22&oslc.select=wonum,status')
          ```

        * Your session is good for 30 minutes, though you can easily destroy and re-create the session if needed.
