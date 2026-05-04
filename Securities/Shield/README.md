AWS Shield & WAF use together, where Shield work on network layer (layer 3 & 4) and protect from the Fake request which increase the load in our server to made slow down.
AWS SHield are use mainly to protect from the DDoS (Distributed denial of Service) attack.

Shield Standard ( Free and it is automatically allow for all users by default).
Shield Advance ( costly and high security and provide some security layer in layer 7 ).

--------------------

AWS WAF work on application layer ( Layer 7 HTTP / HTTPs ). WAF look inside the traffic, what there is inside the traffic. For ex. SQL Injection protection. XSS ( Cross Site Scripting ). WAF help you to get identified and block that site. In WAF we can allow IP or Location based blocking. We can also set rules for Rate Limiting like if any ip requesting a request n number of time, In this case we set the limitation and block that IP if requests are beyound the limits.
WAS depends on Rules and Rules Group. We creates the Rules and these created rules checks each and every requests.

-----------------------
1.  Create CloudFront CDN
2.  Origin Type:
        Selected Elastic Load Balancer  →   Selected created ALB
3.  Create Distribution 



WAF Creation steps:
1.  App category:: Need to select from the given options:
    →   In category, there are different type of options available, we have to choose any one or more based on our requirement:

    | Option                    |       When to Choose          |
    | ------------------------- | ----------------------------- |
    | Content & publishing      | Blogs, WordPress sites        |
    | E-commerce                | Payment, shopping apps        |
    | Enterprise apps           | Internal dashboards (HR, CRM) |
    | API & integration         | ALB + EC2 web/API apps        |
    | Media & file processing   | Video/image platforms         |
    | Other                     | If nothing fits               |

2.  Select resources to protect:
    →   This is the most important step 👍 (attaching WAF to the correct resource).
    →   Purpose of attaching a resource in AWS WAF:
        →   Attaching a resource tells WAF where to apply its security rules.
        
        Think of it like this (Very Simple)
        →   👉 WAF = Security Guard 🚓
        →   👉 Resource (ALB) = Entry Gate 🚪

        If you don’t assign a gate,
        →   👉 the guard has nowhere to check people
    
    →   What exactly WAF does after attaching?
        →   When attached to ALB, WAF can:
        ✅ Allow traffic
            →   Normal users
        
        ❌ Block traffic
            →   SQL injection attacks
            →   XSS attacks
            →   Bad IPs
        
        ⚠️ Rate limit
            →   Prevent DDoS-like behavior (basic level)
    
    🧠 Technical Understanding (Interview Level)
        👉 When you attach WAF to ALB:
            →   WAF integrates at Layer 7 (HTTP/HTTPS)
            →   Inspects:
                →   Headers
                →   Body
                →   Query strings
                →   IP
        👉 Then applies:
            →   Web ACL rules

3.  Choose initial protections:
    →   This is the Rule selection step. There are 3 options to select the rule based on our requirement.
    🟦 1. Recommended Rules (Best Default Choice)
        ✅ What it includes
            →   AWS Core Rule Set (SQLi, XSS, etc.)
            →   IP reputation list (bad IPs)
            →   Anonymous IP protection (VPN/TOR)
            →   Rate limiting (GET + POST)
            →   Bot Control (advanced)

        🧠 When to use (REAL WORLD)
            ✅ Use this when:
                →   You are building:
                    →   Web apps (ALB + EC2)
                    →   APIs
                    →   SaaS platforms
                    →   You want:
                        →   Strong security with minimal effort
                    →   You are:
                        →   Beginner → Intermediate → Even production

        🏢 Real Example
            👉 E-commerce app
            👉 Startup SaaS product
            👉 Portfolio project (your case)

        ⚠️ Tradeoff
            Slightly higher cost 💰
            Less customization
        
        🎯 Verdict
            👉 Best for 80% of real-world use cases

    🟨 2. Essential Rules (Basic Protection)
        ✅ What it includes
            →   Layer 7 DDoS protection
            →   IP allow/block list
            →   Geo blocking
            →   Rate limiting
            →   AWS Core Rule Set (basic)

        🧠 When to use
            ✅ Use this when:
                →   Budget is limited 💰
                →   Traffic is low
                →   App is:
                    →   Internal tool
                    →   Small website
                    →   Dev/Test environment

        🏢 Real Example
            👉 Internal HR portal
            👉 Small company dashboard
            👉 Testing environment

        ⚠️ Limitations
            👉 No bot protection ❌
            👉 No advanced threat intelligence ❌
            👉 Less protection vs modern attacks ❌

        🎯 Verdict
            👉 Good for low-risk apps only

    🟥 3. Build Your Own (Advanced Mode)
        ✅ What it gives
            →   Full control over:
                →   Rules
                →   Conditions
                →   Actions

        🧠 When to use (VERY IMPORTANT)
            ✅ Use this when:
                →   You are working in:
                    →   Enterprise company
                    →   Security-focused team
                    →   You need:
                        →   Custom logic
                        →   Compliance (HIPAA, PCI, etc.)
                        →   You already understand:
                            →   WAF rules deeply

            🏢 Real Example
                👉 Banking application
                👉 Healthcare system
                👉 High-traffic API platform

                Example rules:
                    →   Allow only specific countries
                    →   Block specific headers
                    →   Custom rate limit per API
            
            ⚠️ Risks
                →   Easy to misconfigure ❌
                →   Can block real users ❌
                →   Requires monitoring ❌

            🎯 Verdict
                👉 Use only when you KNOW what you're doing




4.  