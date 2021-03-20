---
layout: single
title:  "Introduction to Application Security"
date:   2020-01-23 10:28:04 -0500
categories: appsec software
author: XanderK
---

# Introduction to Application Security
Writing custom applications involves taking responsibility for a security boundary into your organization. This means that you need to know how to mitigate and prevent common security problems found in software.

Most security concerns are part of a few very general concepts. First, never trust user provided data. Always consider that data to be potentially harmful and sanitize it before using it. Second, encrypt any and all sensitive data, both on disk and in communication with another process or system. Third, always keep your software up to date with latest framework patches, dependency versions, and on a supported operating system. 

What do we mean when we say, “Never trust user provided data?” When accepting input from a user, you should always apply rules to that input with either an “allow” or “deny” approach. An allow approach means you only accept values that exist in a specific list. Alternatively, a deny approach means you will accept any values that are not in a specific list. Either approach is valid but allowing only a specific list of values is more maintainable. Deny lists frequently require changes as new attack strategies are discovered. By only allowing a limited number of values, you eliminate a significant amount of the risk involved with changing attack strategies. 

In addition to never trusting user input, never trust that the operating environment in which your software runs is secure. This includes data stored on a disk and data sent to or received from an external process or system. What does this mean for you, the developer? 

When you decide how you are storing and referencing sensitive information like usernames, passwords, connection strings, sensitive user information, etc., you should ensure proper encryption is in place to protect that information. This includes using existing encryption algorithms to encrypt the configuration data and the user data. If that data is stored in a database, it should be encrypted, at minimum, on disk. Where the data is especially sensitive, such as Personally Identifiable Information (PII), using database field/row level encryption is preferred. 

When you are making a connection to another process or server, regardless of protocol, it should be over an encrypted channel. For web traffic, this would be HTTPS, over other protocols, it could involve utilizing an SSH tunnel. When making a connection to a database server, make use of the built-in encrypted communications provided by the vendor. When considering how to apply encryption, there is a fundamental tenant to follow; never build your own encryption. Always use established encryption implementations provided by reputable software vendors. If you have any questions, reach out to the Application Security team.

Finally, it is critical to understand that you must continue to maintain software as long as that software is in production. This means that you need to continually revisit the codebase to upgrade dependencies and frameworks to keep the code secure and maintainable. This also means that the operating environment in which the software is deployed must be upgraded and maintained. Failure to do either results in out-of-date software which becomes vulnerable as new security flaws are discovered. 