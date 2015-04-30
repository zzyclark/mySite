---
published: true
title: Angularjs Spring Security CSRF configuration
layout: post
author: Clark
category: articles
tags:
- Angularjs
- Spring Security
description: a solution about how to implement Angularjs $http post with Spring MVC
comments: true
---

Recently, my current project is built by Spring mvc framework, and base on project requirement, also integrated with Spring security. My project use spring 4.0 and spring security 3.2.8. To prevent Cross-Site Request Forgery attack, I enable csrf in my spring security context.

We can set a simple spring security context like this:
{% highlight xml linenos %}
<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
                        http://www.springframework.org/schema/security
                        http://www.springframework.org/schema/security/spring-security-3.2.xsd">

    <http auto-config="true">
        <intercept-url pattern="/admin**" access="ROLE_SUPERUSER" />
        <csrf/>
    </http>

    <authentication-manager>
      <authentication-provider>
        <user-service>
        <user name="clark" password="123456" authorities="ROLE_SUPERUSER" />
        </user-service>
      </authentication-provider>
    </authentication-manager>

</beans:beans>
{% endhighlight %}

Now, for normal form we can insert a hidden input to post: the hidden param name set to _csrf.parameterName and param value set to _csrf.token.

If we just use normal form post, everything is perfect now. But if we want to have a better front side user experience, for example, we want to use ajax to handle data and send data to server in order to stay on a static page, normal spring security csrf setting is not enough.

Base on spring docs, if we use jQuery or rest.js, it’s easy for us to setup, because following the spring docs is enough. But the trouble is that i use angularjs to do the front side data handling, angularjs has its own csrf protection setting, which is xsrf. Now, spring csrf normal setting is not enough for us.  We need to change spring csrf token to angularjs xsrf token.

To accomplish this, we need to change csrf headerName in our spring security context:

{% highlight xml linenos %}
<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security-3.2.xsd">

    <http auto-config="true">
        <intercept-url pattern="/admin**" access="ROLE_SUPERUSER" />
        <csrf token-repository-ref="csrfTokenRepository">
    </http>

    <authentication-manager>
      <authentication-provider>
        <user-service>
        <user name="clark" password="123456" authorities="ROLE_SUPERUSER" />
        </user-service>
      </authentication-provider>
    </authentication-manager>
    <beans:bean id="csrfTokenRepository" class="org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository">
      <property name="headerName" value="X-XSRF-TOKEN" />
    </beans:bean>
</beans:beans>
{% endhighlight %}

For angularjs, we will use [spring-security-csrf-token-interceptor](http://github.com/aditzel/spring-security-csrf-token-interceptor) to get xsrf token from spring. Here we should give best thanks to the author aditzel for this easy tool. To use this tool is just adding this module to our angular app. Here i won’t write the detail. This interceptor will call from spring to get our xsrf token. So we also need a filter in our spring to send xsrf token for each request.

The custom filter will write like this:

{% highlight java linenos %}
public class CsrfFilter extends OncePerRequestFilter
{
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException
    {
        CsrfToken csrf = (CsrfToken) request.getAttribute(CsrfToken.class
                .getName());
        if (csrf != null) {
            Cookie cookie = new Cookie("XSRF-TOKEN", csrf.getToken());
            cookie.setPath("/");
            response.addCookie(cookie);
        }
        filterChain.doFilter(request, response);
    }
}
{% endhighlight %}

Finally we will add this custom filter in our spring context.Now, we can use angularjs $http.post() to do server side data management. As for present, i found i still can use spring csrf input tag in my normal form post, this can give us lots of options to choose the most proper implementation in our project.
