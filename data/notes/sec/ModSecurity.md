*2019-10-01*

# ModSecurity

ModSecurity is a WAF engine (library, module) for Apache, Nginx, IIS. ModSecurity 3.0 has a new modular architecture - it's composed of:

1. [ModSecurity-nginx](https://github.com/SpiderLabs/ModSecurity-nginx) - a connector that links libmodsecurity to the web server it is running with - NGINX in this case (it takes the form of an nginx module)
2. [ModSecurity](https://github.com/SpiderLabs/ModSecurity) (a.k.a. libmodsecurity :-) - core component containing the functionality and couple of rules

Files

* `nginx.conf` - ModSecurity-Nginx [directives](https://github.com/SpiderLabs/ModSecurity-nginx#usage) (like `modsecurity on`, `modsecurity_rules_file /etc/nginx/modsecurity/nginx-modsecurity.conf`)
* `modsecurity/modsecurity.conf` - ModSecurity [directives](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual-%28v2.x%29#Configuration_Directives) v2.x (like `SecRuleEngine`, `SecRequestBodyAccess`, `SecAuditEngine`)

Tips

* audit logs are great for visibility but bad for performance - you can disable them via `SecAuditEngine off` (you still have the Nginx error logs)
* you should not inspect static content (like images) for performance reasons

## Directives

### SecRule

```
SecRule VARIABLES   "OPERATOR"                "TRANSFORMATIONS,ACTIONS"
# E.g.
SecRule REQUEST_URI "@streq /index.php" "id:1,phase:1,t:lowercase,deny"
```

* VARIABLES - where to look (targets)
* OPERATOR - when to trigger a match
* TRANSFORMATIONS - how to normalize VARIABLES data
* ACTIONS - what to do when rule matches

### SecDefaultAction

If no ACTIONS are provided is `SecRule`, default actions apply as per `SecDefaultAction`.

```
SecDefaultAction "phase:1,log,auditlog,pass"
SecDefaultAction "phase:2,log,auditlog,pass"
```

## More

* https://www.modsecurity.org/CRS/Documentation/making.html
* https://support.kemptechnologies.com/hc/en-us/articles/209635223-How-to-write-a-WAF-rule-Modsecurity-Rule-Writing

# OWASP ModSecurity Core Rule Set (CRS)

* https://coreruleset.org/
* definitions of the malicious patterns (signatures, blacklist rules)
* documentation: https://github.com/SpiderLabs/OWASP-CRS-Documentation => https://www.modsecurity.org/CRS/Documentation/

Files

* `modsecurity/crs/setup.conf` - config file
* `modsecurity/crs/rules` - directory with rules (you should modify only `*EXCLUSION-RULES*`)

Tips

* should be used for all ModSecurity deployments
* to [tune](https://www.oreilly.com/ideas/how-to-tune-your-waf-installation-to-reduce-false-positives
) set a high anomaly threshold and progressively lower it

Paranoia levels (FP = false positive - a WAF blocking a valid request):

1. (default) basic security, minimal amount of false positives (FPs)
2. elevated security level, more rules, fair amount of FPs
3. online banking level security, specialized rules, more FPs
4. nuclear powerplant level security, insane rules, lots of FPs

You can configure rules via:

* excludes in `RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf` or `REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf`
* [whitelists](https://www.modsecurity.org/CRS/Documentation/exceptions.html#exceptions-versus-whitelist) (complicated, performance impact)
* paranoia levels in `crs/setup.conf`
* other configuration options in `crs/setup.conf` (like DOS protection, IP reputation, protocol enforcement)
* "includes" in `nginx-modsecurity.conf` (`Include /etc/nginx/modsecurity/crs/rules/*.conf`) - not supported by docs AFAIK

More

* https://coreruleset.org
* https://coreruleset.org/20171214/practical-ftw-testing-the-core-rule-set-or-any-other-waf/
* https://github.com/SpiderLabs/owasp-modsecurity-crs

# Attacks for testing WAF

```
curl -I "https://$FQDN/?exec=/bin/bash"           # Remote Code Execution (RCE)
curl -I "https://$FQDN/?id=1'%20or%20'1'%20=%20'" # SQL Injection (SQLi)
curl -I "https://$FQDN/?page=/etc/passwd"         # Local File Inclusion (LFI)
curl -I "https://$FQDN/?<script>"                 # Cross Site Scripting (XSS)
```

See also [waf-tester](https://github.com/jreisinger/waf-tester).
