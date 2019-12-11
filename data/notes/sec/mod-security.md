*2019-10-01*

## Terms

* ModSecurity - a WAF engine (library, module) for Apache, Nginx, IIS
* Core Rule Set (CRS) - define the malicious patterns (signatures)
* False positive - a WAF blocking a valid request

## ModSecurity

* `/etc/nginx/modsec/modsecurity.conf` - config file
* audit logs are great for visibility but bad for performance - you can disable them via `SecAuditEngine off` (you still have the Nginx error logs)
* you should not inspect static content (images, ...) for performance reasons

ModSecurity 3.0 has a new modular architecture, i.e. it's composed of:

1. [libmodsecurity](https://github.com/SpiderLabs/ModSecurity) (https://www.modsecurity.org) - core component containing the functionality and couple of rules ([docs](https://github.com/SpiderLabs/ModSecurity/wiki))
2. a connector that links libmodsecurity to the web server it is running with - [NGINX](https://github.com/SpiderLabs/ModSecurity-nginx), Apache HTTP Server, and IIS

https://www.nginx.com/blog/compiling-and-installing-modsecurity-for-open-source-nginx/

### `SecRule` ModSecurity directive

https://www.modsecurity.org/CRS/Documentation/making.html

```
SecRule VARIABLES "OPERATOR" "TRANSFORMATIONS,ACTIONS"
# E.g.
SecRule REQUEST_URI "@streq /index.php" "id:1,phase:1,t:lowercase,deny"
```

* VARIABLES - where to look (targets)
* OPERATOR - when to trigger a match
* TRANSFORMATIONS - how to normalize VARIABLES data
* ACTIONS - what to do when rule matches

https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual-(v2.x)

## CRS

* blacklist rule set
* should be used for all ModSecurity deployments
* `crs/setup.conf` - config file
* `crs/rules` - directory with rules (you should modify only `*EXCLUSION-RULES*`)
* to tune, set a high anomaly threshold and progressively lower it
* https://coreruleset.org, https://github.com/SpiderLabs/owasp-modsecurity-crs, https://www.modsecurity.org/CRS/Documentation/

Paranoia levels:

1. (default) basic security, minimal amount of false positives (FPs)
2. elevated security level, more rules, fair amount of FPs
3. online banking level security, specialized rules, more FPs
4. nuclear powerplant level security, insane rules, lots of FPs

## Attacks for testing WAF

```
curl 'https://$FQDN/?exec=/bin/bash'           # Remove Code Execution (RCE)
curl "https://$FQDN/?id=1'%20or%20'1'%20=%20'" # SQL Injection (SQLi)
curl 'https://$FQDN/?page=/etc/passwd'         # Local File Inclusion (LFI)
curl 'https://$FQDN/?<script>'                 # Cross Site Scripting (XSS)
```

## More

* https://www.oreilly.com/ideas/how-to-tune-your-waf-installation-to-reduce-false-positives
* https://coreruleset.org/20171214/practical-ftw-testing-the-core-rule-set-or-any-other-waf/