# ModSecurity

## Embedding ModSecurity
> 主要翻译 & 总结官网相关配置

### Step 5: Creating the base configuration
* nginx作为反向代理

``` yaml
# nginx main config:
http {
    modsecurity on;
    modsecurity_rules_file ./modsec/main.conf;
}
```

``` yaml
# modsec/main.conf
# Include the recommended configuration
Include /data/openresty/nginx/conf/modsec/modsecurity.conf

# A test rule SecRule
SecRule ARGS:testparam "@contains test" "id:1234,deny,log,status:403"

Include /data/openresty/nginx/conf/modsec/crs-setup.conf
Include /data/owasp-modsecurity-crs-3.0.0/rules/*.conf
```

* modsecurity main config:

``` yaml
# enable modsecurity
SecRuleEngine On
# enable access request body,
# limit: only examined header lines and size limit
SecRequestBodyAccess On
# file upload size limit(Content-Type: multipart/form-data):
SecRequestBodyLimit 13107200
# form limit(Content-Type application/x-www-form-urlencoded):
SecRequestBodyNoFilesLimit 64000

# 此处虽然开启了response access, 但并不支持response body检查；
SecResponseBodyAccess On
# 不区分file or form;
SecResponseBodyLimit 10000000

# 当访问量时，修改路径；
SecTmpDir /tmp/
SecDataDir /tmp/
SecUploadDir /tmp/

SecAuditEngine RelevantOnly
SecAuditLogRelevantStatus "^(?:5|4(?!04))"
SecAuditLogParts ABEFHIJZ

SecAuditLogType Serial
SecAuditLog logs/modsec_audit.log
# 同时写入single文件的路径；
SecAuditLogStorageDir logs/audit


SecPcreMatchLimit 500000
SecPcreMatchLimitRecursion 500000

SecDebugLog logs/modsec_debug.log
SecDebugLogLevel 0

# 
# pass: 触发规则，动作: request pass;
# log: error log and 分配唯一tag id;
# 
# * SecDefaultAction five phases
#     * phases 1: request headers 
#     * phases 2: request body
#     * phases 3: response header
#     * phases 4: response body
#     * phases 5: logging
SecDefaultAction "phase:1,pass,log,tag:'Local Lab Service'"
SecDefaultAction "phase:2,pass,log,tag:'Local Lab Service'"

# 梳理规则id:
# == ModSec Rule ID Namespace Definition
# Service-specific before Core-Rules:    10000 -  49999
# Service-specific after Core-Rules:     50000 -  79999
# Locally shared rules:                  80000 -  99999
# Recommended ModSec Rules (few):       200000 - 200010
# OWASP Core-Rules:                     900000 - 999999


# === ModSec Recommended Rules (in modsec src package) (ids: 200000-200010)

SecRule REQUEST_HEADERS:Content-Type "(?:application(?:/soap\+|/)|text/)xml" \
"id:200000,phase:1,t:none,t:lowercase,pass,nolog,ctl:requestBodyProcessor=XML"

SecRule REQUEST_HEADERS:Content-Type "application/json" \
  "id:200001,phase:1,t:none,t:lowercase,pass,nolog,ctl:requestBodyProcessor=JSON"

SecRule REQBODY_ERROR "!@eq 0" \
  "id:200002,phase:2,t:none,deny,status:400,log,\
  msg:'Failed to parse request body.',logdata:'%{reqbody_error_msg}',severity:2"

SecRule MULTIPART_STRICT_ERROR "!@eq 0" \
  "id:200003,phase:2,t:none,deny,status:403,log, \
  msg:'Multipart request body failed strict validation: \
  PE %{REQBODY_PROCESSOR_ERROR}, \
  BQ %{MULTIPART_BOUNDARY_QUOTED}, \
  BW %{MULTIPART_BOUNDARY_WHITESPACE}, \
  DB %{MULTIPART_DATA_BEFORE}, \
  DA %{MULTIPART_DATA_AFTER}, \
  HF %{MULTIPART_HEADER_FOLDING}, \
  LF %{MULTIPART_LF_LINE}, \
  SM %{MULTIPART_MISSING_SEMICOLON}, \
  IQ %{MULTIPART_INVALID_QUOTING}, \
  IP %{MULTIPART_INVALID_PART}, \
  IH %{MULTIPART_INVALID_HEADER_FOLDING}, \
  FL %{MULTIPART_FILE_LIMIT_EXCEEDED}'"

SecRule TX:/^MSC_/ "!@streq 0" \
  "id:200005,phase:2,t:none,deny,status:500,\
  msg:'ModSecurity internal error flagged: %{MATCHED_VAR_NAME}'"
```




### reference
* https://www.netnea.com/cms/nginx-tutorial-6_embedding-modsecurity/