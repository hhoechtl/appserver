# Upgrade from 1.1.0 to 1.1.1

## Default Headers

To improve security, we've added some default headers that helps to avoid some high risk security vulnerabilities. The headers are

* X-Frame-Options: Deny
* X-XSS-Protection: 1
* X-Content-Type-Options: Nosniff

The X-Content-Type-Options header with the Nosniff values forces you or your application to set the correct Content-Type header to let the browser render your content. If not, the browser will always render your content as plain text.

## Authentication Manager

If you don't deliver a custom `META-INF/context.xml` file with your application, you don't have to change anything here.

Because of a massive refactoring of the security subsystem, we've refactored the servlet engine's authentication package, whereas the namespace has been switched from `AppserverIo\Appserver\ServletEngine\Authentication\` to `AppserverIo\Appserver\ServletEngine\Security`. Therefore, you've to customize the `META-INF/context.xml` file (if available) by changing

```xml
...
<managers>
        <manager name="AuthenticationManagerInterface" type="AppserverIo\Appserver\ServletEngine\Authentication\StandardAuthenticationManager" factory="AppserverIo\Appserver\ServletEngine\Authentication\StandardAuthenticationManagerFactory">
        </manager>
    ...
</managers>
```

to 

```xml
...
<managers>
        <manager name="AuthenticationManagerInterface" type="AppserverIo\Appserver\ServletEngine\Security\StandardAuthenticationManager" factory="AppserverIo\Appserver\ServletEngine\Security\StandardAuthenticationManagerFactory">
        </manager>
    ...
</managers>
```

## Session Manager

If you don't deliver a custom `META-INF/context.xml` file with your application, you don't have to change anything here.

Because of a massive refactoring of the session handling, you have to replace the following line in the file `META-INF/context.xml` 

```xml
<manager name="SessionManagerInterface" type="AppserverIo\Appserver\ServletEngine\SimpleSessionManager" factory="AppserverIo\Appserver\ServletEngine\SimpleSessionManagerFactory"/>
```

with

```xml
<manager name="SessionManagerInterface" type="AppserverIo\Appserver\ServletEngine\StandardSessionManager" factory="AppserverIo\Appserver\ServletEngine\StandardSessionManagerFactory">
    <sessionHandlers>
        <sessionHandler name="filesystem" type="AppserverIo\Appserver\ServletEngine\Session\FilesystemSessionHandler" factory="AppserverIo\Appserver\ServletEngine\Session\SessionHandlerFactory"/>
    </sessionHandlers>
</manager>
```

## Security Subsystem

Up with this version we removed the old security subsystem that allows you to configure Basic and Digest authentication based on a URL pattern in the `WEB-INF/web.xml` file like

```xml
<security>
    <url-pattern>/dhtml/basic.dhtml*</url-pattern>
    <auth>
        <auth-type>Basic</auth-type>
        <realm>test</realm>
        <adapter-type>htpasswd</adapter-type>
        <options>
            <file>WEB-INF/htpasswd</file>
        </options>
    </auth>
</security>
```

As a replacement, use the webserver authentication a server or a virtual host node of your applications `META-INF/containers.xml`, file like

```xml
<authentications>
    <authentication uri="^\/dhtml\/basic.dhtml.*">
        <params>
            <param name="type" type="string">\AppserverIo\Http\Authentication\BasicAuthentication</param>
            <param name="realm" type="string">PhpWebServer Basic Authentication System</param>
            <param name="file" type="string">WEB-INF/htpasswd</param>
        </params>
    </authentication>
</authentications>
```