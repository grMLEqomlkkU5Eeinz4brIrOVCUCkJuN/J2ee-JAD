
# Reflection: From EJS to JSP

## It's Weirdly Familiar

The first thing that hits you about JSP is how familiar it looks. The `<% %>` syntax is exactly the same as EJS. You embed logic in HTML the same way. Even the mental model is identical: template plus data equals page.

It's honestly eerie. EJS, a Node.js tool from the 2010s, basically copied the syntax JSP invented in the late 90s. I don't know if it was intentional homage or just convergence, but they landed on the same solution. And you know what? The syntax survived because it works. It's clear and it gets the job done.

And I know why JSP was made, to avoid printing lines of html code manually with java server side logic, but this is like SSR-templating engines, but on a really tight budget.

## The Monolithic Way

But here's where it gets interesting. JSP and Tomcat really went all in on the monolithic structure. And I mean *all in*.

Your application doesn't just run on Tomcat. It lives *inside* Tomcat. Your JSP pages compile into servlets that become part of Tomcat's JVM. Everything (business logic, sessions, database connections) exists in this one big runtime. Tomcat isn't proxying requests to your app like Nginx does with Node. Tomcat *is* your app.

Compare that to how we build things now:
- Nginx forwards to separate Node processes
- Docker isolates everything
- Apps are just processes you can restart
- Microservices talk over networks

In JSP land, you deploy by dropping a WAR file into Tomcat's webapps folder. The container absorbs your code. There's no separation, no network boundary, no process isolation. It's monolithic by design, and honestly they owned it.

## Feeling More Mature

Looking at JSP now, I feel like I understand things better. Not because JSP is outdated (plenty of companies still use it), but because I can see the decisions behind it and why they made sense.

When I learned EJS, I just accepted it as "how templates work." Now I see it's one point in a longer story. JSP embedded Java in the 90s. PHP did something similar. Then in the 2010s, template engines like Handlebars tried to separate logic from presentation. EJS brought the embedded approach back. Now React and Vue mix everything together again.

JSP's monolithic approach wasn't a mistake. It was the right design for when:
- You scaled by buying bigger servers
- You deployed every few weeks
- One application did everything
- Standards like J2EE mattered

Understanding this helps me make better choices now. Modern systems use proxies and separate processes not because the old way was wrong, but because everything changed. Cloud infrastructure and containers made distributed architectures possible.

## What JAD Taught Me

J2EE Application Development has been really good. It's easy to write off old tech as irrelevant, but JAD showed me the foundations of modern web dev. Learning servlets helped me understand Express middleware better. Seeing session management taught me how state works over stateless HTTP. Connection pooling explained why database connections matter.

But the biggest thing was context. I'm not just learning tools anymore. I'm seeing how web architecture evolved. Why we went from monoliths to microservices. Why we moved from server pages to SPAs. Why servlet containers became serverless functions.

The module taught me that being a good engineer isn't about chasing the newest framework. It's about understanding tradeoffs and making smart decisions.

JSP and EJS look the same but come from different worlds. One embraced the monolith, the other fits into distributed systems. Both work. Both solve real problems. Learning both makes me better at my job because I can work with legacy systems and modern ones, and I can think critically about whatever comes next.
