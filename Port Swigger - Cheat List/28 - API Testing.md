```
Access documentation in /api to see if it displays useful endpoints


Use OPTIONS header to see PATCHs and that stuff


Use Content Type Converter Extension to easilly change types to JSON or XML


search new params:
username=administrator%26x=y (&x=y)
username=administrator%23 (#) ('field' not specified may be that there are more params)
username=administrator%26field=x%23 (field is a valid parameter) (Try sending to the intruder the value of field - Select the built-in **Server-side variable names**)

Testing parameters:
Testing valid parameter values shows the original response, indicating they are valid.
username=administrator%26field=reset_token%23
-> Shows a reset passw token
Request in browser:
GET /forgot-password?reset_token=123456789&csrf=the-one-of-the-post


See if GETs and POSTs share JSON body for example (stdout of GET into stdin POST), if POST is missing chosen_discount for example, add it and update the discount to 100
```

